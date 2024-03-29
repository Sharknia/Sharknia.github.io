---
IDX: "NUM-78"
tags:
  - Python
  - FastAPI
update: "2024-02-02T16:32:00.000Z"
date: "2023-11-07"
상태: "Ready"
title: "FastAPI와 asyncio"
---
## asyncio란?

### 소개

파이썬에서 비동기 프로그래밍을 위한 표준 라이브러리이다. 이 라이브러리는 `coroutine` 을 사용하여 동시성 코드를 작성하는 데 필요한 구조를 제공한다. 단일 스레드 내에서도 여러 I/O 바운드 작업과 고수준의 구조화된 네트워크 코드를 동시에 실행할 수 있으며, 이는 효율성과 속도에서 큰 이점을 제공한다. 

### 주요 컴포넌트

#### Event Loop

프로그램의 진입점으로써, 비동기적으로 실행될 다양한 작업들을 관리한다. 작업이 실행 준비가 되면 이벤트 루프는 해당 작업을 실행하고 완료되기를 기다리는 다른 작업으로 제어를 전환한다. 

#### Coroutine

코루틴은 `async def` 로 정의되는 비동기 함수이다. 코루틴은 `await`  키워드를 사용하여 실행을 일시 중지하고 이벤트 루프에 제어를 반환할 수 있어 다른 코루틴이 실행될 수 있게 한다. 

#### Future

아직 완료되지 않은 작업을 의미한다. 코루틴이 완료될 때 결과를 저장하거나 예외를 전달할 수 있는데 이를 통해 비동기 작업의 최종 상태를 알 수 있다. 

#### Task

`Future` 를 상속한다. 이는 이벤트 루프에서 코루틴의 실행을 가능하게 한다. Task는 코루틴을 스케줄링하고 실행 결과를 추적한다. 

### 사용 예시

```python
import asyncio

async def main():
    await asyncio.sleep(1)
    print('Hello, World!')

# 이벤트 루프 실행
asyncio.run(main())
```

이 예제에서 `asyncio.run()` 함수는 `main()` 코루틴을 이벤트 루프에서 실행한다. `main()` 내부에서 `asyncio.sleep(1)` 은 비동기적으로 1초간 대기하도록 한다. 이 대기 시간동안 이벤트 루프는 다른 코루틴을 실행할 수 있다. 

### 동기화 기능

asyncio는 비동기 프로그래밍 환경에서 동기화를 위한 여러 도구를 제공한다. asyncio.Lock은 공유 리소스에 대한 동시 액세스를 방지하고 asyncio.Event, asyncio.Condition, asyncio.Semaphore와 같은 다른 동기화 기본 요소도 사용할 수 있다. 

### 네트워킹 지원

TCP, UDP, SSL, TLS 등을 비롯한 다양한 네트워크 프로토콜에 대한 지원을 제공한다. asyncio 스트림을 사용하여 비동기적으로 네트워크 I/O 작업을 할 수 있으며 이를 통해 서버와 클라이언트 양쪽 모두에서 비동기 네트워크 어플리케이션을 만들 수 있다. 

### 성능과 제한 사항

입출력 바운드 작업과 고수준 구조화된 네트워크 코드를 실행하는데 효율적이다. 입출력 작업이 블로킹 되는 것을 피하면서 동시에 여러 입출력 작업을 관리할 수 있기 때문이다. 

예를 들어 웹서버는 많은 수의 동시 http 요청을 처리해야 하며 각 요청은 네트워크 입출력에 의존적이다. asyncio는 각 네트워크 연산이 완료되기를 기다리는 동안 다른 연산으로 전환하여 리소스를 효율적으로 사용할 수 있다. 

그러나 계산을 많이 요구 하는 작업(CPU 바운드 작업)에는 적압하지 않다. 이 작업은 병렬 처리가 필요한데, 이는 프로세스나 스레드를 여러개 사용해야 함을 의미한다. asyncio는 싱글 스레드, 싱글 프로세스 디자인이므로 병렬 CPU 작업을 위해서는 별도의 모듈이나 멀티 스레딩을 사용해야 한다. 

### 생명 주기

#### 이벤트 루프 생성

asyncio 프로그램은 이벤트 루프라는 중앙 실행기를 사용하여 실행된다. 이벤트 루프는 프로그램의 진입점에서 생성된다. 특히 아래의 함수를 사용하면 적절한 이벤트 루프를 설정하고 프로그램을 실행할 수 있다. 

```python
asyncio.run(main())
```

#### 코루틴 실행

`async def` 를 통해 코루틴을 생성하고 `await` 를 사용하여 실행을 스케줄한다. 코루틴은 이벤트 루프에 의해 실행될 태스크로 변환된다. 

```python
async def main():
    # 코루틴 실행
    result = await some_async_function()
```

#### 태스크 스케쥴링

이벤트 루프는 `await` 표현식을 만날 때마다 실행을 일시 중지하고 다른 태스크로 제어를 전환한다. 이를 통해 CPU가 I/O 작업으로 인해 차단되는 것을 방지하고 다른 태스크의 실행을 계속할 수 있다. 

#### 비동기 I/O 처리

`asyncio`는 네트워크 I/O, 파일 I/O와 같은 비동기 작업을 효율적으로 관리한다. 이벤트 루프는 모든 I/O 이벤트에 대해 비블로킹 소켓과 파일 디스크립터를 사용한다. 

#### 이벤트와 콜백 처리

이벤트 루프는 등록된 이벤트가 발생했을 때 적절한 콜백 함수를 실행한다. 콜백은 완료된 I/O 작업의 결과를 다루거나 타이머 이벤트 같은 다른 유형의 이벤트를 처리하기 위해 사용된다. 

#### 종료 처리

모든 태스크가 완료되거나 특정 조건을 만족하면 이벤트 루프는 종료될 수 있다. 이벤트 루프를 종료하기 전에 `close()` 함수를 호출하여 자원을 정리할 수 있다. 

```python
loop = asyncio.get_event_loop()
try:
    loop.run_until_complete(main())
finally:
    loop.close()
```

`asyncio.run()` 을 사용하는 경우에는 이벤트 루프의 생성과 종료가 자동으로 관리된다. 

#### 예외 처리

`asyncio` 는 예외가 발생할 경우 적절한 예외 처리를 위한 매커니즘을 제공한다. 코루틴 내부에서 발생한 예외는 해당 코루틴을 호출한 곳으로 전파되고 이벤트 루프는 처리되지 않은 예외를 포착하여 프로그래머에게 알린다. 

## FastAPI와 Async 함수

FastAP의 가장 큰 특징 중 하나는 asyncio 라이브러리를 기반으로 한 비동기 프로그래밍 지원이다. 이를 통해 개발자는 비동기 파이썬 코드를 작성할 수 있으며 이는 특히 I/O 바운드 작업(네트워크 요청 처리, 디스크에서의 데이터 읽기 /쓰기 등)을 비동기적으로 처리하는데 유리하다. 

### 비동기 함수의 장점

#### 성능 향상

I/O 작업이 완료되기를 기다리는 동안 다른 코드를 실행할 수 있으므로 I/O 바운드 시스템에서 동시성을 크게 향상시킬 수 있다. 

#### 스케일성

적은 수의 워커로 많은 수의 요청을 처리할 수 있으므로 더 효율적으로 리소스를 사용하고 스케일링 할 수 있다. 

#### 비용 효율성

리소스 사용의 효율성은 특히 클라우드 환경에서 환경 절감으로 이어질 수 있다. 

#### 응답성 향상

서버가 요청에 더 빨리 응답할 수 있기 때문에 사용자 경험이 향상된다. 

### 언제 비동기 함수를 쓰는 것이 더 유리할까? 

#### 네트워크 요청 처리

웹 API 호출, 원격 데이터베이스 쿼리 등 네트워크 I/O 관련 작업을 처리할 때

#### 고비용 I/O 작업

디스크 작업이나 네트워크를 통한 파일 전송과 같은 I/O 집중적인 작업을 비동기적으로 실행할 때 

#### 동시성이 필요한 작업

다수의 클라이언트나 서비스로부터 오는 동시 요청을 효과적으로 처리해야 할 때

#### WebSocket을 사용할 때 

실시간 통신을 위해 WebSocket 연결을 관리하는 경우 

### 비동기 함수 사용 시 주의할 점

#### 데드락

비동기 프로그래밍은 데드락에 빠질 위험이 있으므로 올바른 `await` 사용과 태스크 관리가 필요하다. 


        데드락이란?
두 개 이상의 프로세스나 스레드가 서로의 작업 완료를 무한히 기다리게 되는 상태

#### 에러 핸들링

비동기 코드에서는 예외 처리가 더 복잡해질 수 있으므로 주의가 필요하다. 

#### 디버깅의 복잡성

비동기 코드는 디버깅이 더 어려울 수 있다. 

#### 블로킹 코드와의 혼용

비동기 코드 내에서 블로킹 동기 코드를 호출하면 성능이 저하될 수 있으므로 주의해야 한다. 

### `async`

이 키워드는 두 가지 주요 상황에서 사용된다. 

#### 코루틴 함수를 정의할 때

```python
async def fetch_data():
    # 비동기 작업을 수행하는 코드
```

`async def` 는 이 함수가 코루틴 함수임을 나타낸다. 코루틴 함수는 호출될 때 즉시 실행되지 않고 대신에 코루틴 객체를 반환한다. 이 객체는 나중에 이벤트 루프에 의해 실행된다. 

#### async with와 async for를 사용할 때

비동기 컨텍스트 매니저와 비동기 이터레이터를 사용할 때 필요하다. 

### `await`

이 키워드는 코루틴의 실행을 일시 중지하고 코루틴의 결과가 준비될 때까지 기다린다. 코루틴, 태스크, 미래 객체 등이 뒤에 올 수 있다. 

```python
async def fetch_data():
    data = await some_async_function()
    return data
```

여기서 await는 some_async_function이 완료될 때까지 fetch_data 코루틴의 실행을 중지시킨다. 완료되면 그 결과값을 data에 할당하고 실행을 계속한다. 

이러한 비동기 프로그래밍 방식의 장점은 I/O 작업이 완료되기를 기다리는 동안 프로그램이 다른 작업을 계속할 수 있어 프로그램의 전반적인 실행 효율을 개선한다는 것이다.

