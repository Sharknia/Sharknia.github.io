---
tags:
  - Hobby
  - Blogging
  - Notion-API
  - DesignPattern
series: "GitHub Pages와 Notion API 연동"
update: "2024-01-29"
date: "2023-09-03"
상태: "POST"
title: "NotionAPI를 활용한 자동 포스팅"
---
### 갑자기 짚고 넘어가는 프로젝트의 목표

이번 프로젝트의 목표는 다음과 같다. 

#### 개발 내적인 목표

1. 타입스크립트를 사용한다. 

1. 변경에는 닫혀있고, 확장에는 열린 코드를 작성한다. 

1. 이를 위해 구현에만 집중하지 않고 설계에 신경을 써서 진행해본다.

1. 이를 위해 디자인 패턴을 가능한 한 적극적으로 활용해본다. 

1. 가능한 한 사용이 쉽도록 만들어본다.

1. 명확한 명명 규칙을 사용한다.

  1. PascalCase를 클래스 이름에 사용한다.

  1. camelCase를 메소드 및 변수에 사용한다.

#### 개발 외적인 목표

1. 노션으로 공부만 해도 포스팅/풀심기가 모두 되는 꿈의 프로그램을 만든다. 

### 그래서 이번에는?

[Notion API(2)](https://sharknia.github.io/Notion-API2)  

지난번에 API 정상 작동을 확인만 한 코드를 타입스크립트로 바꾸고, 하나의 클래스로 만들려고 한다. 

#### 최초 작성 코드

```typescript
import * as dotenv from "dotenv";
import { Client } from "@notionhq/client";
import { QueryDatabaseResponse } from "@notionhq/client/build/src/api-endpoints";

class NotionToMarkdown {
    private notion: Client;
    private databaseID: string;
    private database?: QueryDatabaseResponse;

    constructor() {
        dotenv.config();
        
        if (!process.env.NOTION_KEY) {
            throw new Error("Environment variable NOTION_KEY is not defined.");
        }

        if (!process.env.NOTION_PAGE_ID) {
            throw new Error("Environment variable NOTION_PAGE_ID is not defined.");
        }

        this.notion = new Client({ auth: process.env.NOTION_KEY });
        this.databaseID = process.env.NOTION_PAGE_ID;
        this.initializeDatabase();
    }
    
    private async initializeDatabase(): Promise<void> {
        this.database = await this.queryDatabase();
    }
    
    public async queryDatabase(): Promise<QueryDatabaseResponse> {
        try {
            const response = await this.notion.databases.query({ database_id: this.databaseID });
            return response;
        } catch (error) {
            console.error("Error querying the database:", error);
            throw error;
        }
    }
    // ... 다른 메소드들 (retrievePage, retrieveBlock, listBlockChildren) ...
}
```

이런 식으로 구성을 했는데 위 코드의 문제는 생성자에서 비동기 작업을 직접 실행하는 것에 있다. 

비동기 작업을 생성자에서 실행할 경우 잠재적으로 발생할 수 있는 문제는 다음과 같다.

1. 비동기 로직이 실패할 경우, 객체 생성 자체에 문제가 생길 수 있다. 생성자는 객체 초기화를 수행하는 로직만 포함해야 하며, 부작용(side-effect)이 발생할 수 있는 코드는 포함되어서는 안된다. 

1. 비동기 로직이 포함되면 생성자의 수행 시간이 길어질 수 있다. 

이 문제는 Factory 패턴을 사용하여 해결할 수 있다. Factory 메소드를 사용하여 비동기 로직을 수행하고 완료되면 객체를 반환한다. 아래와 같이 Factory 패턴을 사용하여 코드를 수정했다.  

#### Factory Pattern을 활용한 코드

```typescript
class NotionToMarkdown {
    private notion: Client;
    private databaseId: string;
    public database: QueryDatabaseResponse;

    private constructor(notion: Client, databaseID: string, database: QueryDatabaseResponse) {
        this.notion = notion;
        this.databaseId = databaseID;
        this.database = database;
    }

    public static async create(): Promise<NotionToMarkdown> {
        dotenv.config();

        if (!process.env.NOTION_KEY || !process.env.NOTION_DATABASE_ID) {
            throw new Error("Environment variable is not defined.");
        }

        const notion = new Client({ auth: process.env.NOTION_KEY });
        const databaseId = process.env.NOTION_DATABASE_ID;
        const database = await new NotionToMarkdown(notion, databaseId, {} as QueryDatabaseResponse).queryDatabase();

        return new NotionToMarkdown(notion, databaseId, database);
    }

    public async queryDatabase(): Promise<QueryDatabaseResponse> {
        try {
            const response = await this.notion.databases.query({ database_id: this.databaseId });
            return response;
        } catch (error) {
            console.error("Error querying the database:", error);
            throw error;
        }
    }

    // ... 다른 메소드들 (retrievePage, retrieveBlock, listBlockChildren) ...
}

// 사용 예제:
NotionToMarkdown.create().then(handler => {
    console.log(handler.database);  // database 속성 출력
});
```

위 코드에서 create 메소드가 비동기 팩토리 메소드이다. 이 메소드는 `NotionToMarkdown` 객체를 비동기적으로 생성하고 반환한다. 생성자는 private로 선언되어 직접 호출할 수 없고, 반드시 create 메소드를 통해 객체를 생성해야 한다. 

### 진행 코드

[https://github.com/Sharknia/Notion-to-Markdown/blob/42911777c0146c4260a5769fe9a7d5b1f9ac4c32/notionApi.ts](https://github.com/Sharknia/Notion-to-Markdown/blob/42911777c0146c4260a5769fe9a7d5b1f9ac4c32/notionApi.ts)



