---
IDX: "NUM-125"
tags:
  - Notion-API
  - Blogging
  - Hobby
  - Typescript
description: "블로그 자동 배포까지 한 번에 해결하는 Workflow 수정 및 타임아웃 에러 대처 코드 삽입"
series: "GitHub Pages와 Notion API 연동"
update: "2024-02-08T15:39:00.000Z"
date: "2024-02-04"
상태: "Ready"
title: "NotionAPI를 활용한 자동 포스팅(11)"
---
## 요약

짬짬이 틈을 내서 하는 작업이다 보니 문서화 작업을 제대로 못했습니다. 짜잘짜잘한 오류 수정이 있었으며, 비교적 큰 수정도 있었고 완성이 됐다고 판단해 버전을 정의하고 릴리즈도 해봤습니다. 

## 버그 수정

- 코드 블록의 들여쓰기가 제대로 적용되지 않는 문제가 수정되었습니다. 

- `코드`  스타일 내에서 언더바가 제대로 인식되지 않는 문제가 수정되었습니다. 

- 타이틀에 스타일이 포함된 경우(노션에서 확인하기 어려움) 타이틀이 짤리던 문제가 수정되었습니다. 

- 기타 일부 스타일이 적용되지 않던 문제가 해결됐습니다. 

- API 호출 시 타임아웃이 될 경우 대책없이 프로그램이 중단되던 문제가 해결됐습니다. 

## 업데이트

- 이제 토글 블록이 지원됩니다.

- 블로그 자동 배포가 워크플로우에 포함되었습니다. 이를 위해 깃허브 계정을 미리 설정할 수 있게 되었습니다. 

- 메타데이터 초기화 기능이 추가되었습니다. 이제 새로운 계정으로 세팅 시 메타데이터 파일이 초기화됩니다.

- 코드 리팩토링의 일부로 일부 코드의 디렉토리가 분리되었습니다. 

- `v1.0.0-beta`가 릴리즈 되었습니다. Readme가 완성되었습니다. 

- 프로젝트 이름이 `nolog`로 확정되었습니다.

- 라이센스가 명시되었습니다. 

## 타임아웃 문제의 해결

기타 다른 버그는 짜잘짜잘한 수정이었지만 타임아웃 문제 해결은 나름대로 머리를 써서 구현했으므로 해당 과정을 별도로 기록합니다. 

### 문제 파악

노션 API를 사용하고 있고, 해당 API를 사용하기 위해서 notionClient 라이브러리를 사용하고 있습니다. 그리고 주로 데이터베이스 정보를 읽어올 때, 페이지 안의 내용을 읽어올 때, 블록 안의 내용을 읽어올 때 해당 라이브러리를 사용하여 통신합니다. 

하지만 가끔 가다 간헐적으로 규칙성 없이 timeout이 발생하는 문제가 있었습니다. 이를 해결하기 위해 Notion Client 라이브러리를 상속한 NotionClientWithRetry 클래스를 생성하고, 세가지 메소드에 대해 재시도 로직을 하는 공통 메소드를 생성하고 코드상에서 Client 직접 호출 대신 해당 메소드를 이용해 Client를 호출하도록 수정해주었습니다. 

<details>
<summary>NotionClientWithRetry.ts</summary>

```typescript
import { Client } from '@notionhq/client';

export class NotionClientWithRetry extends Client {
    private maxRetries: number;
    private retryDelay: number;

    constructor(options: {
        auth: string;
        maxRetries?: number;
        retryDelay?: number;
    }) {
        super({ auth: options.auth });
        this.maxRetries = options.maxRetries || 3;
        this.retryDelay = options.retryDelay || 1000; // 기본 1초 대기
    }

    private async retryAPI<T>(
        operation: () => Promise<T>,
        retries: number = this.maxRetries,
    ): Promise<T> {
        try {
            return await operation();
        } catch (error) {
            if (retries <= 0) throw error;
            console.log(
                `[retryAPI] ${this.maxRetries - retries} retrying.....`,
            );
            await new Promise((resolve) =>
                setTimeout(resolve, this.retryDelay),
            );
            return this.retryAPI(operation, retries - 1);
        }
    }

    public async blocksRetrieve(args: { block_id: string }) {
        return this.retryAPI(() => this.blocks.retrieve(args));
    }

    public async databasesQuery(args: {
        database_id: string;
        filter?: any;
        sorts?: any;
    }) {
        return this.retryAPI(() => this.databases.query(args));
    }

    public async pagesRetrieve(args: { page_id: string }) {
        return this.retryAPI(() => this.pages.retrieve(args));
    }
}
```


</details>

따로 사용자가 설정할 수 있지만 기본적으로는 1초에 한 번씩 최대 3번까지 재시도를 하도록 해주었습니다. 

해당 코드를 적용한 이후로는 API 호출에 실패해서 프로그램이 중단되는 일은 겪지 못했습니다. 

## 블로그 자동 배포가 포함된 워크플로우 작성

블로그 자동 배포 워크플로우는 사실 [github.io](http://github.io/) 쪽 레포에 이미 구현이 되어 있어 추가 구현이 필요 없었으며, 깃허브 블로그 테마마다 배포 방식이 다를 수 있어 사실 꼭 구현이 필수인 영역은 아니었습니다. 

하지만 깃허브 액션에 대해 이해도가 깊어질 수 있고, 또 프로젝트의 목적 상 가능하면 하나로 합쳐지는 예시를 제공하는 것도 좋아보여 공부 겸 진행했습니다. 



