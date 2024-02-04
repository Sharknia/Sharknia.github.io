---
IDX: "NUM-122"
tags:
  - Github Actions
description: "Github Actions 소개"
update: "2024-02-04T10:29:00.000Z"
date: "2024-02-04"
상태: "Ready"
title: "Github Actions"
---
## 소개

Github Actions는 깃허브에서 직접 소프트웨어 개발 워크플로우를 자동화할 수 있는 기능입니다. 개발자는 이를 사용해서 소프트웨어 빌드, 테스트, 배포와 같은 과정을 자동화하여 특정 트리거가 발생했을 경우 자동으로 실행되는 워크플로우를 작성할 수 있습니다. 

## 강점 및 특징

### 깃허브와의 통합

깃허브와 깊게 통합되어 있어 깃허브 리포지토리 내에서 직접 워크플로우를 관리하고 실행할 수 있습니다. 따라서 코드 변경, PR, 이슈 생성 등 깃허브 이벤트에 기반한 자동화를 간편하게 설정할 수 있습니다. 

### 언어와 프레임워크에 대한 광범위한 지원

다양한 프레임워크와 프로그래밍 언어를 지원합니다.이로 인해 거의 모든 소프트웨어 프로젝트에 적용할 수 있는 유연성을 제공합니다. 

### 재사용 가능한 컴포넌트

마켓플레이스에서 다른 개발자가 만든 액션을 재사용하거나 자신의 커스텀 액션을 만들어 공유할 수 있습니다. 

### 유연한 트리거 옵션

다양한 깃허브 이벤트에 대한 트리거 옵션을 제공합니다. 또한 cron 기반의 스케쥴을 통해서도 실행이 가능합니다. 이를 이용해 자유롭게 필요에 따라 자동화 작업을 실행할 수 있습니다. 

## Github Actions의 이해

### Workflow

워크플로우는 하나 이상의 작업을 실행하는 자동화된 프로세스입니다. 워크플로우는 리포지토리의 `.github/workflows` 디렉토리 아래 YAML 파일에 의해 정의되며 리포지토리의 이벤트 또는 스케쥴러에 따라 실행됩니다. 

### Jobs

크게 뭉뚱그려서 이야기하면 워크플로우는 결국 Job들의 집합입니다.

 Job들은 서로 다른 독립된 환경을 가집니다. 각 작업은 독립적으로 실행되며, 각기 다른 러너 환경에서 다른 버전의 도구를 사용할 수 있습니다. 이 방법을 사용하면, 하나의 워크플로우 안에서 서로 다른 환경, 서로 다른 패키지 세트를 사용하여 작업을 수행할 수 있습니다. 

#### 여러 Job의 선언

예를 들어, 첫번째 잡과 두번째 잡이 각각 다른 node 버전을 사용해야 하는 경우 다음과 같이 구성할 수 있습니다. 

```yaml
name: job1-job2

jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    - name: Use Node.js (Version for Build)
      uses: actions/setup-node@v2
      with:
        node-version: '12' # 첫 번째 작업에 필요한 노드 버전

    - name: Install Dependencies for Build
      run: npm install

    - name: Build
      run: npm run build

    - name: Additional Build Steps
      run: # 필요한 추가 빌드 스텝

  job2:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    - name: Use Node.js (Version for Deploy)
      uses: actions/setup-node@v2
      with:
        node-version: '18' # 두 번째 작업에 필요한 노드 버전

    - name: Install Dependencies for Build
      run: npm install

    - name: Build
      run: npm run build

    - name: Additional Build Steps
      run: # 필요한 추가 빌드 스텝

```

### Step

Job은 Step들의 집합입니다. Step은 실행될 셀 스크립트 또는 실행될 action들이이며, Step들은 정의된 순서대로 실행됩니다.

각 Step은 동일한 runner에서 실행되므로(Job이 가진 환경을 공유하므로) step 마다 데이터를 공유할 수 있다. 즉 한 job 안에서 빌드하는 step, 이를 테스트 하는 step을 한 번에 가질 수 있습니다.



## 참조

[https://docs.github.com/ko/actions/learn-github-actions/understanding-github-actions](https://docs.github.com/ko/actions/learn-github-actions/understanding-github-actions)



