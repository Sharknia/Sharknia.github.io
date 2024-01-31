---
tags:
  - VSCode
description: "editor.quickSuggestions 설정에 대해 기록"
update: "2024-01-30"
date: "2024-01-31"
상태: "POST"
title: "vscode-quick Suggestions"
---
editor.quickSuggestions 설정은 VSCode에서 코드를 작성하는 동안 자동완성을 어떻게 표시할지를 결정한다. 

다음 세가지의 옵션이 있다. 

- string

    문자열 내에서 자동 완성 제안을 활성화/비활성화 한다. 

- comments

    주석 내에서 자동 완성 제안을 활성화/비활성화 한다. 

- other

    코드(문자열이나 주석이 아닌 부분) 내에서 자동 완성 제안을 활성화/비활성화 한다. 

VSCode의 설정을 json으로 열어 다음의 내용을 입력하면 된다. 

```json
"editor.quickSuggestions": {
    "strings": true,
    "comments": true,
    "other": true
}
```



