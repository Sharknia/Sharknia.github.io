---
title: "github.io 자동배포"
description: "github.io 자동배포"
date: 2023-08-30
update: 2023-08-31
tags:
  - Blogging
series: "GitHub Pages와 Notion API 연동"
---
# github.io 자동배포

### 지난시간

[github.io를 이용한 블로그](https://sharknia.vercel.app/github.io를-이용한-블로그)

### 왜 하는가

나는 귀찮은게 너무 귀찮다. 

커밋 한 이후에 

```bash
npm run deploy-gh
```

를 해야 배포가 되는 것도 너무 귀찮았다. 

### 무엇을 하는가

그래서, 이걸 `github action` 을 이용해서 자동화를 했다. 

`.gitbub/workflows`에 임의의 yml 파일을 넣어주면 해당 작업을 github action에서 진행한다. 

### 어떻게 했는가

yml 파일 내용은 다음과 같다. 

```bash
name: Deploy

on: # 어떤 작업이 수행될 때 deploy.yml 작업이 수행된다. (트리거)
  push: # push 작업이 수행될 때
    branches: # 특정 브랜치를 대상으로
      - master

permissions: # github action이 수행되는 환경에서 특정 권한을 준다
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: 20.3.1

      - name: Install node packages
        run: yarn
        
      - name: Check lint
        run: yarn check:lint
        
      # 아래와 같은 오류가 남
      # error Command failed with exit code 1.
      # info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
      # - name: Check prettier
      #   run: yarn check:prettier
      
      - name: Build
        run: yarn build
        
      - name: Set up GitHub token
        env:
          GITHUB_TOKEN: ${{ secrets.Token}}
        run: git config --global user.email "email입력" && git config --global user.name "name입력"

      - name: Set Git remote URL with token
        run: git remote set-url origin https://${{ secrets.classicToken }}@github.com/Sharknia/Sharknia.github.io

      - name: Deploy to GitHub Pages
        run: npx gh-pages -d public
```

### 문제는 없었는가

처음에는 npm install을 사용했었는데, 패키지가 없다던지 여러가지 문제가 났다. 해당 문제는 npx 명령을 사용하는 것으로 해결돼

Deploy 까지 자동으로 하기 위해서는 깃허브 설정 관련 명령어가 필요했다. 

특히, 반드시 토큰을 발행하고 Github Action의 환경 변수에 추가를 해주어야 한다.