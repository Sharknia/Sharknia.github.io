---
IDX: "NUM-34"
tags:
  - NestJS
update: "2024-02-02T16:32:00.000Z"
date: "2023-08-20"
상태: "Ready"
title: "NestJS 설치"
---
모 회사 과제 때문에 NestJS를 설치해 볼 일이 생겨 기록해둔다. 

#### Node.js 설치

NestJS는 Node.js를 기반으로 한다. [Node.js](https://nodejs.org/ko/download)에서 맞는 버전을 설치한다. 

#### NestJS 프로젝트 생성

Node.js에 npm이 포함되어있다. NestJS 서버를 구성하기 위해서는 @nestjs/cli 를 설치해야 한다. 

```bash
npm i -g @nestjs/cli
```



원하는 디렉토리로 이동해서 프로젝트를 초기화한다. 

```bash
nest new <project-Name>
```

<project-Name> 에는 원하는 프로젝트 이름을 입력한다. 패키지 매니저를 선택할 수 있는데, npm을 선택했다. 

#### NestJS 프로젝트 구동

npm run start 명령어로 구동할 수 있지만, 

NestJS에는 내부적으로 내부적으로 [**webpack**](https://webpack.js.org/)과 함께 작동하는 [`@nestjs/cli`](https://docs.nestjs.com/cli/overview)를 제공한다. 

이 CLI 도구는 소스 코드의 변경을 감지하고 자동으로 애플리케이션을 재시작하는 기능을 포함하고 있다. 

개발 모드로 애플리케이션을 실행하려면 다음 명령어를 사용하면 된다. 

```bash
npm run start:dev
```



이제 localhost:3000 에서 서버가 실행되었음을 확인할 수 있다. 



