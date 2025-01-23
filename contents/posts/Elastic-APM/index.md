---
IDX: "NUM-237"
tags:
  - 
description: "Elastic APM"
update: "2025-01-23T09:17:00.000Z"
date: "2025-01-23"
상태: "Ready"
title: "Elastic APM"
---
## Elastic APM

Elastic APM (Application Performance Monitoring)은 Elastic Stack(ELK 스택)의 일부로, 애플리케이션의 성능을 모니터링하고 성능 병목 현상과 오류를 추적하는 도구입니다. 주로 백엔드 서비스, 데이터베이스 쿼리, HTTP 요청, 비동기 작업 등의 성능을 분석하는 데 사용됩니다.

### ELK 스택이란? 

ELK 스택은 Elasticsearch, Logstash, Kibana의 약자로, 로그 및 데이터 분석을 위한 오픈소스 도구 모음입니다.

- Elasticsearch: 검색 및 분석을 위한 강력한 검색 엔진

- Logstash: 로그 및 데이터 수집·가공 도구

- Kibana: 데이터를 시각화하는 대시보드 도구

주로 서버 로그, 애플리케이션 로그를 수집하고 검색·분석하는 데 사용됩니다. 최근에는 Beats(경량 데이터 수집기)를 포함하여 ELK → Elastic Stack으로 확장되었습니다.

### 주요 기능

1. 트랜잭션 추적

    - 애플리케이션의 특정 요청(예: API 호출)이 시작부터 끝까지 어떻게 실행되는지 추적

    - 각 요청이 얼마나 걸리는지, 어느 부분에서 병목이 발생하는지 확인 가능

1. 분산 추적 (Distributed Tracing)

    - 여러 서비스 간의 호출을 추적하여 마이크로서비스 아키텍처에서 요청이 어떻게 전달되는지 분석 가능

    - OpenTelemetry 및 Jaeger와 유사한 기능 제공

1. 에러 및 예외 감지

    - 애플리케이션에서 발생하는 오류(예: 500 에러, 데이터베이스 연결 오류)를 자동으로 수집 및 분석

1. 메트릭 수집

    - CPU, 메모리 사용량, 요청 수, 응답 시간 등의 메트릭을 자동으로 수집하여 시각화

1. 로그 통합

    - Elastic Stack의 Elasticsearch 및 Kibana와 연동하여 로그를 함께 분석 가능

### 구조

Elastic APM은 보통 아래와 같은 구성으로 동작합니다.

1. APM Agent (애플리케이션 내부에서 실행)

    - 애플리케이션 코드에 삽입되어 요청, 오류, 메트릭을 수집하는 역할

    - 언어별 지원: Python, Java, Node.js, Go, Ruby, .NET 등

1. APM Server (수집된 데이터를 Elasticsearch로 전달)

    - APM Agent가 보낸 데이터를 받아서 Elasticsearch에 저장

1. Elasticsearch (데이터 저장소)

    - 수집된 성능 데이터를 저장하고 검색하는 역할

1. Kibana (시각화 도구)

    - APM 데이터를 대시보드 형태로 분석하고 시각화하는 역할

### 장점

1. Elasticsearch 기반 → 빠르고 강력한 검색 및 분석 기능 제공

1. 마이크로서비스 친화적 → 분산 트랜잭션 분석 지원

1. 자동 인스트루먼테이션 → 코드 수정 없이도 기본적인 성능 데이터 수집 가능

1. 로그 + 메트릭 + APM 통합 → ELK Stack과 결합하여 강력한 모니터링 시스템 구축 가능

1. 오픈소스 → 기본적인 기능을 무료로 사용할 수 있음

### Elastic APM을 도입해야 하는 경우

1. 마이크로서비스 구조에서 서비스 간의 요청을 추적하고 싶은 경우

1. 성능 저하 원인을 빠르게 찾고 싶은 경우

1. 애플리케이션의 에러와 예외를 자동으로 모니터링하고 싶은 경우

1. ELK Stack을 이미 사용하고 있는 경우

1. Elastic APM은 New Relic, Datadog, AWS X-Ray 같은 APM 도구와 비교할 수 있으며, ELK 스택과의 강력한 연동이 가장 큰 장점입니다.

## Elastic APM을 설치만 하면 request/response 분석이 자동?!

기본적인 요청(Request), 응답(Response), 오류(Error), 메트릭(CPU, 메모리 등) 은 별도 설정 없이 자동으로 수집됩니다. Elastic APM 에이전트를 애플리케이션에 설치하고 실행하면 다음과 같은 데이터를 자동으로 캡처합니다.

트랜잭션(Transaction)

- HTTP 요청 (URL, 응답 시간, 상태 코드)

- DB 쿼리 실행 시간

- 외부 API 호출 시간

스팬(Span)

- 트랜잭션 내부의 세부 작업 (예: SQL 쿼리, HTTP 요청 등)

에러(Error)

- Unhandled Exception, HTTP 500 오류 등

메트릭(Metric)

- CPU, 메모리 사용량

성능 오버헤드를 최소화하기 위해 전체 Body를 저장하지 않으며, URL, 상태 코드, 응답 시간, 헤더 등의 정보만 캡처합니다. 다만 저장하려면 추가 설정을 통해 저장할 수 있습니다. 

## 분산 트레이싱(Distributed Tracing)

Elastic APM에서는 "분산 트레이싱(Distributed Tracing)" 이라는 기능을 사용하여 각 서비스가 독립적이더라도 같은 요청을 추적할 수 있습니다. 이 과정에서 Trace ID와 Transaction ID라는 개념이 사용됩니다.

### 요청 흐름 예시 (MSA 환경)

1. 클라이언트 → `API Gateway`로 요청 보냄

1. API Gateway → `User Service` 호출 (Trace ID 생성)

1. User Service → `Order Service` 호출 (같은 Trace ID 사용)

1. Order Service → `Payment Service` 호출 (같은 Trace ID 사용)

Trace ID가 동일한 요청이라면 여러 서비스에서 발생한 트랜잭션이 하나의 흐름으로 연결됩니다.

각 마이크로서비스는 독립적으로 실행되지만, 각 요청이 "같은 요청"임을 Trace ID를 통해 추적하는 방식입니다.

### 분산 트레이싱이 동작하는 방식

- 최초 요청이 들어오면 APM 에이전트가 `Trace ID`를 생성

- 서비스 간의 HTTP 요청 헤더에 `Trace ID`를 자동으로 추가

- 다른 서비스에서도 이 `Trace ID`를 읽고 이어서 로깅

### 헤더 예시 (Trace Context)

```makefile
elastic-apm-traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

각 MSA가 이 헤더를 참조하여 "같은 요청"임을 식별합니다. OpenTelemetry 및 W3C Trace Context 표준을 지원하기 때문에, 다른 APM 툴과도 연동할 수 있습니다. Elastic APM을 사용하면, 마이크로서비스 간 각 요청이 어떻게 전달되었는지를 Kibana에서 시각적으로 분석할 수 있습니다.

### Trace ID 발행 방식

APM 에이전트가 요청을 처음 만나면, Trace ID를 생성하여 HTTP 헤더에 포함합니다.

1. 요청이 처음 들어오면 (Trace ID가 없음)

    → APM 에이전트가 새로운 Trace ID를 생성

    → HTTP 헤더(`elastic-apm-traceparent`)에 추가

    → APM 로그를 저장할 때 Trace ID를 키값으로 저장

1. 이미 Trace ID가 존재하는 경우 (다른 서비스에서 발행됨)

    → 기존 요청 헤더에서 `Trace ID`를 가져와 그대로 사용

    → 같은 Trace ID를 기반으로 새로운 트랜잭션(Transaction)을 생성

    → APM 로그 저장 시 기존 Trace ID를 유지

APM이 적용된 서비스는 Trace ID가 없는 요청을 만나면 자동으로 생성하고, 이미 있으면 그대로 유지하면서 로깅합니다.

### 로그 저장 및 추적

Elastic APM은 Trace ID를 기반으로 모든 서비스의 로그를 그룹화하여 저장합니다.

- Trace ID를 기준으로 각 요청의 흐름을 추적

- 개별 서비스의 실행 시간을 트랜잭션(Transaction) 단위로 기록

- 요청 내에서 실행된 특정 동작(DB 쿼리, HTTP 요청 등)은 스팬(Span)으로 기록

- 모든 기록은 Elasticsearch에 저장되어 Kibana에서 Trace ID별로 시각화 가능

📌 예제

| Trace ID | 서비스 | 요청 URL | 응답 시간 | 상태 코드 |
| --- | --- | --- | --- | --- |
| `abc123` | API Gateway | `/user/profile` | 120ms | 200 |
| `abc123` | User Service | `/get-user` | 80ms | 200 |
| `abc123` | Order Service | `/get-orders` | 150ms | 200 |
| `abc123` | Payment Service | `/payment-status` | 250ms | 500 |

이런 식으로 하나의 요청이 어떤 서비스들을 거쳤고, 어디에서 병목이 발생했는지 확인 가능합니다.

따라서 Trace ID를 기반으로 로그를 그룹화하려면 모든 요청이 지나는 마이크로서비스(MSA)에 APM 에이전트를 설치하는 것이 중요합니다. 만약 특정 서비스에 APM이 설치되지 않았다면, Kibana에서는 특정 서비스를 거친 요청이 누락되거나 정확한 실행 시간을 알 수 없습니다.

## Kibana

Elastic APM + Kibana를 활용하면 Trace ID별 요청 흐름을 시각적으로 분석할 수 있습니다.

- 트랜잭션 별 요청 흐름을 한눈에 확인

- 서비스 간 실행 시간 비교 (병목 지점 분석)

- 에러가 발생한 서비스, 실행 시간 초과 등을 쉽게 탐색

Kibana의 APM UI에서는 각 서비스에서 실행된 트랜잭션과 관련된 스팬을 트리 형태로 볼 수 있어 트랜잭션이 어느 서비스에서 가장 오래 걸리는지 확인 가능합니다. 

```plain text
Trace ID: abc123
├── API Gateway (120ms)
│   ├── User Service (80ms)
│   ├── Order Service (150ms)
│   ├── Payment Service (250ms) → ⛔ 500 에러 발생
```



