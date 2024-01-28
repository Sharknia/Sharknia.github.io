---
title: "NotionAPI를 활용한 자동 포스팅(5)"
description: "1차 목표 달성"
date: 2024-01-28
update: 2024-01-28
tags:
  - Blogging
  - Notion-API
  - Typescript
series: "GitHub Pages와 Notion API 연동"
---
## 열정

추가로 더 진행해버렸다. 정말로 여기까지만 하려고 한다. 오랜만에 탄력 받으니 계속 하게 되어버렸다. 

## 구현 내용

콜아웃, 디바이더, 인용문, 코드, 번호 매기기 , 글머리 기호 목록 컨버터를 추가했다. 

### 콜아웃, 디바이더

html로 구현했다. 인용문은 hr 태그로 처리했으며, 콜아웃은 div 태그를 사용했다. 다만 콜아웃은 스타일 처리가 필요하다. 지금은 대충 태그만 만들어놨다. 따라서 이건 미완성이나 다름없다. 

```typescript
private convertCallout(calloutBlock: any): string {
        const textContent = calloutBlock.rich_text
            .map((textElement: any) => this.formatTextElement(textElement))
            .join('');
        const icon = calloutBlock.icon ? calloutBlock.icon.emoji : '';
        const color = calloutBlock.color
            ? calloutBlock.color
            : 'gray_background';

        return `
    <div class="callout ${color}">
        ${icon} <span>${textContent}</span>
    </div>\n`;
    }

private convertDivider(): string {
        return `<hr style="border: none; height: 1px; background-color: #e0e0e0; margin: 16px 0;" />\n`;
    }
```

### 인용문

인용문은 있을 줄 몰랐는데, 마크다운에서 지원을 해서 쉽게 구현했다. 

무엇은 지원하고 무엇은 지원안하고, 기준을 명확히 모르겠긴하다. 

```typescript
private convertQuote(quoteBlock: any): string {
        const quoteText = quoteBlock.rich_text
            .map((textElement: any) => this.formatTextElement(textElement))
            .join('');

        return `> ${quoteText}\n\n`;
    }
```

### 번호 매기기, 글머리 기호 

둘은 구현이 상당히 비슷했다. 마치 세트와 같다. 이걸 구현할 때에, 들여쓰기를 하려면 계층 개념이 필요해서 전체적인 코드 수정이 한 번 이뤄졌다. 

```typescript
private formatListItemContent(listItemBlock: any): string {
        return listItemBlock.rich_text
            .map((textElement: any) => this.formatTextElement(textElement))
            .join('');
    }

    private convertNumberedList(listItemBlock: any): string {
        const listItemContent = this.formatListItemContent(listItemBlock);
        const indent = ' '.repeat(this.indentLevel * 2);
        return `${indent}1. ${listItemContent}\n`;
    }

    private convertBulletedList(listItemBlock: any): string {
        const listItemContent = this.formatListItemContent(listItemBlock);
        const indent = ' '.repeat(this.indentLevel * 2);
        return `${indent}- ${listItemContent}\n`;
    }
```

띄어쓰기 네 번으로 계층을 구분한다. 

### 코드 

코드도 쉬웠다. 마크다운에서 지원하는 것은 기본적으로 구현이 쉽다. 

```typescript
private convertCode(codeBlock: any): string {
        const codeText = codeBlock.rich_text
            .map((textElement: any) => textElement.plain_text)
            .join('');

        const language = codeBlock.language || '';

        return `\`\`\`${language}\n${codeText}\n\`\`\`\n\n`;
    }
```

### to_do

to_do도 간단하게 구현했다. 마크다운에서는 체크박스인데, 노션에서는 할 일 목록이다.. 그래서 체크박스를 체크하면 스타일이 추가되는데.. 이것까진 따로 구현하지 않았다. 

```typescript
private convertToDo(toDoBlock: any): string {
        const quoteText = toDoBlock.rich_text
            .map((textElement: any) => this.formatTextElement(textElement))
            .join('');
        let pre = '- [ ]';
        if (toDoBlock.checked == true) {
            pre = '- [x]';
        }
        return `${pre} ${quoteText}\n\n`;
    }
```

## 정리 

### 작업 완료 블록

일단 1차적으로 작업이 완료되었으며, 완료된 지원하는 블록 타입은 다음과 같다. 

- paragraph
- heading_1, heading_2, heading_3
- bookmark
- link_to_page
- image
- callout
- divider
- quote
- code
- numbered_list_item
- bulleted_list_item
- to_do
### 작업 예정 블록

#### table, table-row

꼭 필요해보이지만, 구현을 위해서는 현재 구현된 블록의 재귀 호출 구조를 바꿔야 한다. 상대적으로 까다로워 오늘 작업하진 않겠다. 

### 작업 미정 블록

#### column (n개의 열로 구성된 블록 생성)

가로로 여러 블록을 놓는 기능이다. 굳이 필요한가? 싶기도 하고 또 구현이 꽤나 까다로워 보인다. 다만 table, table-row를 작업할 때에 호출 구조를 잘 짜놓는다면 또 별 노력 없이 잘 될 것 같기도 한데, 마크다운에 원래 가로를 나누는 기능이 있는지는 또 모르겠다. 그래서 미정이다. 

#### 미디어 관련 블록

미디어 관련 기능도 지금보니 노션에 있다. 이건.. 할만하지 않을까? 고려해보겠다. 아직 자세히 살펴보지 못했다. 

### 미구현 확정 블록

#### 토글

토글은 미구현 확정이다. 토글은 마크다운에서 지원하지 않고, 만약 하려면 부트스트랩의 collapse 같은 기능을 직접 구현해야 할 것 같다. 이건 쉽지 않다. 

#### 임베드 관련 블록

기각이다. 블로그 글에 임베드는 쓰지 말자. 

#### 고급 블록

지금보니 토글도 고급에 들어있다. 마크다운 기본 기능이 아닌 것들은 기본적으로 구현에 한계가 있다. 다시 보니 column도 고급에 있다. 

나중에 추후 시간이 되면 만들만한 것들은 한 번 고려해보겠다. 

#### 데이터베이스 관련 블록

웹사이트에 넣을 수 있는 기능이 (아마도) 아니다. 기각이다. 

## 다음 작업 예정은? 

이제 배포 자동화를 목표로 해야 한다. 자동화까지 해둬야, 이 프로젝트를 만든 목적 달성이 아닐까? 얼른 자동화도 하고 싶다..

자동화의 자세한 로직은 다음번에 설계 하도록 하겠다.. 

