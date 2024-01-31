---
tags:
  - Typescript
update: "2024-01-29"
date: "2023-09-02"
상태: "POST"
title: "Typescript 시작하기"
---
## TypeScript

### TypeScript란?

TypeScript는 Microsoft에서 개발한 오픈 소스 프로그래밍 언어이다. JavaScript의 SuperSet(상위 집합)이므로, 기존의 자바스크립트 코드도 타입스크립트에서 동작한다. 타입스크립트의 주요 목적은 큰 규모의 어플리케이션 개발을 돕기 위해 정적 타입, 인터페이스, 클래스, 모듈 등의 기능을 자바스크립트에 추가하는 것이다. 

### 주요 특징

1. 정적 타이핑 : 변수, 함수 매개변수, 함수 반환 값에 대한 타입을 지정할 수 있다. 이를 통해 컴파일 시점에 오류를 발견할 수 있으며, IDE와 에디터는 타입 정보를 사용하여 더 나은 코드 제안과 리팩토링을 제안할 수 있다. 

1. 객체 지향 프로그래밍 : 클래스, 인터페이스, 상속, 제네릭 등과 같은 객체 지향 프로그래밍 기능을 제공하여 코드의 구조와 디자인을 개선한다. 

1. 도구 및 에디터 지원 : 코드 자동완성, 타입검사, 코드 리팩토링과 같은 기능을 지원하는 다양한 도구 및 에디터에 통합된다. 

1. 다운레벨 타겟팅 : 최신 자바스크립트 기능을 사용하여 코드를 작성하고 ES3, ES5, ES6등 이전 버전의 자바스크립트로 컴파일 할 수 있다. 이를 통해 최신 기능을 사용하면서도 구버전 브라우저와의 호환성을 유지할 수 있다. 

1. 확장성 : 모듈화 된 방식으로 동작하여 외부 라이브러리와의 통합을 쉽게 만들어준다. 

## TypeScript의 설치

1. TypeScript 설치

    ```bash
    npm install -g typescript
    ```

1. 새 프로젝트 시작

    ```bash
    mkdir my-ts-project
    cd my-ts-project
    npm init -y
    ```

1. TypeScript 설정 파일 생성

    ```bash
    tsc --init
    ```

    `tsconfig.json` 파일이 생성된다. 이 파일에서 TypeScript의 컴파일 옵션을 설정할 수 있다.

1. 필요한 TypeScript 타입 정의 설치

    Node.js와 TypeScript를 함 사용하기 위해 해당 타입 정의를 설치한다. 

    ```bash
    npm install --save @types/node
    ```

1. 첫 TypeScript 파일 작성

    `index.ts` 파일에 다음과 같은 내용을 작성해보자. 

    ```typescript
    const greet = (name: string): string => {
        return `Hello, ${name}!`;
    }
    
    console.log(greet("TypeScript"));
    ```

1. 컴파일 및 실행

    아래 명령어로 컴파일 할 수 있다. 

    ```bash
    tsc index.ts
    ```

    index.js가 생성되었다. 이제 Node.js로 실행할 수 있다. 

    ```bash
    node index.js
    ```

1. 자동 컴파일 설정(선택사항)

    `ts-node` 라는 패키지를 설치하여 TypeScript 코드를 바로 실행할 수 있다.

    ```bash
    npm install -g ts-node
    ```

    설치 후, 아래 명령어로 TypeScript 코드를 바로 실행할 수 있다.

    ```bash
    ts-node index.ts
    ```

## TypeScript 설정 파일(tsconfig.json)

`tsconfig.json` 은 타입스크립트 프로젝트의 루트 디렉토리에 위치하는 설정 파일이다. 타입스크립트 컴파일러에 대한 구성 옵션을 정의한다. 이 파일을 통해 타입스크립트 코드가 자바스크립트 코드로 변환되는지, 어떤 파일들이 포함되거나 제외되는지 등을 정의할 수 있다. 

주요 옵션들은 다음과 같다. 

- files: 컴파일에 포함할 파일의 목록이다. 이 옵션을 사용하면 특정 파일만 명시적으로 포함시킬 수 있다.

- include: 컴파일에 포함할 파일이나 디렉토리의 패턴 목록다. Glob 패턴을 사용할 수 있다. 예: `["src/**/*.ts"]`

- exclude: 컴파일에서 제외할 파일이나 디렉토리의 패턴 목록다. 예: `["node_modules"]`

- extends: 다른 `tsconfig.json` 파일을 기반으로 현재 설정을 확장하려면 파일 경로를 지정한다.

- compilerOptions: 컴파일러에 대한 다양한 설정을 제공하는 핵심 옵션이다.

- typeRoots와 types: 사용자 정의 타입 선언의 위치와 포함될 타입 선언 패키지를 지정한다.

compilerOptions에 대해 좀 더 자세히 알아보면 다음과 같다. 

- target: 컴파일된 JavaScript의 버전을 지정한다. 예: ES5, ES6 등

- module: 모듈 시스템을 지정한다. 예: CommonJS, ESNext, UMD 등

- outDir: 컴파일된 JavaScript 파일이 저장될 디렉토리를 지정한다.

- rootDir: 입력 파일의 루트 디렉토리를 지정한다.

- strict: 모든 엄격한 타입 검사 옵션을 활성화한다.

- noImplicitAny: 암시적인 'any' 타입이 없는 경우 오류를 발생시킨다.

- esModuleInterop: CommonJS와 ES 모듈 간의 상호 운용성을 위한 코드를 생성한다.

- skipLibCheck: 선언 파일의 타입 검사를 건너뛴다.

