# FastAPI és SQLAlchemy: session, tranzakció és HTTP-válasz

## Mit kell tudnod?

- Kéréshez kötött sessiont létrehozni globális session helyett.
- Commitot a sikeres válasz előtt végrehajtani.
- Constraint-sértést célzottan domain/HTTP-hibára fordítani.

## Magyarázat

Az engine a kapcsolatkezelés és pool kiindulópontja, a Session egy állapottal rendelkező unit of work. A session nem megosztható tetszőleges párhuzamos kérések között. Sync session sync kódhoz, AsyncSession async driverhez való; az async wrapper önmagában nem teszi nem blokkolóvá a sync drivert.

A yield dependency jól kezeli a session bezárását. A commit azonban a művelet üzleti határa: derüljön ki a sikere a 201-es válasz előtt. A repository minden kis műveletében végrehajtott commit megakadályozhatja több változtatás atomikus összefogását.

## Kódpélda: egyediség és commit a válasz előtt

A teljes blokk pytest-modul, valódi ideiglenes SQLite-fájllal. Az API és az adatbázis a teszten belül jön létre.

```python
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException
from fastapi.testclient import TestClient
from pydantic import BaseModel, Field
from sqlalchemy import String, create_engine, select
from sqlalchemy.exc import IntegrityError
from sqlalchemy.orm import DeclarativeBase, Mapped, Session, mapped_column

class Base(DeclarativeBase):
    pass

class Job(Base):
    __tablename__ = 'jobs'
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String, unique=True)

class JobInput(BaseModel):
    name: str = Field(min_length=1)

class JobView(BaseModel):
    id: int
    name: str

def test_database_http_boundary(tmp_path):
    engine = create_engine('sqlite:///' + str(tmp_path / 'jobs.db'),
                           connect_args={'check_same_thread': False})
    Base.metadata.create_all(engine)
    app = FastAPI()

    def get_session():
        with Session(engine) as session:
            yield session

    @app.post('/jobs', response_model=JobView, status_code=201)
    def create_job(body: JobInput, session: Annotated[Session, Depends(get_session)]):
        try:
            with session.begin():
                job = Job(name=body.name)
                session.add(job)
                session.flush()
                result = JobView(id=job.id, name=job.name)
        except IntegrityError as exc:
            raise HTTPException(409, 'Job name already exists') from exc
        return result

    try:
        with TestClient(app) as client:
            assert client.post('/jobs', json={'name': 'report'}).status_code == 201
            assert client.post('/jobs', json={'name': 'report'}).status_code == 409
            assert client.post('/jobs', json={'name': 'export'}).status_code == 201
        with Session(engine) as session:
            assert session.scalars(select(Job.name).order_by(Job.id)).all() == ['report', 'export']
    finally:
        engine.dispose()
```

A flush SQL-t küld és ID-t rendelhet, de még nem commit. A begin context manager sikeres kilépése commitol, hiba esetén rollbackel. A DTO-t a session nyitott állapotában állítjuk elő, és csak sikeres tranzakció után adjuk vissza.

## Senior döntések és tipikus hibák

Itt a szűk séma és a validált bemenet mellett a demonstrált IntegrityError az egyedi név konfliktusa. Több constraintes éles sémában nem szabad minden IntegrityErrort „duplikált név”-nek nevezni: a driver kódja/constraint neve és a dokumentált szerződés szerint osztályozz.

A SQLite check_same_thread=False csak a driver thread-ellenőrzését módosítja; nem teszi a Sessiont párhuzamosan biztonságossá. A fájlos teszt nem bizonyít PostgreSQL-izolációt. A create_all itt tesztbootstrap, nem migrációs stratégia.

Külső HTTP-hívást vagy hosszú streamet ne tarts indokolatlanul ebben a tranzakcióban. Háttérfeladatnak job ID menjen át; saját sessiont nyisson. További részletek a [10. témakörben](../10-databases/README.md).

## Interview questions

**What is the difference between flush and commit?**

Válaszvázlat: A flush a függő változtatásokat SQL-lel szinkronizálja a tranzakción belül; a commit a tranzakció sikeres lezárása.

**Why build the response before closing the session but return after commit?**

Válaszvázlat: Elkerülöm a későbbi lazy betöltést, miközben csak igazolt mentés után küldök sikeres HTTP-választ.

## Önellenőrzés

- [ ] A globális engine és a kérés sessionje külön fogalom.
- [ ] Hibás írás után a következő érvényes kérés is működik.

## Kapcsolódó gyakorlat

[F08 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f08)

## Forrás és továbbolvasás

[SQLAlchemy session basics](https://docs.sqlalchemy.org/en/20/orm/session_basics.html)

[Session transactions](https://docs.sqlalchemy.org/en/20/orm/session_transaction.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
