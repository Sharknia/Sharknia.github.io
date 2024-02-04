---
IDX: "NUM-123"
tags:
  - Github Actions
description: "needs를 사용한 Job들의 종속성 정의"
update: "2024-02-04T10:50:00.000Z"
date: "2024-02-04"
상태: "Ready"
title: "Github Actions Job - needs"
---
## 개요

기본적으로 아무 설정이 없다면 각 Job들은 병렬적으로 실행됩니다.  예를 들어 job1과 job2를 선언한 경우, 이 두 job은 워크플로우 실행 시 동시에 시작이 됩니다. 

이 때, `needs` 를 사용해 Job들 사이에 종속성을 주어 Job 하나가 끝나야 다음 Job이 실행되도록 설정할 수 있습니다.

## 예제

### 순차적으로 실행하기

다음과 같이 needs에 다른 job의 이름을 지정해주면, 해당 job이 끝나야 이 job이 실행되도록 할 수 있습니다. 

```yaml
name: job1-job2

jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
			# 첫번째 작업 실행...
  job2:
		needs: job1 # 'job1' 작업이 성공적으로 완료된 후에 실행
    runs-on: ubuntu-latest
    steps:
			# 두번째 작업 실행...
```

이 경우에는 job1이 실패할 경우, job2의 작업도 건너뛰게 됩니다. 

### 여러개의 needs 선언하기

```yaml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

위와 같이 선언되어있다고 할 때에, job1이 반드시 성공해야 job2가 실행이 되며, job1, job2가 모두 성공적으로 완료되어야 job3이 실행됩니다. 

### 성공-실패 여부 상관없이 순차적으로 실행하기

job의 실행에도 조건문을 걸 수 있습니다. 

```yaml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    if: ${{ always() }}
    needs: [job1, job2]
```

예를 들어 이 조건문은 job1, job2의 성공 여부와는 상관없이 단지 순차적으로 작업을 실행합니다. 즉, job2가 실패하더라도 job3은 job1, job2 완료 이후에 실행됩니다. 

## 참조

[https://docs.github.com/ko/actions/using-jobs/using-jobs-in-a-workflow](https://docs.github.com/ko/actions/using-jobs/using-jobs-in-a-workflow)



