---
tags:
  - Blogging
  - Notion-API
  - Hobby
description: "프로젝트 복귀를 환영합니다. "
series: "GitHub Pages와 Notion API 연동"
update: "2024-01-31"
date: "2024-01-26"
상태: "Ready"
title: "NotionAPI를 활용한 자동 포스팅(3)"
---
## 지난시간

[https://sharknia.github.io/Notion-Api-2/](https://sharknia.github.io/Notion-Api-2/)

## 문제점

아무것도.. 기억이 나지 않는다.. 지난날의 나는 무엇이었나? 5개월만의 복귀가 이렇게 어렵다. 이래서 사람은 꾸준해야 한다. 

## 잡설

최근 업무에 약~간의 여유가 생기면서 IDE를 파이참에서 vs code로 갈아탔다. 파이썬만 할 때에는 파이참이 유리한 것이 사실이지만(심지어 속도도 파이참이 더 빠르다고 느꼈다. ) 추후 여러 언어를 다루게 될 경우 vs code가 유리한 점이 있다고 생각되어 갈아탔고, 환경 세팅을 다시 했다. 

formatter나 기타 여러가지 등등.. 그리고 타입스크립트 코드를 오랜만에 보면서 여기에도 formatter 설정을 추가해주었다. 

## 설계 되새김질

코드를 다시 읽는데에만 시간을 꽤 투자했다. 예전의 나는 코드를 꽤 잘 짠 것 같다. 

블로그 글을 복습하지 않고 코드를 읽고 설계를 다시 했는데 오늘의 나는 예전의 나와 의견이 똑같다. (진작 복습할걸)

### 그래서 어떻게 할 것이냐? 

노션은 블록 기반의 구조로 이루어져있다. 그래서 나는 노션의 메인 데이터베이스를 블로그 홈이라고 가정하고, 그 안에 있는 블록들의 리스트는 페이지라고 정의하여, 페이지의 양식이나 속성은 고정해두려고 한다. 그리고 비로소 페이지안의 컨텐츠, 블록들을 마크다운으로 변환하려고 한다. 

그래서, Page.ts에서 페이지 들을 관리하고, Block.ts에서 블록들을 관리하려고 한다. 블록들은 각자 하위 블록을 다시 가질 수 있는 재귀적 형태를 가진다. 

콘텐츠들은 문단, 이미지, 리스트 등 여러 타입을 가지므로 경우에 따라서 block 클래스를 상속한 하위 클래스가 생길수도 있겠다. 

각자 블록들은 자신의 내용들을 마크다운으로 변환해서 상위 블록에게 전달하는 메소드를 갖는다. 가장 하위 블록부터 전달된 변경된 마크다운 내용들은 상위로 타고 올라가 결국 페이지 클래스에 전달된다. 

페이지 클래스는 이 마크다운들을 모아 파일로 저장한다. 

계획은 완벽해 보인다. 

하지만, 타입이 아주 많고 이걸 마크다운으로 변환하는 작업은 노가다 그 자체일것이다.. 

그래서 일단은 제한된 타입들에 대해서만 변환을 지원하려고 한다. 

## 그래서 오늘 작업은? 

일단 코드를 읽었고, 분석했고, 설계를 굳이 다시 하고 예전의 나와 의견이 같다는 점을 뒤늦게 확인했으며, 

block.ts의 초안을 작성했다. 

```typescript
import { Client } from '@notionhq/client';
import { BlockObjectResponse } from '@notionhq/client/build/src/api-endpoints';
import { GetBlockResponse } from '@notionhq/client/build/src/api-endpoints';

export class Block {
    private notion: Client;
    private blockId: string;
    private blockData?: GetBlockResponse;

    constructor(notion: Client, blockId: string) {
        this.notion = notion;
        this.blockId = blockId;
    }

    public async getMarkdown(): Promise<string> {
        this.blockData = await this.fetchBlockData();
        return await this.processBlock(this.blockData);
    }

    private async fetchBlockData(): Promise<GetBlockResponse> {
        return await this.notion.blocks.retrieve({ block_id: this.blockId });
    }

    private async processBlock(block: BlockObjectResponse): Promise<string> {
        let markdown = '';

        switch (block.type) {
            case 'paragraph':
                markdown += this.convertParagraph(block.paragraph);
                break;
            case 'heading_1':
                markdown += `# ${block.heading_1.rich_text
                    .map((t) => t.plain_text)
                    .join('')}\n\n`;
                break;
            // 다른 블록 유형에 대한 처리를 여기에 추가...
            default:
                console.warn(`Unsupported block type: ${block.type}`);
        }

        // 하위 블록 처리 (재귀적)
        if (block.has_children) {
            markdown += await this.processChildBlocks(block.id);
        }

        return markdown;
    }

    private async processChildBlocks(blockId: string): Promise<string> {
        const children = await this.notion.blocks.children.list({
            block_id: blockId,
        });
        let markdown = '';
        for (const child of children.results) {
            markdown += await this.processBlock(child as BlockObjectResponse);
        }
        return markdown;
    }

    private convertParagraph(paragraph: any): string {
        console.log('paragraph:' + paragraph);
        return paragraph.rich_text;
        // return paragraph.text.map((t) => t.plain_text).join('') + '\n\n';
    }

    // 기타 블록 유형에 대한 변환 함수를 여기에 추가...
}
```

완전히 지극히 초안 그 자체이다. 기초적인 형태만 잡았다. 아직 리턴값에 대한 이해가 충분하지 않아, 해당 부분에 대한 조정이 필요하다. (심지어 오류가 나는 상태이다)

블록의 타입이 모두 정의가 되어있으므로, 해당 타입에 따른 각 클래스를 따로 생성할 필요도 느낀다. 모두 여기에 집중된다면 코드가 너무 길어질 것 같다. 

## 결론

결국 오늘은 한 일이 별로 없다.. 복귀에 의의를 두자. 

