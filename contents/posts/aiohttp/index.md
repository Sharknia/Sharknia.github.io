---
tags:
  - Python
update: "2024-01-29"
date: "2023-11-08"
상태: "POST"
title: "aiohttp"
---
aiohttp는 파이썬의 비동기 HTTP 네트워킹 라이브러리이다. 이 라이브러리는 asyncio를 사용하여 비동기 I/O를 수행하고 클라이언트와 서버 양쪽 모두에 대한 HTTP 지원을 제공한다. 

즉 aiohttp를 사용하면 비동기적으로 HTTP 요청을 보내고 응답을 받을 수 있다. 

## 주요 특징

#### 비동기/동시성 지원

async, await를 사용하여 동시에 여러 HTTP 요청을 비동기적으로 처리할 수 있다. 

#### 서버와 클라이언트 사이드 지원

HTTP 클라이언트 기능과 HTTP 서버 기능을 모두 제공한다. 

#### 웹소켓 지원

웹소켓 연결 및 통신을 지원한다. 

#### 신호 및 슬롯 매커니즘

요청 처리 과정에서 다양한 이벤트에 대응할 수 있게 해준다. 

## 설치

파이썬 설치에 기본적으로 포함되지 않은 외부 라이브러리이다. 

```bash
pip install aiohttp
```

pip를 이용해 간단하게 설치할 수 있다. 

## 예제

```python
import aiohttp
import asyncio

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    async with aiohttp.ClientSession() as session:
        html = await fetch(session, 'http://python.org')
        print(html)

loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```

python 공식 홈페이지에서 HTML을 비동기적으로 가져오고 출력하는 간단한 예제이다. 

## 세션 관리

**ClientSession 객체를 사용하여 HTTP 클라이언트 세션을 관리할 수 있다.** 

#### 연결 재사용

여러 요청을 수행할 때, 세션은 자동으로 재연결을 사용한다.

```python
import aiohttp
import asyncio

async def main():
    async with aiohttp.ClientSession() as session:
        async with session.get('http://httpbin.org/get') as resp:
            print(await resp.text())
        async with session.get('http://httpbin.org/get') as resp:
            print(await resp.text())

asyncio.run(main())
```

#### 쿠키 관리

세션은 자동으로 쿠키를 관리한다. 첫 번째 요청에서 서버로부터 받은 쿠키는 자동으로 저장되고 다음 요청에 사용된다. 

```python
import aiohttp
import asyncio

async def main():
    async with aiohttp.ClientSession() as session:
        # 첫 번째 요청
        async with session.get('http://httpbin.org/cookies/set?cookie_name=cookie_value') as resp:
            print(await resp.text())
        # 두 번째 요청 - 첫 번째 요청에서 설정된 쿠키가 포함됩니다.
        async with session.get('http://httpbin.org/cookies') as resp:
            print(await resp.text())

asyncio.run(main())
```

#### 헤더의 기본값 설정

세션은 생성할 때 기본 헤더를 설정할 수 있으며 이후의 모든 요청에 이 헤더가 포함된다. 

```python
import aiohttp
import asyncio

async def main():
    headers = {'Authorization': 'Bearer your_token'}
    async with aiohttp.ClientSession(headers=headers) as session:
        async with session.get('http://httpbin.org/headers') as resp:
            print(await resp.text())

asyncio.run(main())
```

#### 타임아웃 관리

세션 또는 개별 요청에 대한 타임아웃을 설정할 수 있다. 

```python
import aiohttp
import asyncio
from aiohttp import ClientTimeout

async def main():
    timeout = ClientTimeout(total=5)  # 전체 요청에 대한 타임아웃을 5초로 설정
    async with aiohttp.ClientSession(timeout=timeout) as session:
        async with session.get('http://httpbin.org/delay/10') as resp:
            print(await resp.text())

asyncio.run(main())
```

#### 커스텀 설정

SSL 검증 비활성화, 프록시 설정 등의 커스텀 설정을 할 수 있다. 

```python
import aiohttp
import asyncio

async def main():
    async with aiohttp.ClientSession() as session:
        async with session.get('https://self-signed.badssl.com/', ssl=False) as resp:  # SSL 검증 비활성화
            print(await resp.text())

asyncio.run(main())
```

#### 자원 해제

`async with` 문을 사용하면 세션 사용이 끝날 때 자동으로 자원을 해제한다. 

```python
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:  # 세션 생성 및 자동 자원 해제
        async with session.get(url) as response:
            return await response.text()

async def main():
    html = await fetch('http://python.org')
    print(html)

asyncio.run(main())
```

## 에러 핸들링

#### HTTP 응답 에러

서버가 4XX 클라이언트 에러 또는 5XX 서버 에러를 반환하는 경우, `ClientResponseError` 가 발생할 수 있다. 이는 `raise_for_status()` 메서드를 사용하여 처리할 수 있다. 

```python
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get(url) as response:
                response.raise_for_status()
                return await response.text()
        except aiohttp.ClientResponseError as e:
            print(f"HTTP Response Error: {e.status} {e.message}")
        except aiohttp.ClientError as e:
            print(f"HTTP Client Error: {str(e)}")
        except Exception as e:
            print(f"Unexpected Error: {str(e)}")

async def main():
    await fetch('http://httpbin.org/status/400')  # 이 URL은 400 Bad Request를 반환합니다.

asyncio.run(main())
```

#### 연결 에러

네트워크 문제 또는 DNS 문제로 인해 연결을 설정할 수 없는 경우 `ClientConnectError` 예외가 발생할 수 있다. 

```python
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get(url) as response:
                return await response.text()
        except aiohttp.ClientConnectorError as e:
            print(f"Connection Error: {str(e)}")

async def main():
    await fetch('http://nonexistent.url')  # 존재하지 않는 URL

asyncio.run(main())
```

#### 타임아웃 에러

지정된 타임아웃 내에 서버로부터 응답을 받지 못하는 경우 `asyncio.TimeoutError` 예외가 발생한다.

```python
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get(url, timeout=1) as response:  # 1초 타임아웃 설정
                return await response.text()
        except asyncio.TimeoutError:
            print("Timeout Error: The request timed out")

async def main():
    await fetch('http://httpbin.org/delay/10')  # 응답 지연 URL

asyncio.run(main())
```

#### 일반 HTTP 클라이언트 에러

aiohttp가 발생시킬 수 있는 다른 클라이언트 에러를 처리한다. 

```python
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get(url) as response:
                return await response.text()
        except aiohttp.ClientError as e:
            print(f"Client Error: {str(e)}")

async def main():
    await fetch('http://httpbin.org/status/500')  # 서버 에러 URL

asyncio.run(main())
```

## 스트리밍 응답

스트리밍 응답을 처리하는 것은 큰 데이터를 다룰 때 특히 유용하다. 스트리밍을 사용하여 응답의 전체 내용을 메모리에 한 번에 로드하지 않고 데이터를 조각으로 나누어 처리할 수 있다. 이 방법은 메모리 사용량을 최적화하고 대용량 응답을 효율적으로 처리할 수 있도록 한다. 

```python
import aiohttp
import asyncio

async def stream_response(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            # 스트리밍 응답 처리
            async for data in response.content.iter_any():
                # 데이터 조각을 처리 (예: 파일에 쓰기, 출력 등)
                print(data)

async def main():
    await stream_response('http://httpbin.org/stream/20')

asyncio.run(main())
```

이 예제에서는 `response.content.iter_any()` 를 사용하여 응답 스트림에서 데이터를 순차적으로 읽는다. 

스트리밍 응답을 사용할 때는 데이터의 양이 많거나 응답을 받는 동안 다른 처리를 동시에 해야 하는 경우에 매우 효과적이다. 예를 들어 다운로드한 데이터를 파일에 쓰면서 동시에 다음 데이터 청크를 받거나 데이터를 받아서 실시간으로 사용자에게 전송하는 경우 등에 유용하다. 

스트리밍을 사용할 때에는 네트워크 상황이나 서버의 응답 특성에 따라 데이터를 받는 속도가 일정하지 않을 수 있음을 인지하고 있어야 한다. 데이터를 처리하는 로직이 블로킹되지 않도록 주의해야 하며, 가능하면 각 청크를 처리하는데 시간이 너무 오래 걸리지 않도록 설계해야 한다.

## 테스트

aiohttp의 비동기적인 특성 때문에 전통적인 동기 테스트 방식을 그대로 사용할 수 없다. 대신 aiohttp는 비동기 테스트를 지원하기 위해 pytest와 함께 사용할 수 있는 `aitohttp.test_utils` 모듈을 제공한다. 이를 통해 웹서버와 클라이언트 코드를 테스트 할 수 있다. 

#### 클라이언트 코드 테스트

클라이언트 코드를 테스트 하기 위해서는 일반적으로 pytest와 pytest-aiohttp 플러그인을 사용한다. 이를 통해 aiohttp 클라이언트 세션을 비동기적으로 테스트 할 수 있다. 

```python
import aiohttp
import pytest
from aiohttp import web

async def test_example_client(aiohttp_client):
    async def hello(request):
        return web.Response(text='Hello, world')
    
    app = web.Application()
    app.router.add_get('/', hello)
    
    client = await aiohttp_client(app)
    resp = await client.get('/')
    assert resp.status == 200
    text = await resp.text()
    assert text == "Hello, world"
```

aiohttp_client는 pytest-aiohttp 플러그인에서 제공하는 fixture로, 테스트용 애플리케이션 서버와 상호작용하는 데 사용할 수 있는 aiohttp.ClientSession 객체를 생성한다. 


        <div class="callout gray_background">
            💡 <span>fixture란?
테스트를 실행하기 전에 필요한 준비 작업과 설정을 의미한다. 일반적으로 fixture는 테스트 환경을 설정하고 테스트가 실행되는 동안 필요한 리소스나 상태를 생성하며, 테스트가 완료된 후에 정리 작업을 수행한다. </span>
        </div>
#### 서버 코드 테스트

aiohttp 웹서버를 테스트 할 때에는 aiohttp.test_utils.TestClient를 사용하여 요청을 보내고 응답을 검사한다. 

```python
from aiohttp import web
from aiohttp.test_utils import AioHTTPTestCase, unittest_run_loop
from myapp import create_app  # 가정: myapp 모듈에서 앱을 생성하는 함수

class MyAppTestCase(AioHTTPTestCase):

    async def get_application(self):
        """
        AioHTTPTestCase에서 애플리케이션 인스턴스를 생성하기 위해 오버라이드
        """
        return create_app()

    @unittest_run_loop
    async def test_example(self):
        resp = await self.client.request("GET", "/")
        assert resp.status == 200
        text = await resp.text()
        assert 'Hello, world' in text
```

AioHTTPTestCase는 aiohttp.test_utils에서 제공하는 기본 테스트 케이스 클래스이며, unittest_run_loop 데코레이터는 테스트 코루틴을 이벤트 루프에서 실행할 수 있게 해준다.

aiohttp를 테스트할 때 주의해야 할 점은 테스트 환경에서도 비동기 코드를 올바르게 실행하기 위해 적절한 테스트 실행기를 설정해야 한다는 것이다. pytest는 비동기 테스트를 위한 좋은 선택이며, pytest-asyncio 플러그인을 사용하면 pytest에서 async def 테스트 함수를 직접 사용할 수 있다.

또한, 실제 HTTP 호출을 모킹하기 위해 aiohttp의 pytest_plugin이 제공하는 aioresponses와 같은 도구를 사용할 수도 있다. 이를 통해 실제 외부 서비스와의 통신 없이 HTTP 요청과 응답을 시뮬레이션할 수 있어, 테스트의 견고성을 높이고 실행 시간을 단축시킬 수 있다.



