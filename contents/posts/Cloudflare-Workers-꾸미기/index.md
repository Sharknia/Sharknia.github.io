---
IDX: "NUM-270"
tags:
  - Hobby
  - BackEnd
  - Typescript
  - Cloudflare
  - Serverless
  - Supabase
description: "Oracle Cloud에서 Cloudflare Workers로 서버를 이전한 이유"
update: "2025-07-06T15:42:00.000Z"
date: "2025-07-07"
상태: "Ready"
title: "Cloudflare Workers 꾸미기"
---
![](image1.png)
## 서론

개인 프로젝트를 위해 오라클 클라우드(Oracle Cloud Infrastructure, OCI)의 무료 티어 서버에 FastAPI 기반의 백엔드 서버를 구축하여 사용하고 있었습니다. 그런데 어느 날 갑자기 HTTPS 통신이 완전히 막히는 문제가 발생했습니다. HTTP 연결은 정상적으로 이루어졌지만, 어떤 포트를 사용하든 HTTPS를 통한 연결은 실패했습니다.

정확한 원인은 알 수 없지만, 오라클 클라우드 무료 티어가 일부 사용자들에 의해 불법적인 용도로 악용되는 사례가 있어, 이를 방지하기 위해 특정 IP 대역이나 계정의 아웃바운드 HTTPS 트래픽을 주기적으로 차단하는 것이 아닌가 하는 추측을 하게 되었습니다. 실제 외국 포럼에서도 비슷한 글들을 몇 개 찾았습니다. 해결방법은 운 좋게 다시 며칠 후 차단이 풀리거나, 인스턴스를 새로 만드는 방법을 추천하고 있었습니다. 

무료티어는 지원 문의조차 불가능해 원인 파악이 힘든 상황이었습니다.

그럼에도 포기하지 않고 문제를 해결하기 위해 신뢰도 높은 서비스에서 트래픽을 중계하면 상황이 나아질까 싶어 Cloudflare의 프록시 기능을 이용해 보았습니다. 네임서버를 Cloudflare로 변경하고 트래픽을 우회시켰지만, 오라클 클라우드 단에서의 차단이 워낙 확실했는지 이 방법 역시 통하지 않았습니다. 가뜩이나 서버 속도도 만족스럽지 않던 차에 이런 문제까지 겪게 되니, 다른 대안을 찾아야겠다는 결심이 섰습니다.

## 대안 탐색: Cloudflare Workers와의 만남

새로운 서버 환경을 찾기로 하고, 그 전에 네임서버로 사용하고 있던 Cloudflare의 다양한 기능들을 둘러보게 되었습니다. '이렇게까지 무료로 제공한다고?' 싶은 생각이 들게 한 여러 서비스가 많았는데, 그러다가 Cloudflare Workers를 발견했습니다.

## Cloudflare Workers란?

Cloudflare Workers는 V8 Isolates 기술을 기반으로 하는 서버리스(Serverless) 실행 환경입니다. 사용자가 작성한 코드를 전 세계에 분산된 Cloudflare의 엣지 로케이션에서 실행시켜, 사용자에게서 가장 가까운 데이터 센터에서 요청을 처리하게 해줍니다. 이 덕분에 물리적 서버 관리 없이 코드를 배포하고 실행할 수 있으며, 응답 속도를 크게 단축할 수 있는 장점이 있습니다.

제가 Workers를 선택하게 된 이유는 다음과 같습니다.

1. 합리적인 과금 체계: 대부분의 서버리스 플랫폼이 실행 시간 전체를 기준으로 요금을 부과하는 반면, Workers는 순수 CPU 실행 시간만을 기준으로 과금합니다. 예를 들어 요청이 들어왔을 때 중간 다른 API 등을 호출해 대기 하는 시간은 실행 시간에서 제외됩니다. 따라서 네트워크 요청이 잦은 애플리케이션의 경우 비용 효율성이 매우 높을 것으로 기대됐습니다. 또한, 제가 만들고자 하는 기능은 단순한 CRUD API였기 때문에 무료 티어에서 제공하는 요청당 10ms의 CPU 실행 시간 제한 안에서 충분히 운영 가능하다고 생각했습니다.

1. 프레임워크와 유사한 개발 경험: AWS Lambda와 같은 함수 기반 서버리스는 여러 함수가 공통 로직을 공유할 때 배포가 복잡해지는 경향이 있습니다. 함수 기반이어서 개별 배포가 가능하지만, 코드의 재사용성을 위해 공용 코드를 늘리다보면 결국 코드가 덩어리가 되어 함수 기반의 장점을 잃어버리는 문제를 겪고, 편하지만은 않은 테스트, 배포에 고통 받은 경험이 있습니다. 하지만 Workers는 Hono나 Itty-Router 같은 프레임워크를 사용하면 마치 Express.js나 FastAPI처럼 라우팅 기반의 백엔드 애플리케이션을 손쉽게 구성할 수 있다는 점이 아주 매력적이었습니다.

1. Cloudflare 생태계와의 연동성: Workers KV(Key-Value 저장소), D1(SQLite 기반 관계형 데이터베이스), R2(오브젝트 스토리지) 등 Cloudflare가 제공하는 다른 서비스들과의 통합이 매우 용이하여 별도의 인프라 구축 없이도 강력한 백엔드를 만들 수 있습니다.

## 프레임워크 선택: 왜 Hono인가?

Cloudflare Workers 생태계에서 가장 주목받는 프레임워크는 단연 Hono였습니다. Hono는 엣지 컴퓨팅 환경을 위해 설계된 작고 빠른 웹 프레임워크로, TypeScript를 기반으로 합니다.

과거 Supabase의 Edge Function을 다루며 Deno와 TypeScript를 경험해 본 적이 있었고, Hono의 문법이 Express.js와 유사하여 학습 곡선이 매우 낮을 것이라 판단했습니다. 또 예제 코드를 봤을 때 FastAPI와도 비슷하게 현대적인 아키텍쳐 적용도 쉬울 것 같아 망설이지 않고 선택했습니다. 

## Hono로 API 이전하기

본격적으로 기존 FastAPI 코드를 Hono로 이전하는 작업을 시작했습니다.

### 1. Hono 프로젝트 생성 및 개발 서버 실행

Hono는 CLI를 통해 간단하게 프로젝트를 시작할 수 있습니다. (참조: [가이드](https://hono.dev/docs/getting-started/cloudflare-workers))

```bash
# Hono 프로젝트 생성
npm create hono@latest my-worker-app
```

이렇게 프로젝트를 생성하면, 

```bash
● Hello World example
○ Framework Starter
○ Application Starter
○ Template from a GitHub repo
```

이렇게 고를 수 있는 창이 나오는데, Framework Starter를 고르면 적절하게 백엔드 프레임워크의 모양을 갖춘 채 시작할 수 있습니다. 

```bash
# 생성된 디렉토리로 이동
cd my-worker-app

# 개발 서버 실행
npm run dev
```

`npm run dev` 명령을 실행하면 로컬 환경에서 즉시 개발 서버가 실행되며, 코드 변경 시 자동으로 재시작되어 편리하게 개발을 진행할 수 있습니다.

### 2. FastAPI에서 Hono로 API 코드 이식

기존 FastAPI 서버의 Swagger 문서를 참고하여 API 엔드포인트를 하나씩 Hono 코드로 옮겼습니다. 

예를 들어, 특정 ID를 가진 아이템을 조회하는 FastAPI 코드는 다음과 같습니다.

```python
FastAPI (Python)**
```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/api/items/{item_id}")
def get_item(item_id: int, q: str | None = None):
    # 실제로는 데이터베이스에서 데이터를 조회하는 로직이 들어갑니다.
    return {"item_id": item_id, "name": f"Item {item_id}", "query": q}
```

이 코드를 Hono로 변환하면 아래와 같이 작성할 수 있습니다.

```typescript
import { Hono } from 'hono'

const app = new Hono()

app.get('/api/items/:id', (c) => {
  const id = c.req.param('id')
  const q = c.req.query('q')

  return c.json({
    item_id: parseInt(id),
    name: `Item ${id}`,
    query: q
  })
})

export default app

```

라우팅 경로에서 파라미터를 받고(`:id`), 쿼리 스트링을 추출하는(`c.req.query()`) 방식이 직관적이어서 마이그레이션은 수월하게 진행되었습니다.

DB 연결은 Supabase Client를 사용했습니다.

### 3. 환경 변수 관리

API 키와 같이 민감한 정보는 Cloudflare의 시크릿 스토어를 통해 안전하게 관리했습니다. Cloudflare의 시크릿 스토어에 필요한 환경 변수를 등록한 후, worker-configuration.d.ts 파일에 다음의 내용을 추가합니다. 

```typescript
declare namespace Cloudflare {
    interface Env {
        ASSETS: Fetcher;
        SUPABASE_URL: { get(): Promise<string> };
        SUPABASE_ANON_KEY: { get(): Promise<string> };
        SUPABASE_SERVICE_ROLE_KEY: { get(): Promise<string> };
        JWT_SECRET: { get(): Promise<string> };
    }
}
interface CloudflareBindings extends Cloudflare.Env {}
```

그리고 `wrangler.jsonc` 파일에 시크릿 스토어 바인딩을 추가합니다. 

```json
{
    "$schema": "node_modules/wrangler/config-schema.json",
    "name": "project-name",
    "main": "src/index.ts",
    "secrets_store_secrets": [
        {
            "binding": "SUPABASE_URL",
            "store_id": "abc",
            "secret_name": "SUPABASE_URL"
        },
        {
            "binding": "SUPABASE_ANON_KEY",
            "store_id": "abc",
            "secret_name": "SUPABASE_ANON_KEY"
        },
        {
            "binding": "SUPABASE_SERVICE_ROLE_KEY",
            "store_id": "abc",
            "secret_name": "SUPABASE_SERVICE_ROLE_KEY"
        },
        {
            "binding": "JWT_SECRET",
            "store_id": "abc",
            "secret_name": "JWT_SECRET"
        }
    ],
    ...
```

## 배포

이제 클라우드 플레어에어 워커를 생성합니다. 방금 생성한 깃 레포지토리를 기반으로 만든다면, 자동으로 main에 푸쉬될 때마다 배포가 됩니다. 

## 결론: 만족스러운, 그러나 약간의 아쉬움

배포 후 API를 테스트해 본 결과, 로컬 환경에서는 매우 빨랐던 응답 속도가 실제 환경에서는 간단한 API임에도 1초 이상 소요되었습니다. 확인해 보니 제 요청이 /cdn-cgi/trace 주소로 확인 결과 미국 로스앤젤레스(LA) 리전으로 라우팅되고 있었는데, 물리적 거리로 인한 지연 시간이 발생한 것으로 보입니다. 

로컬에서는 전혀 속도 문제가 없던 것으로 보아, Supabase의 문제는 아닌 것이 확실해 보입니다.

성능 면에서는 아쉬움이 남지만, 일단 가장 큰 문제였던 HTTPS 통신이 정상화되었고, 더 이상 인프라 관리에 신경 쓰지 않고 코드에만 집중할 수 있게 되었다는 점에서 매우 만족합니다. 또, 배포와 관련한 UI등이 정말 깔끔하게 잘 만들어져있습니다. 개인 프로젝트나 간단한 API 서버를 위한 백엔드를 고민하고 있다면 Cloudflare Workers와 Hono는 충분히 고려해 볼 만한 강력한 조합이라고 생각합니다.

다만, 저는 역시 속도 문제와 더불어 계획하고 있던 프론트 리팩토링 작업을 고려해 vercel.app에 next.js로 하나의 프로젝트로 배포하는 방향으로 나중에 추후 다시 서버를 옮길 것 같습니다. 

