---
IDX: "NUM-33"
tags:
  - NestJS
update: "2024-02-02T16:32:00.000Z"
date: "2023-08-20"
상태: "Ready"
title: "NestJS의 디렉토리 구조"
---
NestJS는 MVC 패턴을 지원하지만, 전통적인 Express.js 스타일의 MVC 구조와는 약간 차이가 있다. 

NestJS의 기본적인 디렉토리 구조는 다음과 같다. 

```lua
src/
|-- app.module.ts         // 앱의 주 모듈
|-- main.ts               // 앱의 진입점
|-- controllers/          // 컨트롤러 폴더
|   |-- app.controller.ts
|-- services/             // 서비스 폴더
|   |-- app.service.ts
```

NestJS에서는 각 기능별로 (예: 사용자, 포스트, 댓글 등) 모듈을 분리하는 것을 권장한다. 따라서 각 기능별로 모듈을 생성하고, 해당 모듈 내에서 MVC 구조를 구성하는 것이 더 일반적이다.

```lua
src/
|-- app.module.ts
|-- main.ts
|-- user/
|   |-- user.module.ts
|   |-- user.controller.ts
|   |-- user.service.ts
|   |-- user.model.ts (또는 user.entity.ts)
|-- post/
|   |-- post.module.ts
|   |-- post.controller.ts
|   |-- post.service.ts
|   |-- post.model.ts (또는 post.entity.ts)
```

#### MVC 패턴과의 비교

- NestJS의 장점:

    1. 모듈화: NestJS는 기능별로 코드를 모듈로 분리하고 관리하도록 설계되었다. 이로 인해 각 기능이 독립적으로 개발되고 테스트될 수 있으며, 코드 재사용성도 향상된다.

    1. 의존성 주입: NestJS는 내장된 의존성 주입 컨테이너를 제공하므로, 코드의 결합도를 낮추고 유닛 테스트를 쉽게 할 수 있다.

    1. 타입 안전성: TypeScript를 기본 언어로 사용하기 때문에, 타입 검사와 관련된 오류를 컴파일 시점에 발견할 수 있다.

    1. 데코레이터 기반: 데코레이터를 사용하여 메타데이터를 기반으로 하는 다양한 기능을 쉽게 추가할 수 있다.

    1. 플랫폼 독립성: NestJS는 기본적으로 Express.js를 사용하지만, Fastify와 같은 다른 HTTP 서버 라이브러리로 쉽게 전환할 수 있다.

- NestJS의 단점:

    1. 학습 곡선: NestJS에는 다양한 개념과 기능이 있으므로 초기에 학습하기가 다소 어려울 수 있다.

    1. 추가적인 추상화: 모듈화와 의존성 주입과 같은 기능들은 유용하지만, 간단한 애플리케이션에서는 오버헤드로 느껴질 수 있다.

- MVC 패턴의 장점:

    1. 간단함: 전통적인 MVC 패턴은 많은 웹 개발자들에게 잘 알려져 있으므로 학습 곡선이 낮다.

    1. 직관성: 모델, 뷰, 컨트롤러의 구조는 웹 애플리케이션의 데이터 흐름을 이해하기 쉽다.

- MVC 패턴의 단점:

    1. 확장성: 애플리케이션의 복잡도가 증가하면 MVC 구조만으로는 코드의 복잡성을 관리하기 어려울 수 있다.

    1. 타이트 커플링: 모델, 뷰, 컨트롤러 간에 타이트 커플링이 발생할 수 있어, 변경 사항이 여러 컴포넌트에 영향을 줄 수 있다.

