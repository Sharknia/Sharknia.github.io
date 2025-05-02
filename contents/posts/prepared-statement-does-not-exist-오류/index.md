---
IDX: "NUM-254"
tags:
  - Supabase
  - SqlAlchemy
  - FastAPI
  - Python
  - Postgresql
description: "Supabase+SQLAlchemy2.x 비동기 엔진에서의 오류"
series: "취미생활 - FastAPI 서버 구성"
update: "2025-05-02T13:40:00.000Z"
date: "2025-04-27"
상태: "Ready"
title: "prepared statement does not exist  오류"
---
![](image1.png)
## 서론

Supabase에서 제공하는 PostgreSQL 풀러(PgBouncer)를 Transaction 모드로 운영하는 환경에서,

Python 3.12과 SQLAlchemy 2.x의 비동기(asyncio) 엔진을 함께 사용할 때 `prepared statement does not exist` 오류가 발생하는 경우가 있습니다.

이 글에서는 오류의 원인과 해결 방법을 코드 예제와 함께 정리합니다.

## 개발 환경

- Python 버전: 3.12 이상

- SQLAlchemy 버전: 2.x (비동기 엔진 사용)

- 데이터베이스: Supabase PostgreSQL + PgBouncer

    - `pool_mode = transaction`

- 비동기 드라이버: asyncpg

## 문제 정의

SQLAlchemy `AsyncSession`으로 반복 쿼리를 실행하다가`asyncpg.exceptions._base.InterfaceError: prepared statement "<name>" does not exist` 예외 발생

## 오류 원인 분석

1. asyncpg의 Prepared Statement 캐시

    - asyncpg는 동일 쿼리를 반복 실행할 때 성능을 높이기 위해 내부적으로 prepared statement를 캐시에 저장합니다. 

1. PgBouncer Transaction 모드

    - `pool_mode = transaction` 설정 시, 각 트랜잭션이 끝날 때마다 서버(PostgreSQL) 쪽의 모든 prepared statement를 삭제합니다.

    - 커넥션은 유지되지만, 트랜잭션 경계마다 statement 정보가 사라짐

1. 충돌 과정

    1. asyncpg: “지난번에 준비해 둔 statement가 있으니 이름만 주세요”

    1. PgBouncer: “이미 다 지웠습니다”

    1. asyncpg: “…그럼 없다고요?” → `InterfaceError: prepared statement "<name>" does not exist`

<hr style="border: none; height: 1px; background-color: #e0e0e0; margin: 16px 0;" />
## 문제 재현 예시

```python
from sqlalchemy.ext.asyncio import AsyncSession, async_sessionmaker, create_async_engine
from sqlalchemy.orm import declarative_base
from .config import settings

# DATABASE_URL 예시: "postgresql://user:pass@host:5432/dbname"
ASYNC_DATABASE_URL = settings.DATABASE_URL.replace(
    "postgresql://", "postgresql+asyncpg://", 1
)

# 비동기 엔진 생성 (statement_cache_size 미설정 상태)
async_engine = create_async_engine(
    ASYNC_DATABASE_URL,
    echo=settings.DEBUG,
)

AsyncSessionLocal = async_sessionmaker(
    bind=async_engine,
    class_=AsyncSession,
    expire_on_commit=False,
)

Base = declarative_base()

async def get_items(session: AsyncSession):
    result = await session.execute(select(Item))
    return result.scalars().all()

async def main():
    async with AsyncSessionLocal() as session:
        for _ in range(200):
            items = await get_items(session)
            print(len(items))
```

200회 반복 실행 시, 약 100회 차 전후로 다음과 같은 오류가 발생합니다.

```plain text
asyncpg.exceptions._base.InterfaceError: prepared statement "s_1_select_item" does not exist
```

## **해결 방법**

### 1. ~~statement\_cache\_size 비활성화~~

가장 간단한 해결책은 asyncpg의 prepared statement 캐시를 사용하지 않도록 설정하는 것입니다.

create\_async\_engine 호출 시 다음과 같이 connect\_args에 "statement\_cache\_size": 0을 추가합니다. 

```python
async_engine = create_async_engine(
    ASYNC_DATABASE_URL,
    echo=settings.DEBUG,
    connect_args={
        "statement_cache_size": 0,  # 캐시 크기를 0으로 설정
    },
    pool_pre_ping=False,
    pool_recycle=3600,
)
```

이렇게 하면 asyncpg가 매번 prepare 없이 쿼리를 바로 전송하기 때문에, PgBouncer가 prepared statement를 삭제하더라도 오류가 발생하지 않습니다.

### 2. Session Pooler 사용

statement\_cache\_size를 비활성화 하더라도 PgBouncer의 Poolmode를 transaction으로 사용하면 이 설정만으로는 오류가 발생합니다. UUID로 매번 statement를 자동생성하라는 권고도 나오곤 하는데, 이렇게 해도 발생하는 경우가 있었습니다. 

편한 방법으로 Session pooler를 사용하겠습니다.

Session 모드는 클라이언트가 연결을 유지하는 동안 다른 클라이언트가 해당 연결을 사용할 수 없으므로, Transaction 모드에 비해 연결 재사용 효율이 떨어질 수 있습니다. 특히 짧은 시간 동안만 연결했다 끊는 작업(예: 서버리스 함수)이 매우 빈번하다면, Transaction 모드보다 더 많은 실제 연결이 필요하게 되어 리소스 사용량이 늘거나 연결 제한에 더 빨리 도달할 수 있지만, 현재는 FastAPI만 사용하고 있으므로 걱정이 없습니다. 

### 3. 직접 연결 (PgBouncer 미사용)

PgBouncer Transaction 모드를 완전히 배제하고, 애플리케이션에서 직접 PostgreSQL 서버에 연결하는 방법입니다.

이 경우 SQLAlchemy `create_async_engine`에 일반 `postgresql+asyncpg://` URL을 그대로 사용합니다.

```python
ASYNC_DATABASE_URL = settings.DATABASE_URL  # "postgresql+asyncpg://..." 그대로 사용

async_engine = create_async_engine(
    ASYNC_DATABASE_URL,
    echo=settings.DEBUG,
    pool_size=20,           # 적절히 커넥션 풀 크기 조정
    max_overflow=10,        # 최대 오버플로우 수
    pool_pre_ping=True,     # 커넥션 유효성 검사
)
```

#### Supabase에서는..?

현재 Supabase에서 다음과 같이 권고하고 있습니다.


        Not IPv4 compatible

    Use Session Pooler if on a IPv4 network or purchase IPv4 add-on

Supabase의 기본 데이터베이스 연결 주소가 주로 IPv6 네트워크를 사용하며, 주인님의 서버처럼 IPv4 네트워크 환경에서 직접 접속하려면 문제가 발생할 수 있다는 것을 의미합니다. Supabase는 Session Pooler 사용을 권장하고 있으며 굳이 IPv4를 사용하려면 추가 애드온 구매를 요구합니다. 

### 장단점 평가

| 구분 | 장점 | 단점 |
| --- | --- | --- |
| 성능 | - Prepared statement 캐시 활용으로 쿼리 준비 시간 절감<br/>- PgBouncer 오버헤드 제거 | - 다수의 커넥션이 직접 서버에 연결되어 자원 소모 증가 |
| 안정성 | - Prepared statement 캐싱과 호환<br/>- 트랜잭션 경계 관리 일관성 보장 | - 커넥션 수 초과 시 장애 위험<br/>- 개별 애플리케이션에서 풀링 관리 필요 |
| 운영 복잡도 | - 설정이 단순 (PgBouncer 설정 불필요) | - 장애 발생 시 개별 커넥션 재시도 로직 추가 필요 |
| 확장성 | - 쿼리 성능 최적화 여지 (캐시, 파라미터 바인딩) | - 대규모 트래픽 시 DB 서버 부하 증가 |



