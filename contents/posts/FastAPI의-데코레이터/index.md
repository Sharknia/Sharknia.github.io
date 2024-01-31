---
tags:
  - FastAPI
  - Python
update: "2024-01-29"
date: "2023-11-03"
상태: "POST"
title: "FastAPI의 데코레이터"
---
## 개요

FastAPI의 데코레이터는 파이썬 데코레이터 패턴을 활용하여 FastAPI 프레임워크에서 제공하는 여러 기능을 함수나 클래스에 적용하는 구문이다. 이 데코레이터들은 FastAPI에서 매우 중요한 역할을 한다. 

데코레이터는 `@` 기호를 사용하여 함수나 클래스의 위에 선언된다. 데코레이터는 그 아래에 정의된 함수에 추가적인 기능을 부여하거나 특정 작업을 수행하도록 지시한다. 

데코레이터를 사용하여 개발자는 복잡한 로직을 함수에 직접 쓰지 않고 프레임워크가 제공하는 데코레이터를 사용하여 빠르고 쉽게 웹 애플리케이션을 구현할 수 있다. 

## 주요 데코레이터

### `@app.middleware("http")`

HTTP 요청-응답 사이클에 관여하는 미들웨어를 등록하는데 사용된다.  이 데코레이터 아래에 정의된 함수는 애플리케이션으로 들어오는 모든 http 요청에 대해 처리되고 그리고 해당 요청에 대한 응답을 반환하기 전에 호출된다. 


        미들웨어란?
요청과 응답을 처리하는 과정 사이에 위치하여 들어오는 요청을 가로채 그 요청에 대해 특정 작업을 수행하거나 응답을 조작하는 구성 요소이다. 

#### 예제

```python
@app.middleware("http")
async def custom_middleware(request: Request, call_next):
    # 요청 전에 실행할 코드
    response = await call_next(request)
    # 응답 전에 실행할 코드
    return response
```

크게 두 부분으로 나눠진다. 

- 요청 전에 실행할 코드 : `call_next` 함수에 요청을 전달하기 전에 실행할 코드를 작성한다. 

- 응답 전에 실행할 코드 : `await call_next(request)` 는 다음 미들웨어나 실제 요청을 처리하는 엔드포인트를 호출한다. 이후 응답이 반환되면 그 응답에 추가적인 처리를 하고 싶을 때 사용할 수 있다. 

#### 미들웨어 체인

만약 `@app.middleware("http")` 가 여러개 정의되어 있다면 FastAPI는 그것을 선언된 순서대로 실행한다. 각 미들웨어는 이전 미들웨어에서 `await call_next(request)` 를 호출한 후의 응답을 받아 처리한다. 이를 미들웨어 체인이라고 하며, 요청이 엔드포인트에 도달하기 전에 여러 미들웨어를 통과한다. 

따라서 미들웨어는 가벼운 로직을 수행하는 것이 좋으며, 무거운 작업은 미들웨어에서 피해야 한다. 또한 각 미들웨어는 만드시 `await call_next(request)` 를 호출하여 체인을 계속 진행할 수 있도록 해야 한다. 

### `@app.get`, `@app.post`, `@app.put`, `@app.delete`, `@app.options`, `@app.head`

HTTP 메소드에 맞게 라우트를 설정하는 데코레이터이다. 

각각의 데코레이터는 해당 함수가 지정된 HTTP 메서드의 요청을 처리하는 엔드포인트임을 알려준다. 엔드포인트 함수 내부에서는 파라미터 검증, 비즈니스 로직, 데이터 반환 등의 작업을 수행할 수 있다. 

```python
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

### `@app.api_route`

모든 http 메소드를 하나의 함수로 라우트 할 수 있게 해주는 데코레이터이다. 

하나의 엔드포인트에 여러 http 메소드를 지정할 수 있도록 한다. 예를 들어 같은 경로에 대해 Get과 Post 요청을 모두 처리하고 싶은 경우에 사용할 수 있다. 이 데코레이터를 사용하면 각 메서드에 대한 처리 로직을 한 함수에서 정의할 수 있다. 

```python
@app.api_route("/items/{item_id}", methods=["GET", "POST"])
async def handle_item(item_id: int):
    # 여기에 GET과 POST를 처리하는 로직을 구현
    pass
```

### `@app.websocket`

웹소켓 연결을 처리하는 엔드포인트를 선언하는 데코레이터이다. 클라이언트가 해당 경로로 웹소켓 연결을 시도하면 FastAPI는 해당 함수를 실행해 웹소켓 핸드셰이크를 처리하고 연결을 유지한다. 

### `@app.on_event("startup" | "shutdown")`

애플리케이션의 시작 시 또는 종료 시 실행할 함수를 등록하는 데코레이터이다. 

### `@app.exception_handler(Exc)`

특정 예외를 처리하는 핸들러를 등록하는 데코레이터이다. 특정 예외 유형이 발생했을 때 실행될 커스텀 핸들러를 등록하는 데 사용된다. 표준 예외 로직을 오버라이드 하거나 특정 예외 유형에 대해 특별한 처리를 구현할 수 있다. 

예를 들어 ValueError가 발생했을 때, 표준 HTTP 500 Error 대신 더 구체적인 오류 메세지와 HTTP 400 코드를 반환할 수 있다. 

#### 예제

```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import JSONResponse
from starlette.requests import Request

app = FastAPI()

@app.exception_handler(ValueError)
async def value_error_exception_handler(request: Request, exc: ValueError):
    return JSONResponse(
        status_code=400,
        content={"message": str(exc)},
    )

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id < 0:
        raise ValueError("Item ID must be positive")
    return {"item_id": item_id}
```

이를 이용해 다음과 같은 이점을 얻을 수 있다. 

1. 유저 친화적인 오류 메시지 제공

1. 로그 이록

1. 오류 리포팅

1. 커스텀 http 상태 코드 반환 : 기본적으로 변경된 메시지 대신 사용자에게 안내를 줄 수 있다. 

이러한 예외 핸들러는 API 의 로버스트성을 늘려준다. 


        로버스트성이란?
소프트웨어가 예기치않은 입력이나 사용 상황에서도 안정적으로 작동하는 성질을 의미한다. 

### `@app.dependency`

함수가 종속성으로 작동하게 하며 해당 함수가 다른 경로 작업에서 호출될 때마다 실행되게 한다. 이를 이용해 공통 기능을 중앙에서 관리하고 경로 작업에서 필요한 데이터를 제공하거나 사전 처리를 수행할 수 있다. 

#### 예제

모든 경로에서 공통으로 사용되는 데이터베이스 세션을 생성하는 경우를 가정하자. 아래는 해당 데코레이터를 사용하여 데이터베이스 세션을 경로에 주입하는 예제이다. 

```python
from fastapi import FastAPI, Depends

app = FastAPI()

class DBSession:
    # DBSession 클래스는 데이터베이스 세션을 관리합니다.
    def __init__(self):
        self.session = "DB Connection"

    def close(self):
        self.session = "DB Connection Closed"

# 종속성으로 사용될 함수
@app.dependency
async def get_db_session():
    db_session = DBSession()
    try:
        yield db_session.session
    finally:
        db_session.close()

# 경로 작업에서 종속성 사용
@app.get("/items/")
async def read_items(db: str = Depends(get_db_session)):
    return {"db_session": db}
```

get_db_session 함수가 @app.dependency 데코레이터로 마크되어있다. 이 함수는 호출될 때마다 새로운 DBSession 인스턴스를 생성하고 요청 처리가 완료된 후 세션을 정리한다. 

이 함수는 경로 작업 함수 read_items에 Depends를 사용하여 주입된다. 경로 작업에서는 반환된 데이터베이스 세션을 db라는 변수로 받아 사용할 수 있다. 

Depends를 사용함으로써 FastAPI는 get_db_session 함수를 실행하고 그 반환값을 read_items 경로 작업의 매개변수로 전달한다. 이 패턴은 서비스 계층, 데이터 접근 계층 등에서 특히 유용하며 코드 중복을 줄이고 테스트 용이성을 높여준다. 

FastAPI에서 해당 데코레이터를 사용해 정의된 함수는 생성기(generator) 패턴을 사용한다. yield 키워드를 사용해 이 함수는 값을 반환하기 전과 후에 코드를 실행할 수 있다. 

1. yield를 만날 때까지 함수를 실행한다. 이 때, DBSession 인스턴스가 생성되고, 세션이 초기화된다. 

1. yield에서 함수는 호출한 측에 db_session.session 값을 넘겨주고 실행을 일시 중지(pause) 한다. 

1. 이제 read_items 경로 함수의 본문을 실행한다. 이 때, db 매개변수로 전달된 값을 사용한다. 

1. read_items 함수가 완료되고 응답이 반환되면 yield문 이후의 코드가 실행된다. 이 코드에서는 finally 블록이다. 

즉 finally 블록은 http 요청 처리가 완전히 끝나고 응답이 클라이언트에게 전송된 후에 실행된다. 이는 DBSession 객체의 리소스를 안전하게 정리할 수 있게 해준다. yield를 사용하는 이 패턴은 파이썬의 컨텍스트 매니저와 유사한 방식으로 자원의 정리를 보장한다. 

### `@Query`, `@Path`, `@Header`, `@Cookie`, `@Body`, `@Form`

엔드포인트의 각 파라미터를 특정 데이터 위치(쿼리 파라미터, 경로 파라미터, 헤더, 쿠키, 요청 본문, 폼 데이터)에 연결한다. 

### `@Response`, `@JSONResponse`, `@HTMLResponse`, `@FileResponse`

특정 응답 클래스를 사용하여 응답을 반환한다. 예를 들어 `@JSONResponse`는 JSON 형식의 응답을 반환할 때 사용된다. 



