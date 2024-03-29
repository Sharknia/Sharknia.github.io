---
IDX: "NUM-73"
tags:
  - AWS
  - DataBase
update: "2024-02-02T16:32:00.000Z"
date: "2023-11-01"
상태: "Ready"
title: "DynamoDB의 동시성 제어(Concurrency Control)"
---
## 데이터베이스의 동시성 제어란?

여러 트랜잭션 또는 연산이 동시에 수행될 때 데이터의 일관성을 유지하고 충돌을 방지하기 위한 매커니즘을 말한다. 잘못된 동시성 제어는 데이터의 무결성을 해치고 시스템의 성능을 저하시키며 예측할 수 없는 결과를 초래할 수 있다. 

### 동시성 제어의 주요 개념

#### 락 (Locking)

데이터베이스 항목에 대한 동시 액세스를 제어하기 위해 사용된다. 트랜잭션은 데이터에 액세스 하기 전에 락을 획득해야 하며, 작업이 완료되면 락을 해제해야 한다. 

일반적으로 비관적 잠금과 낙관적 잠금의 두 형태로 구분된다. 

#### 트랜잭션 (Transaction)

일련의 데이터베이스 작업을 하나의 논리적 단위로 묶는 것이다. 트랜잭션은 모두 성공하거나 모두 실패해야 하며, 중단 상태에서 중단되어서는 안된다. 

#### 일관성 (Consistency)

데이터베이스의 모든 트랜잭션이 데이터를 올바른 상태에서 유지하도록 보장하는 속성이다. 

- 강력한 일관성 (Strong Consistency)

    데이터베이스의 모든 노드가 항상 동일한 데이터를 보게 됨을 보장한다. 

    데이터베이스에 쓰기 작업이 수행된 후 모든 후속 읽기 작업이 그 쓰기 작업의 결과를 반영해야 한다. 

    실시간 시스템, 금융 시스템 또는 기타 높은 무결성이 필요한 시스템에서 중요하다. 

    일반적으로 높은 네트워크 지연과 함께 오며, 시스템의 전반적인 성능과 확장성에 영향을 미칠 수 있다. 

- 최종 일관성 (Eventual Consistency)

    시간이 지나면서 데이터베이스의 모든 노드가 동일한 데이터를 볼 수 있게 됨을 보장한다. 

    쓰기 작업이 수행된 후 일정 시간이 지나면 모든 읽기 작업이 쓰기 작업의 결과를 반영한다. 

    일반적으로 높은 확장성과 낮은 지연 시간을 제공하며 분산 시스템에서 높은 성능을 달성하는데 유리하다. 

    일시적인 데이터 불일치를 허용하므로 애플리케이션의 로직이 이러한 불일치를 처리할 수 있도록 설계되어야 한다. 

#### 분리성 (Isolation)

각 트랜잭션이 독립적으로 실행되도록 보장하는 속성이다. 트랜잭션들 사이에 어떤 중간 상태도 다른 트랜잭션에게 보여져셔는 안된다. 

#### 지속성 (Durability)

트랜잭션이 성공적으로 완료된 후에는 시스템의 어떤 실패에도 영향 받지 않고 데이터베이스 변경이 유지되도록 보장하는 속성이다. 

## DynamoDB의 동시성 제어 매커니즘

DynamoDB는 동시성 제어와 데이터 일관성을 관리하기 위한 여러 매커니즘을 제공한다. 

### 조건식 사용 (Conditional Writes)

항목을 작성하거나 업데이트 할 때 조건식을 사용하여 해당 항목이 변경되지 않았음을 확인할 수 있다. 이것은 항목이 기존에 있던 상태에 따라 연산을 수행하도록 할 때 유용하다. 예를 들어 항목이 특정값에 해당하지 않으면 업데이트 하지 않도록 요청할 수 있다. 

### 컨설턴트 (Consistent Reads)

기본적으로 최종적으로 일관성 있는 읽기를 제공한다. 이것은 약간의 지연을 허용하면서 빠른 응답 시간을 제공한다. 하지만 바로 이전의 쓰기 작업 이후에 항목의 최신 상태를 확인하려면 Strongly Consistent Read 옵션을 사용할 수 있다. 이 옵션은 데이터의 최신 상태를 보장한다.

<hr style="border: none; height: 1px; background-color: #e0e0e0; margin: 16px 0;" />
하지만 Dynamo DB에서 동시성은 주로 다음의 두 가지 방법을 사용하여 수행된다. 

## 낙관적 잠금 (Optimistic Locking)

낙관적 잠금은 클라이언트가 업데이트(또는 삭제) 하려는 항목이 DynamoDB의 항목과 동일한지 확인하는 전략이다. 이 전략을 사용하면 데이터베이스 쓰기는 다른 사용자의 쓰기에 의해 덮여지는 일을 방지할 수 있다. 

이 전략은 DynamoDB의 테이블에 `버전 번호` 라는 속성을 사용하여 구현된다. 클라이언트가 데이터 항목을 수정할 때 클라이언트 측의 버전 번호가 테이블 항목의 버전 번호와 동일해야 한다. 버전 번호가 동일하면 다른 사용자가 레코드를 변경하지 않았음을 의미하며 쓰기를 수행할 수 있다. 

그러나 버전 번호가 다르면 다른 사용자가 이미 레코드를 업데이트 했을 가능성이 있으며, 이 경우 DynamoDB는 `ConditionalCheckFailedException` 예외를 발생시켜 쓰기를 거부한다. 

낙관적 잠금을 사용하기 위해서는 코드에서 `ConditionalCheckFailedException` 을 포함하여 `put`, `delete`, `update` 작업에 대한 조건부 쓰기를 구현해야 한다. 

## 비관적 잠금 (Pessimistic Locking)

비관적 잠금은 동시 업데이트를 방지하기 위해 사용하는 또 다른 전략이다. 낙관적 잠금이 버전 번호를 클라이언트에 전송하는 반면, 비관적 잠금은 버전 번호를 유지하지 않고 동시 업데이트를 방지하려고 한다. 

대부분의 데이터베이스 서버는 트랜잭션 시작 시 잠금을 획득하고 트랜잭션이 완료된 후에만 잠금을 해제하지만, DynamoDB는 이를 약간 다르게 처리한다. DynamoDB는 데이터를 잠그지 않고, 대신 트랜잭션을 사용하여 검토 중인 데이터에 대한 다른 스레드의 변경을 식별하며, 변경이 감지되면 DynamoDB는 트랜잭션을 취소하고 오류를 발생시킨다. 

비관적 잠금을 구현하기 위해 `taransacWrite` 라는 메소드를 제공한다. 이를 사용하면 처리 중인 항목을 잠그지 않지만 항목을 모니터링하고, 다른 스레드가 데이터틀 수정하면 전체 트랜잭션이 데이터 변경으로 인해 실패하고 데이터를 롤백한다. 

## RDB 와의 차이점

### 데이터 모델

- RDB

    관계형 데이터 모델을 사용하여 테이블, 행, 열의 데이터로 형태를 데이터 구조하화며, 테이블 간의 관계를 정의하여 데이터의 무결성을 유지한다. 

- DynamoDB

    NoSQL 데이터베이스로 키-값과 문서 데이터 모델을 사용한다. 데이터 무결성 보장을 위해 관계형 데이터베이스와 다른 매커니즘을 사용한다. 

### 동시성 제어

- RDB

    트랜잭션을 사용하여 동시성을 제어한다. 트랜잭션을 시작할 때 데이터에 잠금을 걸고, 트랜잭션이 완료될 때까지 잠음을 유지하여 다른 트랜잭션에 의한 동시 수정을 방지한다. 

- DynamoDB

    낙관적 잠금과 조건부 업데이트를 사용하여 동시성을 제어하거나 `taransacWrite` API를 사용하여 트랜잭션을 지원한다. 

### 일관성

- RDB

    보다 강력한 일관성(Strong Consistency)을 지원한다. 트랜잭션의 모든 변경이 즉시 모든 다른 트랜잭션에 표시된다. 

- DynamoDB

    기본적으로 최종 일관성(Eventual Consistency)을 제공한다. 필요한 경우 강력한 일관성(Strong Consistency)을 요청할 수 있다. 

### 분리성

- RDB

    트랜잭션의 분리성을 보장하여 각 트랜잭션이 독립적으로 실행되도록 한다. 

- DynamoDB

    `TransactWrite` API를 통해 여러 항목에 대한 트랜잭션을 지원하여 분리성을 제공한다. 

### 지속성

두 DB 모두 트랜잭션이 성공적으로 완료된 후에는 시스템의 어떤 실패에도 영향을 받지 않고 데이터베이셔 변경이 유지되도록 지속성을 보장한다. 

## 참고 문헌

[https://dynobase.dev/dynamodb-locking/](https://dynobase.dev/dynamodb-locking/)

