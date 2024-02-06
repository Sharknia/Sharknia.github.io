---
IDX: "NUM-127"
tags:
  - Typescript
description: "타입스크립트에서 테스트코드를 맛보았습니다. "
update: "2024-02-06T14:48:00.000Z"
date: "2024-02-06"
상태: "Ready"
title: "Typescript의 Testcode 맛보기"
---
## 서론

파이썬에서 잠깐 테스트코드를 맛본적이 있습니다. 문법 자체는 조금 복잡했지만(익숙하지 않았지만) 설치 자체는 그다지 어렵지 않았던 것으로 기억합니다. 

그래서 타입스크립트에서도 그럴 줄 알았습니다. 

## 설치 

일단, 설치가 한 두개가 아니었습니다. 또, 설치하는 패키지를 package.json 파일의 devDependencies에 추가했습니다. 이는 개발 시에만 필요한 의존성을 나타냅니다. 처음에는 멋모르고 다음의 패키지 하나만 설치해주었습니다. 

### @types/jest 설치

```shell
npm i --save-dev @types/jest
```

#### `--save-dev`

설치하는 패키지를 package.json 파일의 devDependencies에 추가하는 옵션이 바로 이 옵션입니다. 프로덕션에서는 필요하지 않은 테스트 라이브러리, 빌드 도구 등을 설치할 때에 해당 옵션을 붙여 설치합니다. 이 옵션을 붙여 설치할 경우, 다음과 같이 추가됩니다. 

```yaml
"devDependencies": {
    "@types/jest": "^26.0.0"
}
```

이후 빌드 시스템이나 배포 파이프라인에서 `npm install --production` 명령어를 사용하면 이 곳에 정의된 패키지들은 설치를 건너뛰게 됩니다. 

#### `@types/jest`

Jest는 자바스크립트 테스팅 프레임워크입니다. `@types/jest` **** 는 Jest의 타입 정의를 포함하는 타입스크립트 패키지입니다. 타입스크립트 프로젝트에서 Jest를 사용할 때, Jest의 함수와 객체에 대한 자동 완성, 타입 체킹 등의 이점을 제공합니다. 

그렇습니다. Jest는 별도로 설치해야 합니다. 

### Jest 설치

Jest 역시 설치해줍니다. 역시 개발 환경에서만 사용할 라이브러리 이므로 —save-dev 옵션을 사용합니다.

```bash
npm install --save-dev jest
```

Jest 설치 후 `package.json`  scripts 섹션에 다음의 내용을 추가합니다. 

```json
"scripts": {
    "test": "jest"
}
```

이렇게 하면 `npm test` 명령어가 Jest를 실행합니다. 

### TypeScript를 위한 Jest 설정

Jest는 기본적으로 JavaScript 파일을 지원하지만 TypeScript 파일을 처리하기 위해서는 추가적인 설정이 필요합니다. 

#### `ts-jest` 설치

Jest가 TypeScript 파일을 이해할 수 있도록 ts-jest를 설치해야 합니다. 

```json
npm install --save-dev ts-jest
```

#### jest.config.js

프로젝트 루트에 jest.config.js 파일을 생성 또는 수정하여 ts-jest를 사용하도록 Jest를 구성합니다. 

```json
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
};
```

### `npm test`

모든 설정을 마친 후 npm test를 실행하면 `__test__` 를 자동으로 탐색하여 `.test.js` 또는 `.spec.js` 접미사를 가진 파일의 테스트를 실행합니다. 

## 테스트 파일의 경로

테스트 파일을 어디에 둘 것인가? 는 몇가지 패턴이 있고, 가장 널리 사용되는 방식은 다음의 두가지입니다. 

### 동일 디렉토리 내에 테스트 파일 배치 

코드 파일과 동일한 디렉토리에 그에 해당하는 테스트 파일을 둡니다. 테스트 파일과 관련된 코드 파일이 가까이에 있어서 관리하기가 쉬운것이 장점입니다. 파일 이름은 대체로 테스트되는 파일의 이름을 따르며, `.test` 또는 `.spec` 접미사를 추가하여 구분합니다. 

### 별도의 테스트 디렉토리 사용

프로젝트 루트 또는 각 기능별 디렉토리에 `__tests__` 디렉토리를 만들고, 그 안에 관련 테스트 파일을 모으는 것입니다. 이 구조는 테스트 파일을 소스 코드에서 분리하여, 소스 코드 디렉토리를 더 깔끔하게 유지할 수 있게 해줍니다.

또는 프로젝트 루트에 `tests` 디렉토리를 만들고 그 안에 모든 테스트 파일을 분류하는 방법도 있습니다.

## 오늘의 마무리

여기까지 하면 일단 test 코드를 실행할 수는 있습니다. 코파일럿의 도움을 받아 테스트코들 작성하고, 코드를 읽는데까지는 성공했습니다(?)

예를 들어 완성한 테스트 코드 하나는 다음과 같습니다. 

<details>
<summary>envConfig.test.ts</summary>

```python
import { EnvConfig } from '../envConfig';

describe('EnvConfig', () => {
    let envConfig: EnvConfig;
    beforeAll(() => {
        // 환경 변수 설정
        process.env.BLOG_URL = 'https://example.com/';
        process.env.SAVE_DIR = '/path/to/save';
        process.env.SAVE_SUB_DIR = 'subdir';

        // EnvConfig 인스턴스 생성
        envConfig = EnvConfig.create();
    });

    afterAll(() => {
        // 환경 변수 정리
        delete process.env.BLOG_URL;
        delete process.env.SAVE_DIR;
        delete process.env.SAVE_SUB_DIR;
    });

    it('should have the correct notionKey value', () => {
        expect(envConfig.notionKey).toEqual(process.env.NOTION_KEY || '');
    });

    it('should have the correct databaseid value', () => {
        expect(envConfig.databaseid).toEqual(
            process.env.NOTION_DATABASE_ID || '',
        );
    });

    it('should correctly handle trailing slashes', () => {
        expect(envConfig.blogUrl).toEqual('https://example.com'); // 끝 슬래시 제거 확인
        expect(envConfig.saveDir).toEqual('/path/to/save/'); // 끝에 슬래시 추가 확인
        expect(envConfig.saveSubDir).toEqual('subdir/'); // 끝에 슬래시 추가 확인
    });
});
```


</details>

테스트코드를 작성하면서 느낀점은, 역시 테스트코드는 사전에 짜는게 의미가 있는 것 같습니다. 또 역시 문법이 상당히 어색합니다. 추가적인 노력이 많이 필요하다고 느끼고, 가능하면 다음에는 프로젝트 처음부터 테스트 코드를 먼저 짜보려고 합니다. 

오히려 좀 더 필요성을 느끼게 되었습니다. 살짝 고생했지만 좋은 시간이었습니다. 

