---
tags:
  - SqlAlchemy
  - DataBase
  - Python
  - Work
update: "2024-01-31"
date: "2024-01-29"
상태: "Ready"
title: "Sqlalchemy에서의 트랜잭션 격리 수준 구현"
---
## 구현 목적

상품 구매에 관련된 API를 구현하려고 한다. DynamoDB를 사용할 때에 동시성 이슈로 쿠폰 중복 구매 이슈가 있었으므로 이번에 RDS로 옮긴 김에 해당 문제를 완벽하게 해결하기 위해 다각도로 방법을 고민했다. 

그 방안 중 하나가 [트랜잭션 격리 수준(Transaction Isolation Level)](https://sharknia.github.io/2024-01-29/트랜잭션-격리-수준Transaction-Isolation-Level) 을 이용한 것이다. 

SqlAlchemy - Postgresql을 사용하고 있는데, 이 라이브러리에서 트랜잭션 격리 수준을 어떻게 구현했는지 기록한다. 

## 현재

현재에는 기본 격리 수준을 사용한(별다른 옵션값이 없는) 엔진만 사용하고 있다. API 별로 별도의 세션을 사용하기 위해 의존성 주입 방식을 사용하며, 이를 위해 `AsyncIterable[AsyncSession]`을 생성한다. 대략적인 코드는 다음과 같다. 

```python
_db_connection: AsyncEngine

...

async def on_startup():
		....
    _db_connection = create_async_engine(
        DATABASE_URL,
        pool_size=pool_size,
        max_overflow=max_overflow,
        echo=echo,
    )
		....

....

async def get_db_connection() -> AsyncEngine:
    assert _db_connection is not None
    return _db_connection

....

async def get_db_session(
    db_conn: AsyncEngine = Depends(get_db_connection),
) -> AsyncIterable[AsyncSession]:
    session = None
    try:
        async with sessionmaker(
            db_conn,
            class_=AsyncSession,
            expire_on_commit=False,
        )() as session:
            yield session
    except Exception as e:
        logger.error(f"[get_db_session] {e}")
        await session.rollback()
        raise
    finally:
        if session:
            await session.close()
```

그리고 각 API의 엔드포인트에서 get_db_session을 주입받아 사용한다. 

이 엔진은 기본적으로 Read Committed 격리수준만 지원한다. 

## 모험

###  sessionmaker 레벨에서 트랜잭션 격리 수준 설정?

이렇게 내용이 조금만 깊어져도 챗지피티는 믿을 놈이 못된다. 챗지피티에서는 sessionmaker 레벨에서 트랜잭션 격리 수준 설정이 가능하다며, sessionmaker의 매개변수에 `isolation_level="SERIALIZABLE”` 를 추가해주면 된다고 주장한다. 

하지만 sessionmaker의 생성자에는 `isolation_level` 매개 변수가 없어 해당 설정은 바로 오류를 낸다. 이럴 경우 답은 스스로 해결하는 것 밖에 없다. 

### 트랜잭션 별 엔진 별도 생성?

첫 구현은 단순하게 생각해서 엔진을 여러개를 만들었다. 즉, `isolation_level` 옵션을 각각 다르게 준create_async_engine을 여러번 하는 것이다. 이러면 간단하게 해결이 된다. 하지만 이렇게 할 경우에는 문제가 있다. 각각의 엔진이 모두 별도의 풀을 생성하면서 의도치 않게 커넥션이 증가할 위험이 있는 것이다. 안그래도 [SqlAlchemy의 QueuePool](https://sharknia.github.io/2024-01-18/SqlAlchemy의-QueuePool) 를 겪었었기 때문에 해당 이슈는 꼭 피하고 싶었다. 

## 개선(해결)

코드를 짜다보면 “이건 있어야 하는데?”라고 느낄때가 있다. 그런것들은 대부분, 너무 미완성인 라이브러리나 프레임워크가 아니라면 반드시 나온다. 구현 가능한데 필요성을 느끼는 것은 반드시 누군가 만들어둔 것이다. 

명확하게 다른 엔진이면서 풀을 공유하는 방법이 존재한다. `execution_options()` 를 사용하면 된다. 

### execution_options()

`AsyncEngine`에서 `execution_options()` 메소드를 사용하면 반환되는 엔진도 `AsyncEngine` 타입이 된다. 이 메소드는 Engine, Connection, Session 객체에서도 사용할 수 있으며, 특정 실행 옵션을 동적으로 설정하거나 변경하기 위해 사용된다. 

이 메소드를 사용하면 동일한 연결 풀을 공유하면서도 다른 실행 옵션을 가진 엔진을 생성할 수 있다. 

개선된 코드는 다음과 같다. 

_db_connection 설정 후, _db_connection_serializable를 `execution_options()` 를 사용해 정의한다. 

```python
_db_connection_serializable = _db_connection.execution_options(
        isolation_level="SERIALIZABLE",
    )
```

해당 값을 사용한 의존성 주입용 메소드를 선언한다. 

```python
async def get_db_connection_serializable() -> AsyncEngine:
    assert _db_connection_serializable is not None
    return _db_connection_serializable

async def get_serializable_db_session(
    db_conn: AsyncEngine = Depends(get_db_connection_serializable),
) -> AsyncIterable[AsyncSession]:
    session = None
    try:
        async with sessionmaker(
            db_conn,
            class_=AsyncSession,
            expire_on_commit=False,
        )() as session:
            yield session
    except Exception as e:
        logger.error(f"[get_serializable_db_session] {e}")
        if session:
            await session.rollback()
        raise
    finally:
        if session:
            await session.close()
```

이제 엔드포인트에서 기존 get_db_session 대신 이 메소드를 사용하면 다른 옵션의 격리 레벨을 사용할 수 있다!

