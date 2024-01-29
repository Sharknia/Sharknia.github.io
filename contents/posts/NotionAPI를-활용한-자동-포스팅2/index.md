---
tags:
  - Notion-API
  - Blogging
  - Hobby
  - Typescript
series: "GitHub Pages와 Notion API 연동"
update: "2024-01-29"
date: "2023-09-04"
상태: "POST"
title: "NotionAPI를 활용한 자동 포스팅(2)"
---
## 지난 시간

지난시간에는 DataBase를 불러오는 비동기 로직 때문에 Factory Method를 사용해서 노션의 DataBase를 불러오는 코드를 작성했다. 

기능만 점검 후, 전체적인 설계를 다시 하고 프로젝트의 디렉토리 구조도 다시 짰다. 

## 새로운 설계

일단, 크게 네 가지의 클래스를 만들기로 했다. 

### DataBase 클래스

- 생성시에 날짜를 입력 받아 파라미터마다 다른 조건으로 쿼리 하여 노션 API에서 결과값을 받아오는 필터링 기능을 가진다. 

### Page 클래스

- Page 타입의 Block이다. 

- 해당 페이지를 마크다운으로 저장(Print)하는 메소드를 가지고 있다. 

- Property들을 가지고 있다. (페이지의 속성)

### Content 클래스

- Page에 귀속된다. 

- 종류가 여러가지이다. h1, h2, h3.. 등등 직접적으로 포스팅의 내용이 될 블록이다.

- 가장 상위의 Content는 부모가 Page이다. 

- 자기 자신을 마크다운으로 변환하여 부모에게 리턴하는 method를 가지고 있다. 만약 child가 있다면, 해당 child도 함께 변환하여 부모에게 리턴한다. 

- 일단 기본적으로 다음의 속성을 가진다. 

  - **id** : idx

  - **parent** : 부모 ID

  - **hasChidren** : 자식 요소가 있는지 여부 (true, false)

  - **type** : `heading_1` 등등 블록의 종류. 종류마다 마크다운 변환 전략이 달라야 하므로 중요하다. 

  - **is_toggleable** : **** 토글 여부

### Posting 클래스

- 생성자에 날짜를 넣어서 생성하면 해당하는 DataBase를 조회하고, Page → Content 를 돌면서 마크다운으로 페이지를 변환하여 저장하는 클래스들의 메소드를 모두 여기서 실행한다. 메인함수 같은 개념이다. 

위 내용은 언제든지 달라질 수 있다. 실제로 매일매일 하루하루 숨 쉬듯이 달라지고 있다. 

## 디렉토리 구조

src/models 디렉토리를 새로 만들고 여기에 모든 클래스, 비즈니스 로직을 넣기로 했다. 

## database.ts

일단 이번에는 database.ts를 1차적으로 완성했다. 

```typescript
import { Client } from "@notionhq/client";
import { QueryDatabaseResponse } from "@notionhq/client/build/src/api-endpoints";
import * as dotenv from "dotenv";


export class DataBase {
    private notion: Client;
    private databaseId: string;
    public database: QueryDatabaseResponse;

    public pageIds: { pageId: string }[] = [];

    private constructor(notion: Client, databaseId: string) {
        this.notion = notion;
        this.databaseId = databaseId;
        this.database = {} as QueryDatabaseResponse;
    }

    public static async create(filterUdate?: string): Promise<DataBase> {
        dotenv.config({ path: `${__dirname}/../../.env` });
        const notionkey: string = process.env.NOTION_KEY || "";
        const databaseid: string = process.env.NOTION_DATABASE_ID || "";

        if (!notionkey || !databaseid) {
            throw new Error("NOTION_KEY or NOTION_DATABASE_ID is missing in the environment variables.");
        }

        const notion = new Client({ auth: notionkey });
        const instance = new DataBase(notion, databaseid);
        if (filterUdate == "lastest") {
            const today = new Date(); // 현재 날짜와 시간을 가져옵니다.
            today.setDate(today.getDate() - 1); // 날짜를 하루 전으로 설정합니다.

            // YYYY-MM-DD 형식의 문자열로 날짜를 가져옵니다.
            filterUdate = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`;
        }
        instance.database = await instance.queryDatabase(filterUdate);
        return instance;
    }

    public async queryDatabase(filterUdate?: string): Promise<QueryDatabaseResponse> {
        try {
            const response = await this.notion.databases.query({
                database_id: this.databaseId,
                filter: {
                    and: [
                        {
                            property: '상태',
                            select: {
                                equals: "POST",
                            },
                        },
                        ...(filterUdate ? [{
                            property: 'update',
                            date: { on_or_after: filterUdate }
                        }] : []),
                    ]
                },
                sorts: [
                    {
                        property: 'update',
                        direction: 'descending',
                    },
                ],
            });
            // pageId 리스트 업데이트
            this.pageIds = response.results.map(page => ({ pageId: page.id }));
            return response;
        } catch (error) {
            console.error("Error querying the database:", error);
            throw error;
        }
    }
}
```

전에 제작한 클래스를 수정하여 만들었으며, 필터 기능을 추가하였다. 

Notion 서버에는 반드시 세계공통시? 로 저장이 되는 문제가 있다. 서울 시간대가 달라서 필터에도 현재 시간을 변환하여 조회하는 로직을 짜야 할 것 같다. 

`pageIds` 속성으로 DataBase에 저장된 페이지들의 ID를 받을 수 있다. 

## 코드 작성시 고려한 점

### DataBase 클래스가 Page 인스턴스 리스트 관리 책임을 가진다면?

- SRP(Single Responsibility Principle) 원칙에 의거해 하나의 클래스는 하나의 책임만을 져야 한다. DataBase 클래스가 Page 인스턴스 리스트 관리 책임도 가진다면 이 원칙에 어긋날 수 있다. 

- 두 클래스가 강한 연결성을 가지게 되어 유연성이 떨어질 수 있다. 

- 코드가 더 직관적으로 보일 수 있다. (추적이 쉽다)

결론적으로, DataBase 클래스와 Page 클래스의 연결을 줄이고(유연성을 늘리고) SRP를 준수하기 위해 현재의 방식을 선택했다. 

### `databaseId` , `notionkey` 를 생성자의 파라미터로 전달한다면? 

- 해당 값을 생성자의 파라미터로 값을 전달할 경우 코드가 길어지는 단점이 있다. 또한, 다양한 notionkey나 databaseid로 클래스를 생성할 수 있다. 

- 하지만, 해당 프로젝트에서 Database는 여러개를 생성하거나 값을 바꿔가면서 클래스를 생성할 일이 없으므로 현재의 방법을 택했다. 

- 다만 지금처럼 할 경우 클래스 내부에서 외부 환경 변수에 직접 접근하므로 캡슐화 원칙에 어긋나기는 한다. 하지만 앞서 언급한대로 해당 변수가 바뀔일은 거의 없다고 여겨지므로 지금의 방법을 택했다. 

## 추가 수정

### 고려사항

- DataBase는 여러개일 필요가 없으므로, Singleton 패턴을 고려한다.

- `**DataBase**` 클래스는 Notion과의 통신 뿐만 아니라, 환경 변수의 로딩 및 데이터베이스 ID 및 키의 유효성 검사까지 담당하고 있다. 이러한 기능들을 분리하여 각각의 책임을 명확히 하는 것이 좋다. 

- Typings: 현재 코드에서는 TypeScript를 사용하고 있다. 가능한 한 모든 변수, 함수 매개변수 및 반환 타입에 타입 주석을 추가하는 것이 좋다.

## 완성코드

위의 문제를 고려해 Notion과의 통신을 NotionAPI 클래스로 분리했다. Page나 Block 같은 다른 클래스에서도 동일한 Notion Client를 사용해야 하므로, 이는 아주 타당한 선택이었다. 또 하나의 프로젝트에 API는 유일하므로 싱글톤 패턴을 사용했다. 다음은 이를 위해 만들어진 notionapi.ts 코드이다. 

### notionapi.ts

```typescript
import { Client } from "@notionhq/client";

export class NotionAPI {
    private static instance: NotionAPI | null = null;
    public client: Client;

    private constructor(notionKey: string) {
        this.client = new Client({ auth: notionKey });
    }

    public static async create(notionKey: string = "") {
        if (!this.instance) {
            if (!notionKey) {
                throw new Error("NOTION_KE is missing");
            }
            this.instance = new NotionAPI(notionKey);
        }
        return this.instance;
    }
}
```

최초 클래스 생성시에만 notionKey가 필요하도록 해두었다. 나중에 이 클래스를 호출할 때에는 번거롭게 환경변수를 조회할 필요가 없다. 

### database.ts

```typescript
import { Client } from "@notionhq/client";
import { QueryDatabaseResponse } from "@notionhq/client/build/src/api-endpoints";
import * as dotenv from "dotenv";
import { NotionAPI } from "./notionapi";

export class DataBase {
    private static instance: DataBase | null = null;

    private notion: Client;
    private databaseId: string;
    public database: QueryDatabaseResponse;

    public pageIds: { pageId: string }[] = [];

    private constructor(notion: Client, databaseId: string) {
        this.notion = notion;
        this.databaseId = databaseId;
        this.database = {} as QueryDatabaseResponse;
    }

    public static async create(filterUdate?: string): Promise<DataBase> {
        if (!this.instance) {
            dotenv.config({ path: `${__dirname}/../../.env` });
            const notionkey: string = process.env.NOTION_KEY || "";
            const databaseid: string = process.env.NOTION_DATABASE_ID || "";

            if (!notionkey || !databaseid) {
                throw new Error("NOTION_KEY or NOTION_DATABASE_ID is missing in the environment variables.");
            }

            const notionApi: NotionAPI = await NotionAPI.create(notionkey);

            this.instance = new DataBase(notionApi.client, databaseid);
            if (filterUdate === "lastest") {
                const today: Date = new Date();
                today.setDate(today.getDate() - 1);
                filterUdate = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`;
            }
            this.instance.database = await this.instance.queryDatabase(filterUdate);
        }
        return this.instance;
    }

    public async queryDatabase(filterUdate?: string): Promise<QueryDatabaseResponse> {
        try {
            const response: QueryDatabaseResponse = await this.notion.databases.query({
                database_id: this.databaseId,
                filter: {
                    and: [
                        {
                            property: '상태',
                            select: {
                                equals: "POST",
                            },
                        },
                        ...(filterUdate ? [{
                            property: 'update',
                            date: { on_or_after: filterUdate }
                        }] : []),
                    ]
                },
                sorts: [
                    {
                        property: 'update',
                        direction: 'descending',
                    },
                ],
            });
            // pageId 리스트 업데이트
            this.pageIds = response.results.map(page => ({ pageId: page.id }));
            return response;
        } catch (error) {
            console.error("Error querying the database:", error);
            throw error;
        }
    }
}
```

타입 주석을 모두 추가해주었고, 싱글톤 패턴으로 변경되었다. Notion Client의 생성부가 다른 곳으로 분리되었다. 

환경변수 역시 분리하는것을 고려중이지만, 

일단은 database.ts를 이 정도로 완성 하려고 한다. 

database.ts 완성!

[https://github.com/Sharknia/Notion-to-Markdown/tree/database-class-complete](https://github.com/Sharknia/Notion-to-Markdown/tree/database-class-complete)

