---
IDX: "NUM-169"
tags:
  - Deno
  - Typescript
description: "Deno에서 GET 요청으로 넘어온 쿼리 스트링 변수를 처리하기"
update: "2024-04-02T07:33:00.000Z"
date: "2024-04-02"
상태: "Ready"
title: " Deno에서 URL의 쿼리 스트링(Query String) 다루기"
---
## 정답

URL 객체를 활용하면 됩니다. 

## 자세히!

Deno는 웹 표준 API를 지원하기 때문에,  `URL` 객체와 `URLSearchParams` 객체를 제공합니다. 이를 활용하면 쿼리 스트링을 쉽게 다룰 수 있습니다. 먼저 `Request` 객체에서 URL을 가져온 후, `URL` 객체를 생성하고 이 객체의 `searchParams` 프로퍼티를 통해 쿼리 파라미터에 접근할 수 있습니다.아래는 GET 방식으로 넘어온 쿼리 스트링 변수를 어떻게 처리하는지에 대한 예시입니다.

```typescript
// Deno.serve() 안에 있는 async 함수 내부에 추가합니다.
// req 객체에서 URL을 가져온 후 URLSearchParams 객체를 사용하여 쿼리 파라미터에 접근합니다.

const url = new URL(req.url, `http://${req.headers.get("host")}`);
const queryParams = url.searchParams;

// 예를 들어, "user_id"라는 쿼리 파라미터 값을 얻고 싶다면
const userId = queryParams.get("user_id"); // "user_id" 쿼리 스트링의 값을 가져옵니다.

// 이제 userId 변수를 사용하여 필요한 로직을 구현할 수 있습니다.
```

## URLSearchParams?

`URLSearchParams` 객체는 URL의 쿼리 스트링을 편리하게 다룰 수 있는 다양한 메서드를 제공합니다.

- `get(name)`: 지정된 이름의 첫 번째 값을 반환합니다.

- `getAll(name)`: 지정된 이름의 모든 값을 배열로 반환합니다.

- `has(name)`: 지정된 이름의 쿼리 파라미터가 있는지 확인합니다.

- `set(name, value)`: 지정된 이름과 값으로 새 쿼리 파라미터를 설정합니다.

- `append(name, value)`: 지정된 이름과 값으로 새 쿼리 파라미터를 추가합니다.

- `delete(name)`: 지정된 이름의 쿼리 파라미터를 제거합니다.

예를 들어, `URLSearchParams` 객체를 활용하여 여러 개의 값을 가진 쿼리 파라미터를 처리할 수 있습니다.

```typescript
const url = new URL("https://example.com/?tags=javascript&tags=deno&tags=nodejs");
const queryParams = url.searchParams;
const tags = queryParams.getAll("tags");// ["javascript", "deno", "nodejs"]
```

