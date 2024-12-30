---
IDX: "NUM-228"
tags:
  - FastAPI
  - Python
description: "FastAPI 의존성 주입의 심화 활용법과 주의점"
update: "2024-12-30T05:42:00.000Z"
date: "2024-12-30"
상태: "Ready"
title: "FastAPI 의존성 주입의 심화 활용법과 주의점"
---
![](image1.png)
## 의존성 간 관계 자동 해결

FastAPI는 의존성 간의 관계를 자동으로 해결해 주기 때문에 개발자는 복잡한 로직을 작성하지 않아도 FastAPI가 필요한 리소스를 적절히 연결해줍니다. 

### FastAPI 의존성 주입의 기본 개념

의존성은 서로 다른 의존성을 참조할 수 있으며, FastAPI는 이를 바탕으로 **의존성 그래프**를 생성해 자동으로 해결합니다.

```python
from fastapi import Depends, FastAPI

app = FastAPI()

async def get_db_session() -> str:
    return "DB 세션"

async def get_redis_client(db_session: Annotated[str, Depends(get_db_session)]) -> str:
    return f"Redis 클라이언트 (DB: {db_session})"

@app.get("/")
async def read_data(redis_client: Annotated[str, Depends(get_redis_client)]) -> str:
    return {"redis_client": redis_client}
```

- `get_db_session`: DB 세션 문자열을 반환하는 의존성.

- `get_redis_client`: `get_db_session`의 반환값을 의존성으로 사용.

- `read_data` 엔드포인트: `get_redis_client`의 반환값을 의존성으로 사용.

FastAPI는 `read_data` → `get_redis_client` → `get_db_session` 순서로 의존성을 파악해 의존성 그래프를 생성합니다. 가장 하위 의존성(`get_db_session`)부터 실행하여 결과를 상위로 전달합니다. 최종적으로 `get_redis_client`의 결과를 `read_data` 함수로 전달합니다.

### 의존성 간 관계 자동 해결의 장점

- 복잡한 의존성 관리 단순화

    - FastAPI는 의존성 간의 관계를 자동으로 파악하고 올바른 순서로 호출합니다.

    - 개발자는 의존성 간의 호출 순서를 걱정하지 않아도 됩니다.

- 유연한 확장 가능성

    - 새로운 의존성을 추가하거나 기존 의존성을 수정해도 FastAPI가 자동으로 관계를 재구성합니다.

### 의존성 간 관계 자동 해결시 주의점

의존성 함수가 서로를 참조하는 경우(`A → B → A`), FastAPI는 순환 참조를 해결하지 못합니다.

```python

async def get_a(b: Annotated[str, Depends(get_b)]):
    return "A"

async def get_b(a: Annotated[str, Depends(get_a)]):
    return "B"
```

중간 레벨의 의존성을 생성하거나 의존성을 리팩토링해 순환을 제거해야 합니다.

## 의존성에 매개변수 전달하기

FastAPI에서 의존성 함수에 직접 값을 전달하는 기능은 없지만, 아래와 같은 방법으로 유사한 동작을 구현할 수 있습니다.

### 람다(Lambda) 함수로 전달

의존성 함수가 동적으로 다른 값을 받을 수 있도록 람다를 사용합니다.

```python
from fastapi import Depends, FastAPI

app = FastAPI()

def get_value(value: int):
    return f"Value is {value}"

@app.get("/")
def read_root(custom_value: Annotated[str, Depends(lambda: get_value(42))]):
    return {"custom_value": custom_value}
```

- `lambda: get_value(42)`를 통해 `42`를 매개변수로 전달.

- `Depends()`는 동적으로 생성된 값을 사용할 수 있습니다.

### 의존성 팩토리 함수

의존성 함수를 팩토리 함수로 감싸서 동적으로 값을 전달합니다.

```python
from fastapi import Depends, FastAPI

app = FastAPI()

def get_value_factory(value: int):
    def get_value():
        return f"Value is {value}"
    return get_value

@app.get("/")
def read_root(custom_value: Annotated[str, Depends(get_value_factory(42))]):
    return {"custom_value": custom_value}
```

- `get_value_factory(42)`는 `get_value`를 반환하며, 내부적으로 `42`를 사용할 수 있습니다.

- 이 방법은 더 복잡한 로직에서 유용합니다.

### 의존성 컨텍스트 전달 (Request State 활용)

FastAPI의 `Request.state`를 사용하여 동적으로 데이터를 전달할 수 있습니다.

```python
from fastapi import Depends, FastAPI, Request

app = FastAPI()

async def get_custom_value(request: Request):
    return request.state.custom_value

@app.middleware("http")
async def add_custom_value(request: Request, call_next):
    request.state.custom_value = 42
    response = await call_next(request)
    return response

@app.get("/")
async def read_root(custom_value: Annotated[str, Depends(get_custom_value)]):
    return {"custom_value": custom_value}
```

- 미들웨어를 사용해 `Request.state`에 값을 추가.

- 의존성 함수에서 `Request.state` 값을 참조.

### 의존성 클래스 활용

의존성 클래스를 사용해 동적으로 상태를 전달할 수 있습니다.

```python
class ValueProvider:
    def __init__(self, value: int):
        self.value = value

    def __call__(self):
        return f"Value is {self.value}"

@app.get("/")
def read_root(custom_value: Annotated[str, Depends(ValueProvider(42))]):
    return {"custom_value": custom_value}
```

- `ValueProvider` 클래스는 생성자에서 값을 받고, 호출 시 이 값을 반환.

- FastAPI의 `Depends`는 호출 가능한 객체를 의존성으로 사용할 수 있으므로 동적으로 값을 전달 가능.

### 주의점

일반적으로 FastAPI의 의존성 주입 시스템을 우회해서 매개변수를 전달하는 방식(람다 함수나 팩토리 함수 등)은 유지보수성과 가독성 측면에서 좋지 않을 수 있습니다. 이러한 방법은 특정 상황에서는 유용하지만, 남용할 경우 프로젝트의 구조를 복잡하게 만들고 디버깅을 어렵게 할 수 있습니다.

람다 함수, 팩토리 함수 등 우회 방법은 특수한 상황에서만 신중히 사용해야 합니다.

가능한 경우, 동적 값을 처리하는 로직은 상위 레벨(미들웨어, 라우터)에서 관리하고 의존성 함수는 단순화하는 것을 권장합니다.

## `Annotated` 를 활용한 의존성의 선언

의존성의 선언에서 `Annotated`는 0.95 버전 이후에 추가된 기능으로, 타입 힌트를 더 명확히 표현하고 코드를 더 읽기 쉽게 만들어줍니다. FastAPI에서 `Annotated`를 사용하는 것은 가독성 외에도 타입 안정성, 코드 재사용성, 확장 가능성 등 여러 측면에서 이점을 제공합니다.

### 코드 가독성 및 명확성

`Annotated`를 사용하면 의존성의 역할을 타입 힌트를 통해 명확히 나타낼 수 있습니다. 코드의 의도를 더 직관적으로 파악할 수 있으므로, 팀원 간 협업이나 유지보수 시 유리합니다.

```python
from typing import Annotated
from fastapi import Depends

AdminUser = Annotated[str, Depends(admin_permissions)]

@app.get("/admin")
def admin_only_route(user: AdminUser):
    return {"message": f"Hello, {user}"}
```

- `AdminUser`: 타입 힌트를 통해 이 변수가 관리자의 인증과 권한 검증을 포함한다는 점을 명확히 전달

- 가독성과 의미 전달이 좋아져 코드 리뷰와 유지보수가 용이

### 타입 안정성 및 도구 지원

`Annotated`는 타입 힌팅을 강화하여, IDE의 자동 완성과 정적 분석 도구(`mypy` 등)가 더 정확하게 동작하도록 도와줍니다. 복잡한 의존성 체계를 사용할 때 코드 안정성을 높이는 데 기여합니다.

```python
AdminUser = Annotated[str, Depends(admin_permissions)]
```

- IDE에서 `AdminUser`를 사용할 때 자동 완성 기능으로 `Depends(admin_permissions)`가 연결됨.

- `mypy`와 같은 도구는 타입 충돌을 방지하고 코드 품질을 유지하는 데 도움을 줌.

### 확장 가능성

타입 힌트와 의존성을 결합하여 더 복잡한 구조나 동작을 설계할 수 있습니다. 데이터를 검증하거나 추가적인 메타데이터를 전달하는 데 사용할 수 있습니다.

```python
from pydantic import BaseModel
from typing import Annotated
from fastapi import Depends

class AdminPayload(BaseModel):
    action: str

AdminRequest = Annotated[AdminPayload, Depends(admin_permissions)]

@app.post("/admin/action")
def perform_admin_action(payload: AdminRequest):
    return {"action_performed": payload.action}
```

- `AdminRequest`: 관리자의 인증/인가와 함께 데이터를 검증하는 구조로 확장 가능.

- 인증/인가와 데이터 검증을 결합하여 더욱 강력한 의존성 관리 가능.

### 모든 의존성을 Annotated로 바꿔야 할까? 

아니요, 기존 방식(`Depends`)도 완벽히 동작합니다. Annotated는 더 명확한 타입 힌팅과 가독성을 제공하지만, 필수는 아닙니다.



