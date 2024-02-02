---
IDX: "NUM-117"
tags:
  - Notion-API
  - Blogging
  - Hobby
  - Typescript
description: "삭제/수정 기능 구현 2"
update: "2024-02-02T17:43:00.000Z"
date: "2024-02-02"
상태: "Ready"
title: "NotionAPI를 활용한 자동 포스팅(10)"
---
## 메타데이터 구조 확정

지난시간, [메타데이터를 관리하기 위한 클래스 초안](https://sharknia.github.io/NotionAPI를-활용한-자동-포스팅9)을 작성했다. 시간이 없어서 급하게 만들었는데, 제대로 매개변수 이름을 지어주고 메타데이터 구조를 확정지었다. 

크게 변하지는 않았고, 이름을 정해주는 수준이었다. 

그리고 추가로 저장된 디렉토리 명을 사용해 포스팅된 마크다운 디렉토리를 삭제하는 메소드를 추가해주었다. 

<details>
<summary>수정된 코드 전문</summary>

```typescript
import { promises as fs } from 'fs';
import { join } from 'path';
import { EnvConfig } from './envConfig';

const METADATA_FILE_PATH = './pageMetadata.json';

interface PageMetadata {
    path: string;
}

interface Metadata {
    [pageIdx: string]: PageMetadata;
}

export class MetadataManager {
    private static instance: MetadataManager;
    private metadata: Metadata | null;
    private envConfig: EnvConfig;

    private constructor() {
        this.metadata = null;
        this.envConfig = EnvConfig.create();
    }

    /**
     * 인스턴스를 반환하는 메서드입니다.
     * @returns {MetadataManager} MetadataManager 인스턴스
     */
    public static async getInstance(): Promise<MetadataManager> {
        if (!this.instance) {
            this.instance = new MetadataManager();
            await this.instance.loadMetadata();
        }
        return this.instance;
    }

    /**
     * 메타데이터를 로드합니다.
     * @returns {Promise<void>} Promise 객체
     */
    public async loadMetadata(): Promise<void> {
        try {
            const data = await fs.readFile(METADATA_FILE_PATH, 'utf8');
            this.metadata = JSON.parse(data) as Metadata;
            console.log('메타데이터 파일 읽기 성공:', this.metadata);
        } catch (error) {
            console.error('메타데이터 파일 읽기 오류:', error);
            this.metadata = {};
        }
    }

    /**
     * 메타데이터를 반환합니다.
     * @returns 메타데이터 객체 또는 null
     */
    public getMetadata(): Metadata | null {
        return this.metadata;
    }

    /**
     * 페이지 메타데이터를 업데이트합니다.
     *
     * @param pageIdx 페이지 식별자
     * @param pageData 페이지 메타데이터
     */
    public updatePageMetadata(pageIdx: string, pageData: PageMetadata): void {
        if (!this.metadata) {
            this.metadata = {};
        }
        this.metadata[pageIdx] = pageData;
        console.log(`메타 데이터 업데이트 [${pageIdx}]`);
    }

    /**
     * 페이지 메타데이터를 삭제합니다.
     * @param pageIdx 삭제할 페이지의 ID
     */
    public deletePageMetadata(pageIdx: string): void {
        if (this.metadata && this.metadata[pageIdx]) {
            delete this.metadata[pageIdx];
        }
    }

    /**
     * 메타데이터를 파일에 저장합니다.
     * @returns 메타데이터가 성공적으로 저장될 때 해결되는 Promise입니다.
     */
    public async saveMetadata(): Promise<void> {
        if (this.metadata) {
            try {
                await fs.writeFile(
                    METADATA_FILE_PATH,
                    JSON.stringify(this.metadata, null, 2),
                );
                console;
            } catch (error) {
                console.error('메타데이터 파일 저장 오류:', error);
            }
        }
    }

    /**
     * 지정된 페이지 인덱스에 대한 메타데이터를 삭제합니다.
     * @param pageIdx 삭제할 페이지 인덱스
     * @returns 삭제 작업이 완료된 후에는 아무 값도 반환하지 않습니다.
     */
    public async deleteFromMetadata(pageIdx: string): Promise<void> {
        if (this.metadata && this.metadata[pageIdx]) {
            let dir = join(
                this.envConfig.saveDir!,
                this.metadata[pageIdx].path,
            );
            try {
                await fs.unlink(dir);
                console.log('파일 삭제 성공:', dir);
            } catch (error) {
                console.error('파일 삭제 오류:', error);
            }
        }
    }
}
```


</details>

## Posting 클래스 수정

메타데이터 정보는 모아두었다가, 프로그램 종료시에 한 번에 파일에 저장하도록 종료되는 부분에서 saveMetadata 메소드를 호출해주었다. 

<details>
<summary>수정된 Posting 클래스 코드</summary>

```typescript
public async start(): Promise<void> {
        console.log('[posting.ts] start!');
        try {
            this.metadataManager = await MetadataManager.getInstance();
            this.EnvConfig = EnvConfig.create();
            const notionkey: string = this.EnvConfig.notionKey || '';
            const databaseid: string = this.EnvConfig.databaseid || '';
            this.notionApi = await NotionAPI.create(notionkey);
            this.dbInstance = await DataBase.create(databaseid, '');

            console.log('[posting.ts] page 순회 시작');
            for (const item of this.dbInstance.pageIds) {
                const page: Page = await Page.create(item.pageId);
            }
            this.metadataManager.saveMetadata();
        } catch (error) {
            console.error('Error creating database instance:', error);
        }
    }
```




</details>

## Database 클래스 수정

기존에는 상태값이 Ready 인 것만 쿼리하고 있었는데, 이제는 삭제도 진행해야 하므로 상태가 ToBeDeleted인 것도 쿼리하도록 수정해주었다. 

<details>
<summary>수정된 코드</summary>

```typescript
public async queryDatabase(): Promise<QueryDatabaseResponse> {
        try {
            const response: QueryDatabaseResponse =
                await this.notion.databasesQuery({
                    database_id: this.databaseId,
                    filter: {
                        or: [
                            {
                                property: '상태',
                                select: {
                                    equals: PageStatus.Ready,
                                },
                            },
                            {
                                property: '상태',
                                select: {
                                    equals: PageStatus.ToBeDeleted,
                                },
                            },
                        ],
                    },
                });
            // pageId 리스트 업데이트
            this.pageIds = response.results.map((page) => ({
                pageId: page.id,
            }));
            return response;
        } catch (error) {
            console.error('Error querying the database:', error);
            throw error;
        }
    }
```


</details>

## Page 클래스 수정

### init() 메소드 수정

- pageIdx 속성을 추가하고 init() 메소드에서 해당 속성을 초기화해준다.

### create() 메소드 수정

- create 메소드에서 상태값에 따른 분기처리를 추가했다. 

    기존에는 ready상태만 있었으므로 일관된 처리를 진행하면 됐지만, 이제는 삭제 대기 상태가 추가되었으므로 상태값에 따른 분기처리를 추가하고 각각 다르게 작동하도록 해주었다. 

    1. Ready, ToBeDeleted 두 상태 모두 일단 해당하는 파일을 삭제한다. Ready 인데도 삭제하는 이유는 타이틀이 바뀐 경우에는 파일을 삭제해야 하기 때문이다. 

    1. Ready 상태인 경우에는 기존의 로직을 실행하고, Metadata를 업데이트 해준다. 

    1. ToBeDeleted 상태인 경우에는 메타데이터에서 해당하는 내용을 삭제한다. 

<details>
<summary>create() 메소드 안쪽 수정된 내용 코드</summary>

    ```typescript
    public static async create(pageId: string) {
            const notionApi: NotionAPI = await NotionAPI.create();
            const page: Page = new Page(pageId, notionApi.client);
            MarkdownConverter.imageCounter = 0;
            await page.init(page);
            const status = page.properties!['상태'];
            console.log(
                `[page.ts] start - pageTitle : (${status})${page.pageTitle}`,
            );
            // 저장하기 전에도 기존 파일을 삭제한다. 타이틀이 달라진 update 일 수 있기 때문이다.
            await page.metadataManager?.deleteFromMetadata(page.pageIdx!);
            if (status === PageStatus.ToBeDeleted) {
                // 페이지가 삭제될 예정인 경우
                await page.metadataManager?.deletePageMetadata(page.pageIdx!);
                await page.updatePageStatus(PageStatus.Deleted);
                return page;
            } else if (status === PageStatus.Ready) {
                // 포스팅이 준비된 경우
                page.contentMarkdown = await page.fetchAndProcessBlocks();
                await page.printMarkDown();
                await page.metadataManager?.updatePageMetadata(page.pageIdx!, {
                    path: page.pageUrl!,
                });
                await page.updatePageStatus(PageStatus.Updated);
                return page;
            } else {
                console.error(`[page.ts] start - status : ${status}`);
                throw new Error(`[page.ts] start - status : ${status}`);
            }
        }
    ```


</details>

## 기타 변경 내용

- 열거형 클래스를 새로 만들어 상태값 정의를 미리 해주었다. 

- 또한, 메타데이터 파일도 레포의 일부인데 깃허브 액션 워크플로우 작동 후 이 파일도 수정될 것이므로 한 번 더 커밋/푸시가 필요하다. 해당 내용을 워크플로우의 Run Script 바로 다음에 추가해주었다. 

<details>
<summary>추가된 워크플로우</summary>

    ```yaml
    - name: Commit and Push Changes to Current Repository
          run: |
            git config --global user.name 'name'
            git config --global user.email 'mail'
            git add .
            git commit -m "Update contents" || echo "No changes to commit in current repo"
            git push
    ```


</details>

## 주의할 점

일단, 로컬에서 실행 시 권한 문제로 파일 삭제가 제대로 되지 않았다. 그래서 깃허브 액션에서도 파일 권한 문제가 생길 우려가 있어 권한을 조정하는 방법이 있나 찾아보았는데, 깃허브 액션에서는 외부를 컨트롤 하려는게 아닌 이상 별도의 권한 문제가 발생하지 않는다고 한다. 

따라서 로컬에서 발생하는 문제는 따로 수정하지 않았다. 

