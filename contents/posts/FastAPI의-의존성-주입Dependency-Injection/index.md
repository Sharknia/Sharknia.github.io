---
IDX: "NUM-227"
tags:
  - FastAPI
  - Python
description: "FastAPI의 의존성 주입과 친해지기"
update: "2024-12-30T05:39:00.000Z"
date: "2024-12-26"
상태: "Ready"
title: "FastAPI의 의존성 주입(Dependency Injection)"
---
![](image1.png)
## 서론

이번에 FastAPI 과제를 진행하면서 아무생각없이 기존에 짜던대로 코드를 짜고 피드백을 받은 후, 제가 의존성 주입을 할 때 Depends()만 사용하는 방식이 구식 방법이라는 것을 뒤늦게 알게 됐습니다. 심지어 공식문서에도 권장하지 않는 방법이라고 적혀있었습니다. 

아니 정말? 이라는 생각이 들어서 공식문서의 커밋 년도까지 확인을 했는데, 공식 문서가 해당 내용으로 작성된 것은 2023년 중순으로 제 작업 이전이었습니다. 

변명의 여지가 없는 것으로, 곰곰이 생각해보니 의존성 주입이라는 친구와 제가 그다지 친하지 않다는 사실을 깨달았습니다. 예를 들면 매개변수는 어떻게 들어가는지, 이게 다른 의존성이 스윽 들어가고 이런 것들 그냥 아무 생각 없이 쓰던 것들이 사실은 제가 정확히 모르고 있다는 사실을 알게 됐습니다. 

그래서 이번 기회에 의존성 주입에 대해 한 번 정리하고 넘어가려고 합니다. 

## 의존성 주입(Dependency Injenction) 이란?

의존성(dependency)이라는 단어를 한국어로 풀이하면 "다른 것에 기대어 의존하는 상태"를 의미합니다. FastAPI 같은 프로그램에서는 의존성은 어떤 코드가 특정 기능이나 데이터를 수행하기 위해 다른 객체, 함수, 또는 리소스를 필요로 하는 상태를 말합니다.

예를 들면,

```python
from fastapi import Depends
from typing import Annotated

def get_db():
    return "데이터베이스 세션"

@app.get("/items/")
def read_items(db: Annotated[str, Depends(get_db)]):
    return {"db": db}
```

여기서 "read\_items" 함수가 의존성(`get_db`)에 의존합니다.

프로그램에서 데이터베이스 연결이 필요한 코드가 있다고 합시다. 이때 그 코드는 데이터베이스 연결(리소스)에 의존합니다. 즉, "내가 작동하려면 데이터베이스 연결이 있어야 해!"라고 말하는 상황입니다. FastAPI의 의존성 주입(Dependency Injection)은 이러한 의존성을 명확하게 관리하고, 코드의 재사용성과 유지보수성을 높이는 데 목적을 둡니다.

## FastAPI의 의존성 주입의 작동

### 작동 순서

1. 종속성 확인

    - 함수의 매개변수에 `Depends()`가 있는지 확인합니다. 

    - `Depends()`는 호출될 함수를 정의하거나, 리소스를 반환하는 팩토리 역할을 합니다.

1. 함수 호출 전 평가(evaluate)

    - FastAPI는 필요한 의존성을 모두 호출하거나 평가 하여 결과값을 준비합니다.

    - 예를 들어, 데이터베이스 세션을 생성하거나, 인증 정보를 확인합니다.

1. 값 전달

    - 준비된 의존성을 매개변수로 주입하여 실제 라우터, 미들웨어, 이벤트 핸들러 등에서 사용합니다.

1. 범위(Scope) 관리

    - FastAPI는 의존성의 **범위(scope)**를 관리합니다. (`request`, `session`, `singleton` 등)

    - 예를 들어, `request` 범위의 의존성은 요청이 끝나면 자동으로 정리됩니다.

### "의존성을 평가한다"는 의미

FastAPI에서 의존성을 평가(evaluate)한다는 것은 `Depends()`에 지정된 함수나 객체를 실행하거나 처리하여, 해당 결과값을 준비한다는 뜻입니다.

- 평가(evaluation)는 단순히 함수를 호출하는 것을 넘어서, 해당 의존성이 반환하는 값을 준비하고, 필요한 경우 예외를 처리하거나 적절한 스코프를 설정하는 과정을 포함합니다.

#### 예시1: 단순한 함수 의존성

```python
def get_db():
    return "데이터베이스 세션"

@app.get("/items/")
def read_items(db: Annotated[str, Depends(get_db)]):
    return {"db": db}
```

- FastAPI는 `get_db`를 호출하여 결과값 `"데이터베이스 세션"`을 반환합니다.

- “평가”는 여기서 단순히 `get_db()`를 호출하고, 반환값을 준비하는 과정을 의미합니다.

#### 예시2: 의존성 내부에 추가 로직이 있는 경우

```python
def check_auth_header(auth: str = Header(...)):
    if auth != "secret-token":
        raise HTTPException(status_code=401, detail="Unauthorized")
    return auth
```

- 이 경우 FastAPI는 `check_auth_header`를 호출하며, 헤더 값이 유효한지 검증합니다.

- "평가"는 검증 로직이 실행되어 예외가 발생할 수도 있는 전체 과정을 포함합니다.

### 의존성은 반드시 함수여야 할까? 

의존성은 반드시 함수일 필요는 없으며, 호출 가능한 객체도 사용할 수 있습니다. FastAPI의 `Depends()`는 **함수뿐 아니라 호출 가능한 객체(callable)**도 의존성으로 사용할 수 있습니다. 호출 가능한 객체란, `__call__` 메서드를 구현한 클래스 인스턴스를 의미합니다.

#### 예시1: 호출 가능한 클래스

```python
class DBSession:
    def __call__(self):
        return "데이터베이스 세션"

@app.get("/")
def read_root(db: Annotated[str, Depends(DBSession())]):
    return {"db": db}
```

- 여기서 `DBSession` 클래스는 호출 가능한 객체로 동작하며, `__call__` 메서드가 실행됩니다.

- FastAPI는 `DBSession`의 인스턴스를 평가하여, `__call__`의 반환값 `"데이터베이스 세션"`을 준비합니다.

#### 예시2: 상수 값 (Callable이 아님)

일반적으로 의존성은 **함수 또는 호출 가능한 객체**를 사용하는 것이 일반적이지만, 다음과 같은 객체도 사용할 수 있습니다.

```python
db_config = {"host": "localhost", "port": 5432}

@app.get("/")
def read_root(config: Annotated[str, Depends(lambda: db_config)]):
    return config
```

- 여기서 `db_config`는 상수 값이지만, 람다 함수 `lambda: db_config`로 감싸서 의존성으로 주입할 수 있습니다.

### FastAPI 의존성의 범위(Scope)

#### 범위란? 

범위는 의존성이 언제 생성되고 언제 소멸되는지를 정의합니다. FastAPI는 요청(Request) 기반의 애플리케이션이므로 일반적으로는 요청이 처리되는 동안 의존성이 유지되다가 요청이 끝날 때 정리됩니다.

#### 주요 범위(scope) 종류

1. `request` 범위 (기본값)

    - 요청이 들어오면 의존성이 생성되고, 요청이 끝나면 소멸됩니다.

    - 대부분의 의존성은 기본적으로 `request` 범위를 가집니다.

    ```python
    def get_db():
        db = "데이터베이스 세션 생성"
        try:
            yield db
        finally:
            print("데이터베이스 세션 종료")
    
    @app.get("/")
    def read_items(db: Annotated[str, Depends(get_db)]):
        return {"db": db}
    ```

    - `get_db`는 요청이 들어올 때 실행되고, 요청이 끝날 때 `finally` 블록이 실행되어 세션이 종료됩니다.

    - 요청이 끝난 후 해당 의존성(`db`)은 더 이상 사용되지 않습니다.

1. `singleton` **** 범위

    - 애플리케이션이 시작된 후 종료될 때까지 의존성을 단 한 번만 생성하고 유지합니다.

    - FastAPI에서 특정 경우 `@lru_cache`를 사용해 싱글톤 동작을 흉내낼 수 있습니다.

    ```python
    from functools import lru_cache
    
    @lru_cache
    def get_config():
        return {"key": "value"}  # 설정 정보
    
    @app.get("/")
    def read_items(config: Annotated[str, Depends(get_config)]):
        return config
    ```

    - `get_config`는 애플리케이션 생명주기 동안 단 한 번 실행됩니다.

    - 이후 모든 요청은 같은 객체를 재사용합니다.

1. `session` 범위

    - 특정 세션 동안 유지되는 의존성을 정의할 때 사용됩니다.

    - FastAPI에서 기본적으로 제공하지 않지만, `Dependency Injection Containers`(의존성 관리 라이브러리)를 사용해 구현할 수 있습니다.

    - 하지만 FastAPI는 기본적으로 RESTful 아키텍처를 따르기 때문에, 요청(Request) 단위를 기본 범위(scope)로 처리합니다. 각 요청은 독립적으로 처리되며, 요청이 끝나면 관련 자원(예: DB 세션, 인증 정보 등)이 정리됩니다. 따라서 일반적인 FastAPI 애플리케이션에서는 세션 범위(scope)를 따로 고려할 필요가 없습니다.

    - 다만, WebSocket, 장기 실행 작업, 상태를 유지해야 하는 특수한 경우에는 세션 스코프를 고려할 수 있습니다.

#### lru\_cache

`lru_cache`는 Python의 내장 데코레이터로, 함수의 반환값을 캐싱하여 이후 호출 시 동일한 결과를 반환하는 데 사용됩니다. 이를 활용해 싱글톤 객체를 쉽게 구현할 수 있습니다.

```python
from functools import lru_cache

@lru_cache
def get_config():
    return {"db_host": "localhost", "db_port": 5432}

@app.get("/")
def read_config(config: Annotated[str, Depends(get_config)]):
    return config
```

FastAPI에서 설정 정보나 리소스와 같이 **상태를 유지하지 않는 객체**를 공유할 때는 `lru_cache`를 사용하는 것이 훨씬 더 일반적입니다.

직접 싱글톤을 구현하는 경우는 다음과 같은 상황에서 주로 사용됩니다.

1. 복잡한 초기화 로직: 초기화 과정에서 외부 리소스를 다루거나 복잡한 설정을 적용해야 할 때.

1. 상태를 유지해야 하는 경우: 싱글톤 객체가 내부적으로 상태를 관리해야 할 경우.

### 의존성 사용시 주의점

보통 의존성은 Router Function에서 사용합니다. Request의 시작과 동시에 의존성을 사용하는게 일반적입니다. 하지만 의존성 주입이 라우터에서만 가능한 것은 아닙니다. 

따라서 라우터가 아닌 서비스 레이어에서 직접 의존성을 주입해 사용(클래스등을 활용)하는 것이 기술적으로 가능하지만, FastAPI의 **의존성 관리 철학**과 **응집성/분리성**의 원칙에 따라 판단해야 합니다. 

일반적으로, **라우터에서 의존성을 주입한 후 서비스에 전달하는 방식**이 권장됩니다. 이는 의존성을 명확히 관리하고, 서비스 레이어를 테스트하기 쉽고, FastAPI와의 결합도를 줄이는 데 유리하기 때문입니다.

## 결론

의존성 주입은 반복되는 코드를 획기적으로 줄이면서도 결합도가 떨어져 유지보수가 굉장히 쉬워지고, 테스트코드를 짤 때에도 독립된 코드이므로 훨씬 수월해지므로 장점이 많습니다. 

다만 의존성이 지나치게 중첩되면 코드의 복잡도가 증가할 수 있어 주의가 필요합니다. 

