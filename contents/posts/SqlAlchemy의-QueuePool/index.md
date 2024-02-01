---
tags:
  - SqlAlchemy
  - DataBase
  - Python
  - Work
update: "2024-02-01"
date: "2024-01-18"
상태: "Ready"
title: "SqlAlchemy의 QueuePool"
---
개발중인 서비스에서 Postgresql을 적용한지 이제 한달 조금 더 지났다. 

아직 DB 마이그레이션 작업은 거의 되지 않았으므로, 실제로 Postgresql DB를 이용하는 서비스는 그다지 많지 않았다. 

새로 업데이트 하는 기능들에 대해서는 적극적으로 Postgresql을 이용하기로 했고, 이번에 새로 퀴즈 기능을 개발하면서 이 기능은 전부 RDB 기반으로 만들어졌다. 

퀴즈 기능의 개발은 유저의 접속률을 늘리기 위한 것으로, 모든 테스트를 마친 후 실서버에 배포가 되고 클라이언트까지 패치가 된 후 당당하게 퀴즈 풀러 오시라고 사용자들에게 푸시를 날렸다. 

그리고 펑! 서버가 터졌다. 

## 현상

실제 서비스가 엄청나게 느리거나 거의 작동하지 않았다. 

Postgresql DB와 연결된 API들이 작동하지 않았다. 

서버에서는 `QueuePool limit of size 5 overflow 10 reached, connection timed out, timeout 30.00` 오류가 대량으로 발생했다. 

서버 모니터링 결과 푸쉬 직후 대량의 트래픽이 집중적으로 발생했다. 

Flarelane을 통해 확인한 결과, 2000명 정도가 동시 접속을 시도했다. 

Supabase의 DB 커넥션이 순간적으로 크게 늘어났다. (풀러를 사용중인데도 불구하고)



시간이 지나면서 (약 10분이 안되는 짧은 시간) 트래픽이 줄어들고 자연스럽게 오류가 줄고, 서비스가 정상화되었다. 

## 원인 파악 및 해결

Sqlalchemy의 QueuePool을 현재 지금 pool size 5, max overflow 10으로 맞춰놓았었는데, 트래픽이 증가하면서 QueuePool의 한도가 다 차고, 대다수 요청이 최대 대기 시간 30초를 꽉 채워 대기하다가 오류가 발생했다. 

조사 결과, 대다수 많은 실제 서비스에서 해당 설정값은 대체적으로 너무 적어 값을 늘려서 사용한다고 한다. 정확한 값을 찾기 위해서는 여러번의 테스트가 필요하며, 점진적으로 늘려가면서 적당한 값을 찾아야 한다고 한다. 

supabase 기반의 postgresql을 사용하고 있는데, 일단 postgresql의 connection 여유는 충분한 상황이었으므로 일단 설정값을 pool size 10, max overflow 20으로 각각 두배로 늘려주었다. 

이벤트나, 푸쉬가 있을 경우 추가적인 모니터링이 필요할 것이다. 

## Sqlalchemy의 QueuePool

데이터베이스 Connection은 리소스가 많이 필요하다. QueuePool은 이런 Connection을 효율적으로 관리하여 성능을 향상시키기 위해 만들어졌다. Connection을 빠르게 재사용함으로써, 애플리케이션의 반응 시간을 단축 시키고 부하 상황에서도 안정적으로 동작하도록 한다. 

Quepool은 내부적으로 Connection 객체들을 큐로 관리한다. 동시성을 고려하여 설계되어 여러 스레드 또는 프로세스에서 안전하게 사용될 수 있다. 

### QueuePool의 생명주기 

#### 생성

QueuePool은 Sqlalchemy 엔진이 생성될 때 함께 생성된다. 이 때 데이터베이스와의 Connection 설정이 정의된다. 

#### Connection 관리 

애플리케이션에서 데이터베이스 Connection이 필요할 때, QueuePool이 Connection을 제공한다. 사용 가능한 Connection이 없으면 새 Connection을 생성하거나 대기열에서 Connection을 기다린다. 

#### Connection 반환

작업이 완료되면 Connection은 Pool에 반환되어 재사용된다. 

#### 종료

애플리케이션 종료와 함께 Pool에 있는 모든 Connection이 안전하게 종료된다. 

### 작동

QueuePool은 기본적으로 미리 설정된 `pool_size` 내에서 Connection을 관리한다. 사용 가능한 Connection이 있다면 그 연결을 제공하고, 없으면 새로은 Connection을 생성한다. 

`pool_size`는 동시에 활성화 할 수 있는 최대 연결수이다. 만약 이 한계를 초과한다면, `max_overflow` 에 설정된 값에 따라 추가 Connection을 생성한다. `max_overflow` 는 `pool_size` 를 초과하여 생성할 수 있는 추가 연결의 최대 수이다. 

`max_overflow` 까지 초과한다면 추가 Connection 연결 요청은 대기열에 들어가며 사용 가능한 Connection 이 생길때까지 대기한다. 이 때, 연결 대기 시간이 `timeout` 설정을 초과하면 연결 요청은 실패한다. 

#### pool_size

이 설정은 가능한 최대 연결의 최대 수를 정의 하는 것이므로, 5로 설정했다고 해서 실제 연결이 5개가 유지되는 것은 아니다. 사용되지 않는 Connection은 pool에 반환되며, 유휴 상태로 남아있다가 `recycle` 설정 시간을 넘어서면 끊어진다. 

#### max_overflow

pool_size를 초과하는 최대 개수를 정의한다. pool_size 가 5이고, max_overflowrk 10이라면 동시에 최대 5+10 = 15개의 연결을 허용하는 것이다. max_overflow에 해당되어 생성된 Connection이라 할지라도 사용 후에 풀에 반환된다(바로 끊어지는 것이 아니다). 다만, 해당 Connection은 pool_size의 기본 한계를 넘어선 것이므로 풀에서 더 높은 우선 순위로 종료될 수 있다. 

### Sqlalchemy의 풀링 전략

Sqlalchemy의 기본 풀링 전략은 Queuepool이지만, 이외에도 다른 전략들이 있다. 

### StaticPool

연결의 고정된 집합을 유지한다. 모든 Connection 요청은 이 고정된 집합에서 제공된다. 단일 사용자 또는 단일 프로세스 어플리케이션에 적합하다. 

### NullPool

Connection Pooling을 사용하지 않는다. 매 요청마다 새로운 DB Connection이 열리고 작업 종료 후 Connection이 닫힌다. 

매우 드문 요청이 있는 어플리케이션에 적합하다. 

### Singleton ThreadPool

각 스레드에 대해 하나의 Connection을 유지한다. 스레드마다 고유한 Connection을 사용한다. 

멀티 스레드 환경에서 각 스레드가 자체 Connection을 가질 필요가 있을 때 유용하다. 



