---
title: "github.io를 이용한 블로그"
description: "github.io를 이용한 블로그"
date: 2023-08-28
update: 2023-08-28
tags:
  - Blogging
series: "GitHub Pages와 Notion API 연동"
---
# github.io를 이용한 블로그

[🚀 2. 빠르게 시작하기](https://devhudi.github.io/gatsby-starter-hoodie/quick-start-kr/)

디자인이 마음에 들고, 필요로 하는 기능이 모두 들어가 있기 때문에 해당 테마를 선택했다. (시리즈 기능, 목차 기능, 댓글 기능)

5번까지는 무사히 테스트 했는데, 

6번 특히 Netlify를 활용한 배포에서 막혔다. 

Repository를 생성 후, 해당 저장소를 바로 Netlify에서 빌드하려고 하면 의존성 오류가 났다. 

Netlify를 활용하려고 한 이유는 Github Pages를 통해 배포할 경우 마스터 브랜치와 별도의 브랜치가 생성되는것이 번잡스러웠기 때문이다. 

조금 더 알아본 결과,

```bash
$ npm run build
```

를 실행하면, 

`/public` 에 빌드 결과물이 생성되고 해당 폴더만 배포하면 될 것 같긴하다. 또는 `github action` 을 이용할 수도 있겠다. 

조금 더 편하게 블로그를 하려는 길은 아직 멀고 험하다.