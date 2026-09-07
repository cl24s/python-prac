# ORM, identity map, lazy betöltés és N+1

## Mit kell tudnod?

- Az ORM mögötti SQL-t és queryszámot megfigyelni.
- Session-állapotot és lazy betöltést felismerni.
- A betöltési stratégiát a felhasználási esethez választani.

## Magyarázat

Az ORM objektumokra képezi a relációs adatokat, de nem szünteti meg a query költségét. A Session identity mapje azonos elsődleges kulcshoz adott sessionen belül jellemzően ugyanazt az objektumot kapcsolja. Ez nem megosztott alkalmazáscache, és nem általános „nincs több SQL” garancia.

Lazy kapcsolatnál az owner.jobs attribútum olvasásakor új SELECT indulhat. Ha N ownert kérsz le, majd mindegyik kapcsolatát bejárod, 1 + N query keletkezhet. A lista HTTP-szerializációja is észrevétlenül előidézheti ezt.

## Kódpélda: queryszámmal kimutatott N+1

```python
from sqlalchemy import ForeignKey, create_engine, event, select
from sqlalchemy.orm import DeclarativeBase, Mapped, Session, mapped_column, relationship, selectinload

class Base(DeclarativeBase):
    pass

class Owner(Base):
    __tablename__ = 'owners'
    id: Mapped[int] = mapped_column(primary_key=True)
    jobs: Mapped[list['Job']] = relationship(back_populates='owner')

class Job(Base):
    __tablename__ = 'jobs'
    id: Mapped[int] = mapped_column(primary_key=True)
    owner_id: Mapped[int] = mapped_column(ForeignKey('owners.id'))
    owner: Mapped[Owner] = relationship(back_populates='jobs')

def test_eager_loading_reduces_queries():
    engine = create_engine('sqlite://')
    statements = []
    def record(conn, cursor, statement, parameters, context, executemany):
        if statement.lstrip().upper().startswith('SELECT'):
            statements.append(statement)
    event.listen(engine, 'before_cursor_execute', record)
    try:
        Base.metadata.create_all(engine)
        with Session(engine) as session, session.begin():
            session.add_all([Owner(id=i, jobs=[Job(id=i)]) for i in range(1, 4)])
        statements.clear()
        with Session(engine) as session:
            owners = session.scalars(select(Owner).order_by(Owner.id)).all()
            assert [len(owner.jobs) for owner in owners] == [1, 1, 1]
        assert len(statements) == 4
        statements.clear()
        with Session(engine) as session:
            owners = session.scalars(select(Owner).options(selectinload(Owner.jobs))).all()
            assert sum(len(owner.jobs) for owner in owners) == 3
        assert len(statements) == 2
    finally:
        event.remove(engine, 'before_cursor_execute', record)
        engine.dispose()
```

Két friss sessiont használunk, hogy az első futás betöltött kapcsolatai ne maszkolják a második queryszámát. A két query erre a három ownerre igaz; nagy selectin betöltés több batch-et is jelenthet.

## Betöltési stratégia és senior döntések

selectinload külön lekérdezéssel, az ownerazonosítók alapján tölti a kapcsolatot. joinedload JOIN-nal dolgozik; kollekcióknál sorokat sokszorozhat és SQLAlchemy 2-ben az eredmény unique() kezelése szükséges lehet. Nincs minden kapcsolatra optimális alapértelmezés.

Csak a szükséges mezőt és kapcsolatot töltsd be. Nagy lista .all() hívása teljes memóriába olvasás lehet. Stream/batch feldolgozásnál a session és tranzakció életét is számold; egy export nem foglalhat korlátlanul kapcsolatot.

Commit után az ORM-attribútumok lejárhatnak, lezárt session után a lazy olvasás hibázhat. Async ORM-nél a rejtett I/O különösen problémás: használj explicit betöltést és olyan DTO-t, amely már nem kér új queryt. Az expire_on_commit=False sem általános konzisztenciamegoldás.

## Tipikus hibák

Ne csak a tesztadat egyetlen ownerével vizsgáld a listaendpointot: ott az N+1 alig látszik. Ne nevezd az identity mapet Redis helyettesítőjének. A Session egy threadhez, az AsyncSession egy feladathoz igazított munkaegység legyen, ne párhuzamos megosztott objektum.

## Interview questions

**How can serialization cause an N+1 problem?**

Válaszvázlat: A válaszmodell kapcsolat-attribútumokat olvashat, amelyek ownerenként új SQL-t indítanak.

**Why use a fresh session when comparing loading strategies?**

Válaszvázlat: A korábban betöltött identity-map állapot elfedheti a lazy queryket és hamis eredményt adhat.

## Önellenőrzés

- [ ] Queryszámot több szülőobjektummal mérek.
- [ ] A betöltött DTO session nélkül is használható.

## Kapcsolódó gyakorlat

[DB05 – önálló feladat](../../../exercises/senior-python/10-databases.md#db05)

## Forrás és továbbolvasás

[Relationship loading](https://docs.sqlalchemy.org/en/20/orm/queryguide/relationships.html)

[Session state management](https://docs.sqlalchemy.org/en/20/orm/session_state_management.html)

[AsyncSession concurrency](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
