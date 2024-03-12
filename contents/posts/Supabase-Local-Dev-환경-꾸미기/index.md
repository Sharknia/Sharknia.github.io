---
IDX: "NUM-156"
tags:
  - Supabase
  - Postgresql
  - Edge-Function
description: "Edge Function 개발을 위한 Supabase Local Dev 환경 만들기"
update: "2024-03-12T02:00:00.000Z"
date: "2024-03-12"
상태: "Ready"
title: "Supabase Local Dev 환경 꾸미기"
---
## Supabase CLI 설치

우선 supabase CLI를 설치해야 합니다. 

```shell
brew install supabase/tap/supabase
```

제대로 설치가 되면 다음의 명령어를 통해 설치가 된 것을 확인합니다. 

```shell
supabase help
```

login을 진행합니다. 터미널의 다음의 명령어를 입력한 후, 로그인은 웹에서 이뤄집니다. 

```shell
supabase login
```

Project를 연결해야 합니다. 먼저 다음의 명령어를 통해 프로젝트 리스트를 확인합니다. 

```shell
supabase projects list
```

다음과 같은 결과가 출력됩니다. 

```shell
    LINKED │        ORG ID        │     REFERENCE ID     │   NAME   │         REGION         │  CREATED AT (UTC)
  ─────────┼──────────────────────┼──────────────────────┼──────────┼────────────────────────┼──────────────────────
           │ [ORG ID        ]     │ [REFERENCE ID     ]  │ [NAME]   │ Northeast Asia (Seoul) │ 2024-02-17 04:10:12
```

이 중에서 REFERENCE ID를 사용해 project에 연결할 수 있습니다. 

```shell
supabase link --project-ref [REFERENCE ID]
```

#### 참고

## Edge Function 로컬에서 개발하기

로컬에 supabase project를 생성합니다. 

```shell
supabase init
```

다음의 명령어를 사용하면 예제 Edge function을 만들 수 있습니다. 이 명령어는 함수를 로컬에 생성합니다. 

```shell
supabase functions new hello-world
```

다음과 같은 구조로 Edge Function이 생성됩니다. 

```shell
└── supabase
    ├── functions
    │   └── hello-world
    │   │   └── index.ts ## Your function code
    └── config.toml
```

### 예제 코드

```typescript
Deno.serve(async (req) => {
  const { name } = await req.json()
  const data = {
    message: `Hello ${name}!`,
  }

  return new Response(JSON.stringify(data), { headers: { 'Content-Type': 'application/json' } })
})
```

Edge Function은 native Deno.serve를 사용합니다. 

## Edge Function 로컬에서 실행하기

start 명령어를 사용하면, 도커를 이용해서 로컬에서 supabase service를 실행할 수 있습니다. 

```shell
supabase start
```

성공적으로 실행되면 다음과 같은 내용이 출력됩니다. 해당 내용은 `supabase status` 명령어를 사용해 다시 볼 수 있습니다.

```shell
Started supabase local development setup.

         API URL: http://localhost:54321
          DB URL: postgresql://postgres:postgres@localhost:54322/postgres
      Studio URL: http://localhost:54323
    Inbucket URL: http://localhost:54324
        anon key: eyJh......
service_role key: eyJh......
```

`supabase stop` 명령을 사용하여 로컬 데이터베이스를 재설정하지 않고 언제든지 모든 서비스를 중지할 수 있습니다. 또한 `supabase stop --no-backup`을 사용하여 모든 서비스를 중지하고 로컬 데이터베이스를 재설정할 수 있습니다.



다음의 명령어를 사용해 Function Watcher를 실행합니다. 

```shell
supabase functions serve
```

이 명령어는 hot-reloading 기능을 가지고 있어 코드에 수정이 생기면 Deno server를 자동으로 재시작합니다. 



여기까지 마친 후, 다음의 명령어를 통해 방금 만든 Edge Function을 실행해볼 수 있습니다. 

```shell
curl --request POST 'http://localhost:54321/functions/v1/hello-world' \
  --header 'Authorization: Bearer SUPABASE_ANON_KEY' \
  --header 'Content-Type: application/json' \
  --data '{ "name":"Functions" }'
```

SUPABASE\_ANON\_KEY는 start 이후 생성된 anon key를 사용하면 됩니다. 성공적으로 실행되면 다음과 같은 문구를 확인할 수 있습니다. 

```shell
{"message":"Hello Functions!"}
```

## Edge Function 배포하기

기본적인 배포는 다음의 명령어를 사용합니다. 모든 Edge Function을 배포합니다. 

```shell
supabase functions deploy
```

특정 Edge function만 배포할 수도 있습니다. 

```shell
supabase functions deploy hello-world
```

기본적으로 Edge Function은 JWT 토큰이 필수입니다. 토큰이 필요없는 Edge function을 만들기 위해서는 `--no-verify-jwt` 옵션을 사용합니다. 

```shell
supabase functions deploy hello-world --no-verify-jwt
```

배포가 성공적으로 이뤄지면 supabase 대쉬보드에서도 이를 확인할 수 있습니다. 



다음의 명령어를 사용해 방금 배포한 Edge Function을 테스트 해볼 수 있습니다. 

```shell
curl --request POST 'https://[REFERENCE ID].supabase.co/functions/v1/hello-world' \
  --header 'Authorization: Bearer ANON_KEY' \
  --header 'Content-Type: application/json' \
  --data '{ "name":"Functions" }'
```

`--no-verify-jwt` 옵션을 사용한 경우에는 ANON\_KEY가 필요 없으며, ANON\_KEY는 Supabase 대시보드에서 확인할 수 있습니다. 

## Edge Function 삭제

다음의 명령어를 통해 Edge Function을 삭제할 수 있습니다.

```shell
supabase functions delete hello-world --project-ref dcrdrguovpqchlptvlqy
```

단 이 명령어는 로컬에서는 작동하지 않습니다. 로컬의 함수는 수동으로 삭제하면 됩니다. 

## 기타 Edge Function 관련 CLI 명령어

function의 리스트를 확인합니다. 이 코드는 연결된 프로젝트의 함수 리스트를 가져옵니다. 

```shell
supabase functions list
```

특정 function을 다운로드 합니다. 

```shell
supabase functions download <Function name>
```

공통적으로 `--project-ref` **옵션을 사용해 특정 project를 명시할 수 있습니다.** 



## Deno 환경에서 supabase client 사용하기

Deno 환경에서는 npm을 사용하지 않으므로, supabase client가 필요할 경우 다음의 방법을 사용해 import 합니다. 

```typescript
  import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'
```

또는 import map을 사용할 수 있습니다. 

supabase/functions/import\_map.json 파일을 생성 후, 다음의 내용을 입력해줍니다. 

```json
{
    "imports": {
        "supabase": "https://esm.sh/@supabase/supabase-js@2"
    }
}
```

import map을 사용하면 다음과 같이 쉽게 import 할 수 있습니다. 

```typescript
import { createClient } from 'supabase';
```

## Deno 환경에서 환경변수 사용하기

다음의 코드를 사용해 환경변수를 사용할 수 있습니다. 

```typescript
Deno.env.get(MY_SECRET_NAME)
```

현재 설정되어있는 환경 변수들은 다음의 명령어를 통해 확인할 수 있습니다. 

```shell
supabase secrets list
```

기본적으로 supabase는 4개의 환경 변수를 설정하지 않아도 지원합니다. 

- SUPABASE\_URL

- SUPABASE\_ANON\_KEY

- SUPABASE\_SERVICE\_ROLE\_KEY

- SUPABASE\_DB\_URL

### 환경변수 추가하기 - 로컬

추가적으로 환경변수를 설정해야 할 때가 있습니다. 이 때 다음의 방법을 사용합니다. 

먼저 환경 변수를 생성합니다. 

```shell
echo "MY_NAME=Yoda" >> ./supabase/.env.local
```

그리고 로컬에서 serve 시에 다음의 옵션값을 함께 줍니다. 

```shell
supabase functions serve --env-file ./supabase/.env.local
```

이렇게 하면 로컬에서 MY\_NAME 환경 변수를 사용할 수 있습니다. 

### 환경변수 추가하기 - 실서버

우선 실서버에 적용할 환경변수 파일을 생성합니다. 

```shell
cp ./supabase/.env.local ./supabase/.env
```

그리고 다음의 명령어를 사용해 환경변수를 등록합니다. 

```shell
supabase secrets set --env-file ./supabase/.env

# You can also set secrets individually using:
supabase secrets set MY_NAME=Chewbacca
```

#### 참고

[https://supabase.com/docs/guides/functions/secrets](https://supabase.com/docs/guides/functions/secrets)

