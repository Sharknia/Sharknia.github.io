---
tags:
  - Postgresql
  - SqlAlchemy
  - Python
  - DataBase
  - Work
description: "Sqlalchemy 2.0에서 PostgreSQL Advisory Locks을 트랜잭션 레벨에서 구현해 동시성 제어를 하자. "
update: "2024-01-31"
date: "2024-01-30"
상태: "Ready"
title: "PostgreSQL Advisory Locks 트랜잭션 레벨에서 구현"
---
## 서론

[동시성 제어 문제를 해결하기 위한 여러가지 방법을 고민](https://sharknia.github.io/2024-01-30/동시성-제어문제-해결) 끝에 PostgreSQL Advisory Locks를 사용한 방법으로 구현하기로 했다. 

구현을 하면서 겪은 과정을 여기에 기록한다. 

## 구현 목표

현재 회사에서는 FastAPI와 SqlAlchemy 2.0, PostgreSQL을 사용하고 있다. 아주 짧은 텀으로 중복으로 온 API 요청에 대해 중복 구매가 되지 않도록 동시성 제어를 하려고 한다. 

유저의 아이디 값 또는 API 고유의 값 + 유저의 아이디 값을 사용해 트랜잭션 레벨에서 PostgreSQL Advisory Locks를 생성하고, 동일한 유저가 PostgreSQL Advisory Locks에 걸린 경우에는 중복 요청이라고 판단하고 두번째 요청은 중단 하거나, 또는 반드시 유효성 검사를 하도록 해 중복 구매가 되는 일이 없게 사전에 차단하려고 한다. 

## 구현 사전단계

[공식문서](https://www.postgresql.org/docs/9.1/functions-admin.html)를 살펴보니 트랜잭션 레벨에서의 락을 위해 내가 사용할 수 있는 함수는 두 가지가 있었다. 트랜잭션 레벨의 락은 공통적으로 명시적인 언락이 불가능하다. 

아래의 두가지 함수는 모두 트랜잭션 레벨의 함수로 Advisory Lock을 획득할 수 있다는 점은 같지만, Advisory Lock을 바로 획득하지 못했을 때의 반응이 다르다. 

### pg_advisory_xact_lock

지정된 키에 대한 Advisory Lock을 획득할 때까지 호출을 블로킹(대기)한다. 만약 다른 세션이 이미 해당 키에 대한 Advisory Lock을 갖고 있다면 그 Advisory Lock이 해제될 때까지 현재 세션에서의 처리가 중단된다. 

Advisory Lock을 반드시 획득해야 해서 대기해야 할 경우에 적합하다.

### pg_try_advisory_xact_lock

이 함수는 즉시 Advisory Lock을 시도하고 성공하면 True, 실패하면 False를 반환한다. 이 함수는 잠금 획득을 위해 대기하지 않으며, 잠금이 이미 다른 세션에 의해 보유되고 있는 경우 즉시 실패한다.

Advisory Lock을 반드시 획득할 필요가 없고 대기하지 않고 다른 작업을 수행해야 할 경우에 적합하다. 

### 결정

우리의 경우에는 쿠폰 중복 발급을 막으려고 하는 것이므로 만약 이미 잠금이 걸린 경우에는 굳이 나머지 DB 작업을 실행할 필요가 없다. 따라서 pg_try_advisory_xact_lock를 사용하는 것이 적절해보인다. 

## 구현

따라서, 서비스 로직의 시작 지점에 다음의 코드를 넣어 락을 얻거나, 락을 얻지 못한 경우 해당 요청은 실행을 중단하려고 한다. 

```python
# Advisory Lock 획득
    result = await db.execute(f"SELECT pg_try_advisory_xact_lock({lock})")
    # Advisory Lock을 획득하지 못한 경우에는 이미 선제 작업이 있는 것이므로 DB 작업을 계속할 필요가 없으므로
    # 바로 중단한다.
    if not result.scalar():
        raise Exception("Unable to acquire lock")
```

SQL 인젝션과 같은 보안 문제를 줄이고 쿼리 구성과 관련된 오류를 예방하기 위해 다음과 같이 코드를 수정했다. 

```python
# Advisory Lock 획득
    result = await db.execute(func.pg_try_advisory_xact_lock(100, lock))
    # Advisory Lock을 획득하지 못한 경우에는 이미 선제 작업이 있는 것이므로 DB 작업을 계속할 필요가 없으므로
    # 바로 중단한다.
    if not result.scalar():
        raise Exception("Unable to acquire lock")
```

## 잠금 해제 지점은 어떻게 될까? 

막연히 트랜잭션이 커밋되거나 롤백될 때라고 알아둬도 충분할 것 같다(세션 주입 방식을 통해 에러가 발생하거나 요청이 정상정이 끝나지 않는 경우에 세션을 롤백하며, 정상적으로 요청이 정리된 경우에는 세션을 커밋하는 과정이 포함되어 있을 것이므로 충분할 것이다).

다만, 트랜잭션 레벨에서의 락은 명시적으로 언락이 불가능하므로 정확한 언락 지점을 알아둬야 할 필요성이 있다고 느꼈다. 

- 트랜잭션 커밋 또는 롤백

    함수 내부에서 명시적으로 await db.commit() 또는 await db.rollback()이 호출되는 경우에 같은 세션이라고 하더라도 언락된다. 

- 비동기 세션 컨텍스트 종료

    AsyncSession 인스턴스가 컨텍스트 매니저(async with) 내에서 사용되고, 해당 컨텍스트 블록이 종료되는 경우

- 예외 발생

    함수 실행 중 예외가 발생하여 처리 흐름이 중단되는 경우. SQLAlchemy의 세션 관리는 트랜잭션이 커밋되지 않으면 자동으로 롤백을 수행하므로 사실상 트랜잭션이 롤백되는 시점과 일치한다. 



