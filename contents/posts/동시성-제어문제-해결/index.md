---
tags:
  - DataBase
  - Postgresql
  - Work
description: "PostgreSQL Advisory Locks"
update: "2024-01-30"
date: "2024-01-30"
상태: "POST"
title: "동시성 제어문제 해결"
---
## 문제

가~끔 네트워크 문제 때문인지 클라이언트에서 같은 요청이 0.01초 미만의 간격으로 두 번씩 들어오는 경우가 있다. 사실상 요청이 동시에 들어오는 것과 같다. 대부분의 경우에는 문제가 되지 않지만, 쿠폰 구매 같은 민감한 요청에 대해서는 회사의 손해 또는 사용자의 불편과 예민하게 직결되므로 문제가 커질 수 있다. 

따라서 해당 문제를 완벽히 해결하고자 한다. 

현재 별다른 조치가 없는 상황에서 발생하는 문제 상황은 다음과 같다. 

```plain text
1번 요청과 2번 요청이 0.005초 차이이고(사실상 동시),

1번 요청 - 트랜잭션 시작
2번 요청 - 트랜잭션 시작
1번 요청 - 트랜잭션 완료, 커밋. 새로운 구매 내역이 테이블에 추가됨.
2번 요청 - 트랜잭션 완료, 커밋. 새로운 구매 내역이 테이블에 추가됨. 

따라서 동일한 유저가 두 번 쿠폰을 구매한 것 처럼 처리되었다. 
```

### 제약조건 확인

현재 테이블의 키값이 user\_id가 아니며, 난수 발생을 통해 생성된 값을 키 값으로 하고 있다. 또한 Redis와 같은 외부 라이브러리를 도입할 여유는 없다. 따라서 가능한 한 현재 시스템(Sqlalchemy, Postgresql)내에서 해결책을 찾고자 한다. 

## 해결방안 고민

### 최근 구매 내역 확인(기각)

가장 쉽게 생각할 수 있는 방법은 쿠폰 구매 로직에서 이 유저의 최근 구매 로직을 확인하는 것이다. 하지만 현재 이 상황에서는 사실상 요청이 동시에 들어오고 있으므로, 2번 요청이 시작됐을 때에는 해당 유저의 구매 내역이 아직 DB에 없을 것이므로, 이 방법은 사용할 수 없다. 

### Database-Level Optimistic Locking(기각)

각 트랜잭션에 버전 번호를 추가하고 트랜잭션이 커밋되기 전에 해당 버전 번호가 여전히 유효한지 확인한다. 하지만 이 방법도 여전히 2번 요청이 1번 요청의 결과를 감지하지 못할 수 있다. 

### Application-Level Locking(기각)

코드 내에서 직접 잠금 로직을 구현하는 방법이다. 데이터베이스가 아닌 애플리케이션의 메모리 내에서 잠금을 관리한다. 주로 멀티스레드 환경에서 특정 자원에 대한 동시 접근을 제어하는데 사용된다. 현재 FastAPI를 사용하고 있고, FastAPI는 기본적으로 비동기 방식으로 단일 스레드 환경에서 실행된다는 점을 고려하면 전통적인 스레드 기반의 잠금 매커니즘보다 async 라이브러리에 들어있는 비동기 프로그래밍에 적합한 잠금 메커니즘인 asyncio.Lock을 사용하는 것이 이상적이다.

#### FastAPI에서의 Application-Level Locking

```python
import asyncio

locks = {}

async def purchase_coupon(user_id: int):
    lock = locks.setdefault(user_id, asyncio.Lock())

    async with lock:
        # 여기에 쿠폰 구매 로직을 구현합니다.
        # 이 블록 내의 코드는 동시에 하나의 요청만 처리합니다.
        pass

# FastAPI 엔드포인트에서 이 함수를 호출합니다.
```

이 코드는 user\_id에 대해 별도의 asyncio.Lock을 생성하고 관리한다. 해당 user\_id에 대한 작업이 진행중인 경우 다른 요청들은 해당 블록이 해제될 때까지 대기한다. 

예를 들어 다음 코드를 가정하자. 예를 들기 위해 아주 러프하게 작성했다. 

```python
async def test_function(a: int, b: int):
    lock = locks.setdefault(a, asyncio.Lock())

    async with lock:
        await asyncio.sleep(1)
        print(a)
```

이 코드는 다음과 같이 호출된다. 

```plain text
test_function(1, 2) 호출 -> a 값이 1인 잠금 획득 -> 처리 시작
test_function(1, 2) 호출 -> a 값이 1인 동일한 잠금으로 인해 대기
test_function(2, 2) 호출 -> a 값이 2인 새로운 잠금 획득 -> 처리 시작
test_function(2, 2) 처리 완료 -> a 값이 2인 잠금 해제
test_function(1, 2)의 첫 번째 호출 처리 완료 -> a 값이 1인 잠금 해제
test_function(1, 2)의 두 번째 호출이 대기 상태에서 해제 -> 처리 시작

따라서 print는 1, 2, 1의 순서가 된다. 
```

다만, 비동기 프로그래밍, 특히 Python의 [asyncio](https://sharknia.github.io/FastAPI와-asyncio)를 사용할 때 코드의 실행순서는 이벤트 루프에 의해 관리된다. 작업의 일시 중단과 다른 작업의 실행이 이벤트 루프에 의해 어떻게 스케줄링 될지는 여러 요인에 따라 달라질 수 있다. 

### PostgreSQL Advisory Locks(당장 채용)

Advisory Locks는 애플리케이션에서 데이터베이스 레벨의 잠금을 관리할 수 있도록 해준다. 이를 통해 특정 user\_id 에 대한 동시 요청을 제어할 수 있다. 

Advisory Locks는 테이블이나 행에 대한 접근을 잠그는 개념이 아니다. 특정 값 (정수)에 대한 잠금을 제공한다. 즉, 작업 자체에 1번이라는 이름을 붙이고 1번을 잠근다면, 다시 요청되는 1번 작업은 잠금이 된다. 

#### **Advisory Locks의 작동 방식**

- 잠금 설정 : 애플리케이션이 pg\_advisory\_lock(key) 함수를 호출하여 특정 키에 대한 잠금을 요청한다.

- 동일 키 사용 시 대기 : 같은 키를 사용하는 다른 데이터베이스 작업이 잠금을 보유하고 있다면, 새로운 잠금 요청은 해당 잠금이 해제될 때까지 대기한다. 

- 잠금 해제 : 원래의 작업이 완료되면 pg\_advisory\_unlock(key) 함수를 호출하여 잠금을 해제한다. 이후 대기 중이던 다른 작업이 잠금을 획득하고 진행될 수 있다. 

#### **Advisory Locks의 잠금 레벨**

Advisory Locks는 두 가지 레벨에서 사용할 수 있다. 

- 세션 레벨 : 이 잠금은 데이터베이스 세션과 연결되어있다. 세션이 종료되면 자동으로 해제된다. 세션 레벨 잠금은 장시간 유지되어야 하는 경우에 유용하다. 

    ```python
    async def use_session_level_lock(db: AsyncSession, lock_key: int):
        # 세션 레벨 잠금 획득
        await db.execute(f"SELECT pg_advisory_lock({lock_key})")

        # 데이터베이스 작업 수행
        # ...

        # 필요한 경우, 명시적으로 잠금 해제
        # await db.execute(f"SELECT pg_advisory_unlock({lock_key})")

        # 세션 종료 시 잠금이 자동으로 해제됩니다.
    ```

- 트랜잭션 레벨 잠금 : 트랜잭션 레벨 잠금은 현재 트랜잭션과 연결되어 있으며 트랜잭션이 커밋되거나 롤백 될 때 자동으로 해제된다. 

    ```python
    from sqlalchemy.ext.asyncio import AsyncSession

    async def use_transaction_level_lock(db: AsyncSession, lock_key: int):
        async with db.begin():
            # 트랜잭션 레벨 잠금 획득
            await db.execute(f"SELECT pg_advisory_xact_lock({lock_key})")
            
            # 데이터베이스 작업 수행
            # ...

        # 트랜잭션이 종료되면 잠금이 자동으로 해제됩니다.
    ```

쿠폰 구매 서비스 로직에서 트랜잭션 시작 전에  user\_id를 기반으로 Advisory Lock을 요청하면 1번 요청이 처리되는 동안 동일한 user\_id 에 대한 다른 요청은 대기하게 된다. 쿠폰 구매 처리가 완료되면, Advisory Lock을 해제하면 된다. 

## 결론 - PostgreSQL Advisory Locks 채용

이 방법을 사용하면 매우 짧은 간격의 요청도 효과적으로 처리할 수 있다. 특히 트랜잭션 레벨의 잠금은 각 트랜잭션이 커밋되거나 롤백될 때까지만 유지되므로 0.005초와 같은 짧은 간격의 요청을 동기화하는데 매우 적합하다. 

백엔드 팀에서 협의를 거쳐 해당 방법을 사용해 구현하기로 최종 결정하였다. 

