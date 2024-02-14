---
IDX: "NUM-131"
tags:
  - SqlAlchemy
  - Work
  - Python
  - DataBase
  - FastAPI
description: "Sqlalchemy의 create_all()를 활용한 테이블 생성"
update: "2024-02-14T01:53:00.000Z"
date: "2024-02-12"
상태: "Ready"
title: "SQLAlchemy create_all() 메소드로 데이터베이스 테이블 자동 생성하기"
---
## 서론

현재 회사에서는 Alembic 같은 데이터마이그레이션 도구를 사용하고 있지 않습니다. 따라서 스키마 버전 관리등은 별도로 하고 있지 않으며, 다만 SqlAlchemy의 create\_all() 메소드를 이용해 프로덕션 환경이나 dev 환경에서의 테이블의 누락은 안생기게끔만 간단하게 관리하고 있습니다. 

이 방법은 테이블의 수정 또는 삭제를 반영할 수는 없는 방법이지만, 아직 RDB 기반으로 옮긴지 얼마 되지 않아 상대적으로 DB의 크기가 작아 현재는 이 방법으로 충분한 상황입니다. 

create\_all() 메소드를 사용하면서 알게 된 점을 기록해두려고 합니다. 

## create\_all() 메소드란? 

SqlAlchemy의 create\_all() 메소드는 `Metadata` 객체에 등록된 모든 테이블 정의를 바탕으로 해당 데이터베이스 엔진에 테이블을 생성하는데 사용됩니다. 이 메소드는 이미 생성되어있는 테이블은 생성하지 않으며, 새로 정의된(DB에 없는) 테이블만 새롭게 데이터베이스에 생성합니다. 

따라서, 스키마가 달라도 뭔가 수정이 되었어도 DB에 존재만 한다면 생성하거나 덮어쓰지 않으므로 만약 수정 사항을 반영하는걸 해당 메소드로 진행하려면 테이블을 drop한 뒤 생성해야 합니다. 물론 데이터가 많지 않거나 데이터를 날려도 상관없는 dev 환경에서만 유효한 방법입니다. 

### 기본적인 사용 방법

```python
from sqlalchemy import create_engine, MetaData

# 데이터베이스 엔진 생성
engine = create_engine('sqlite:///example.db', echo=True)

# MetaData 인스턴스 생성
metadata = MetaData()

# 테이블 정의들...
# 예: users_table = Table('users', metadata, Column('id', Integer, primary_key=True), ...)

# create_all 호출하여 모든 테이블 생성
metadata.create_all(engine)
```

`create_engine` 함수는 데이터베이스와의 연결을 설정합니다. `echo=True`는 SQLAlchemy가 실행하는 SQL 명령을 콘솔에 출력하도록 설정하는 옵션입니다.

`create_all()` 메소드는 첫 번째 인자로 `Engine` 객체를 받습니다. 이 `Engine` 객체는 SQLAlchemy가 데이터베이스와 통신하기 위해 사용하는 기본 구성 요소입니다.

### 비슷한 메소드들? 

#### drop\_all()

`drop_all()` 메소드는 `create_all()`과 반대로 작동합니다. `MetaData` 객체에 등록된 모든 테이블을 데이터베이스에서 삭제합니다. 테이블에 데이터가 있을 경우, 데이터도 함께 삭제되므로 사용할 때 주의가 필요합니다.

```python
metadata.drop_all(engine)
```

#### Table.create() 및 Table.drop()

`create_all()` 또는 `drop_all()` 메소드 대신, 개별 `Table` 객체에 대해 `create()` 또는 `drop()` 메소드를 호출할 수 있습니다. 이 방법을 사용하면 특정 테이블에 대한 작업을 더 세밀하게 제어할 수 있습니다.

```python
# 개별 테이블 생성
users_table.create(engine)

# 개별 테이블 삭제
users_table.drop(engine)
```

#### reflect()

`reflect()` 메소드는 데이터베이스의 기존 스키마 정보를 `MetaData` 객체로 로드합니다. 이 메소드는 기존 데이터베이스 구조를 SQLAlchemy 모델로 반영할 때 유용합니다.

```python
metadata.reflect(engine)
```

## 나는 어떻게 사용하고 있나? 

dev 또는 로컬 환경에서 테스트를 할 경우 적극적으로 사용하고 있습니다. 주로 처음 테이블을 생성할 때에 굳이 create 쿼리문을 별도로 쓰지 않고 모델만 생성 후 해당 메소드를 이용해 테이블을 생성하거나, 테이블에 수정 사항이 생긴 경우 drop 후 create\_all() 메소드를 생성해 다시 DB 테이블을 생성합니다. 

물론, 그때 그때 create 함수를 쓰기는 너무나 귀찮은 일이기 때문에, FastAPI의 lifespan을 활용해서 FastAPI 어플리케이션이 시작될 때에 자동으로 create\_all() 메소드를 호출하도록 세팅을 해두었습니다. 

```python
from sqlalchemy.orm import declarative_base

Base = declarative_base()
_db_connection: Engine = Optional[Engine]

@asynccontextmanager
async def lifespan(app: FastAPI):
    await supabase_on_startup()
    settings = Settings()
    logger.info(f"[SERVER_ENVIRONMENT] {settings.desc.SERVER_ENVIRONMENT}")
    yield
    await supabase_on_shutdown()


async def on_startup():
    global _db_connection
    DATABASE_URL = f"postgresql+asyncpg://postgres.{settings.postgresql_setting.projectid}:{settings.postgresql_setting.password}@aws-0-ap-northeast-2.pooler.supabase.com:6543/postgres?prepared_statement_cache_size=0"

    # pool_size와 max_overflow의 초기값 설정
    pool_size = 5  # 기본값
    max_overflow = 10  # 기본값
    echo = False

    _db_connection = create_async_engine(
        DATABASE_URL,
        pool_size=pool_size,
        max_overflow=max_overflow,
        echo=echo,
    )
    # 비동기 엔진을 사용하여 테이블 생성
    async with _db_connection.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
```

이런식으로 구현해두었습니다. 이렇게 하면 어플리케이션이 수정될 때마다 존재하지 않는 테이블을 생성하게 됩니다. 다만, 이렇게 구현했을 때에 주의해야 할 점이 있습니다. 

### 주의

위에서 언급했듯이, create\_all() 메소드는 `Metadata`에 등록된 테이블 정보를 가지고 테이블을 생성합니다. 뒤집어 말하면 `Metadata`에 테이블이 등록되지 않는다면 생성되지 않는 다는 말인데, 따라서  `Metadata`에 테이블 정보가 등록되는 시점을 정확히 아는게 중요합니다. 

### Metadata에 테이블 정보가 등록되는 시점

`MetaData` 객체에 테이블 정보가 등록되는 시점은 해당 `Table` 객체가 생성될 때입니다. SqlAlchemy에서는 `Table` 객체를 `MetaData` 객체에 직접 연결하여 테이블을 정의합니다. 클래스 정의 내에서 `Column`과 같은 필드를 사용하여 `Base` 클래스를 상속받는 모델을 정의하면, 이러한 모델 클래스가 생성될 때 내부적으로 `Table` 객체가 생성되고 `MetaData` 객체에 자동으로 등록됩니다. `declarative_base()`를 호출하여 생성된 `Base` 클래스는 내부적으로 `MetaData` 인스턴스를 포함하고 있으며, 이 `MetaData` 인스턴스는 모델 클래스를 통해 정의된 모든 테이블의 메타데이터를 관리합니다.

따라서, 모델 클래스를 정의하는 순간 해당 클래스에 연결된 `Table` 객체가 생성되고, 이 객체는 자동으로 `Base` 클래스에 내장된 `MetaData` 객체에 등록됩니다. 이후 `Base.metadata.create_all(engine)`와 같은 방식으로 호출하면, `MetaData`에 등록된 모든 테이블에 대한 생성 명령이 데이터베이스 엔진으로 전송됩니다.

#### FastAPI에서 모델 클래스가 정의되는 순간은? 

FastAPI 어플리케이션에서 “모델 클래스가 정의되는 순간”은 Python 스크립트가 실행되어 해당 모델 클래스가 메모리에 로드되는 시점을 의미합니다. 

FastAPI 어플리케이션을 실행하면(예를 들어 `uvicorn main:app` 등의 명령어를 사용하여) Python 인터프리터는 main.py를 로드하고 실행합니다. 이 과정에서 main.py에서 직접적으로, 또는 임포트된 모듈 내에서 다시 임포트된 SqlAlchemy의ㅏ 모델 클래스들도 함께 로드되고 메모리에 생성됩니다. 바로 이 시점에 각 모델 클래스에 해당하는 `Table` 객체가 `Metadata` 객체에 등록됩니다. 

다시 쉽게 이야기하자면, app.py 또는 main.py 같은 파일과 이 파일에 임포트 된 파일들, 다시 임포트 된 파일들 중에 모델 객체가 있어야만 합니다. 외딴섬처럼 덩그러니 model 파일을 생성해두고 아무곳에도 import 하지 않는다면(메인 py 파일과 연결되어 있지 않다면) 테이블은 생성되지 않습니다. 어떻게 생각하면 당연한 일인데, 이 당연한 부분을 놓쳐 길게 헤맸던 기억이 있습니다. 이 기억 때문에 해당 문서를 작성한 셈입니다. 

