---
IDX: "NUM-119"
tags:
  - Typescript
description: "타입스크립트에서의 열거형 정의"
update: "2024-02-02T16:33:00.000Z"
date: "2024-02-03"
상태: "Ready"
title: "Typescript의 열거형"
---
타입스크립트에서 `enum` 을 사용해 특정 값들의 집합을 정의하고 각 멤버에 문자열 값을 할당할 수 있다. 이는 열거형이라고 하며 관련된 상수 값들의 집합에 이름을 부여하여 코드의 가독성을 높이고 오류 가능성을 줄여준다. 

임의의 열거형을 정의하면 다음과 같다. 

```typescript
enum PageStatus {
    Deleted = "deleted",
    Writing = "writing"
}
```

이렇게 정의하면 PageStatus 타입의 변수를 사용할 때 PageStatus.Deleted 로 사용해서 열거형 멤버게 접근할 수 있다. 

이렇게 얻어진 값은 각각에 할당된 문자열과 같다. 

사용법 예시 코드는 다음과 같다. 

```typescript
let currentPageStatus: PageStatus = PageStatus.Writing;

if (currentPageStatus === PageStatus.Deleted) {
    console.log("The page is deleted.");
} else if (currentPageStatus === PageStatus.Writing) {
    console.log("The page is in writing status.");
}
```

<hr style="border: none; height: 1px; background-color: #e0e0e0; margin: 16px 0;" />
만약, 열거형을 정의할 때 값을 명시하지 않으면 열거형의 값은 기본적으로 0에서 시작해 순차적으로 증가한다.  예를 들어 다음과 같다. 

```typescript
enum PageStatus {
    Deleted, // 0
    Writing  // 1
}
```

물론 임의로 숫자를 지정할 수도 있다. 

```typescript
enum StatusCode {
    Success = 200,
    NotFound = 404,
    ServerError = 500
}
```

또한 숫자 열거형에서 일부 멤버에만 값을 지정하고 다른 멤버에는 값을 지정하지 않는다면 타입스크립트는 자동으로 이전 멤버의 값에서 1을 더해 계산한다. 

```typescript
enum Example {
    Start = 1,
    Middle, // 값이 2로 자동 할당됩니다.
    End // 값이 3으로 자동 할당됩니다.
}
```



