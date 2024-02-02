---
IDX: "NUM-110"
tags:
  - Blogging
  - Typescript
  - Hobby
  - Notion-API
description: "삭제/수정 기능 구현"
series: "GitHub Pages와 Notion API 연동"
update: "2024-02-02T16:32:00.000Z"
date: "2024-02-02"
상태: "Ready"
title: "NotionAPI를 활용한 자동 포스팅(9)"
---
## 지난시간

지난시간, 신나게 자동 배포를 만들었다. 

이제 새로 생성된 문서들에 대해서 자동으로 한시간마다 배포가 되어 글이 포스팅된다. 



**근데, 이런 경우 글을 어떻게 삭제하지?**

**현재 타이틀이 키 값인데 타이틀 명이 바뀌면 내용은 똑같은데 제목만 다른 글이 두 개?** 



그렇다, 삭제 및 수정 기능이 구현되어야 하는 것이다. 

## 설계

어떤걸 키 값으로 해야 하나 고민이 많았는데, 답은 간단했다. 노션에 ID를 추가하는 기능이 있었다. 이렇게 되면, 내 노션 블로그를 위한 필수 속성은 타이틀, 상태, ID 세 가지가 된다. 이런게 너무 많이 생기는걸 원하지 않지만 어쩔 수 없는 부분이라고 생각이 든다. 

계획은 다음과 같다. 

- 노션 글들에 ID 속성을 추가해 키값으로 삼는다. 

- 상태값에 ToBeDeleted, Deleted를 추가한다. 

- 키 값 - 저장된 디렉토리가 매핑된 메타데이터 json 파일을 생성한다. 

- ToBeDeleted를 쿼리해와서, 매핑 테이블을 참조해 데이터를 삭제 후 매핑 정보도 삭제하고 Deleted 상태로 바꿔준다. 

- Ready를 쿼리해와서 매핑 테이블을 참조해 기존 데이터를 삭제 후 매핑 정보를 갱신하고 마크다운을 다시 저장한다. 

이렇게 하면 수정과 삭제에 모두 대응이 가능해보인다!

## 구현

우선, 전역적으로 메타데이터를 읽어오고, 메모리에 데이터를 저장한 다음 프로그램 종료 직전에 메타 데이터를 한 번에 수정해주는 역할을 할 클래스를 생성했다. 

<details>
<summary>metadataManager.ts</summary>

```typescript
import { promises as fs } from 'fs';

const METADATA_FILE_PATH = './pageMetadata.json';

interface PageMetadata {
    url: string;
}

interface Metadata {
    [pageId: string]: PageMetadata;
}

export class MetadataManager {
    private static instance: MetadataManager;
    private metadata: Metadata | null;

    private constructor() {
        this.metadata = null;
    }

    /**
     * 인스턴스를 반환하는 메서드입니다.
     * @returns {MetadataManager} MetadataManager 인스턴스
     */
    public static getInstance(): MetadataManager {
        if (!this.instance) {
            this.instance = new MetadataManager();
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
     * @param pageId 페이지 식별자
     * @param pageData 페이지 메타데이터
     */
    public updatePageMetadata(pageId: string, pageData: PageMetadata): void {
        if (!this.metadata) {
            this.metadata = {};
        }
        this.metadata[pageId] = pageData;
    }

    /**
     * 페이지 메타데이터를 삭제합니다.
     * @param pageId 삭제할 페이지의 ID
     */
    public deletePageMetadata(pageId: string): void {
        if (this.metadata && this.metadata[pageId]) {
            delete this.metadata[pageId];
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
            } catch (error) {
                console.error('메타데이터 파일 저장 오류:', error);
            }
        }
    }
}
```


</details>

이 클래스는 프로그램 시작과 동시에 초기화되며, 끝날 때 메타데이터 파일을 갱신하는 역할을 할 것이다. 

정확한 매개변수는 아직 정하지 않았으며, 초안만 짜뒀다. 

이제 내일, 이 클래스를 명확히 정의하고 이 클래스를 사용해 수정/삭제를 구현하면 된다. 

