---
IDX: "NUM-163"
tags:
  - Postgresql
  - Supabase
description: "postgresql의 pg_jsonschema를 활용한 JSONB에 대한 유효성 검증"
update: "2024-03-18T09:48:00.000Z"
date: "2024-03-18"
상태: "Ready"
title: "postgresql의 pg_jsonschema"
---
![](image1.png)
## JSON 데이터와 스키마 유효성 검사의 필요성

데이터베이스에서 데이터 무결성을 보장하기 위해서는 데이터에 대한 제약조건을 정의하고 검사하는 것이 필수적입니다. 관계형 데이터베이스에서는 테이블 스키마를 정의하여 열 데이터 타입, NOT NULL 제약조건 등을 설정할 수 있습니다. 하지만 JSON 데이터 타입의 경우, 스키마가 존재하지 않기 때문에 유연성은 높지만 데이터 무결성 보장이 어렵습니다.

따라서 JSON 데이터에 대해서도 스키마를 정의하고 유효성 검사를 수행할 수 있다면, 데이터 무결성을 높이고 애플리케이션의 안정성을 향상시킬 수 있습니다. 여기에서 JSON 스키마와 pg\_jsonschema 확장 기능이 등장합니다.

## JSON 스키마란?

JSON 스키마는 JSON 데이터의 구조와 형식을 정의하는 스키마입니다. 키-값 쌍의 데이터 타입, 필수 필드, 허용되는 값의 범위 등을 지정할 수 있습니다. JSON 스키마는 JSON 데이터 포맷으로 작성되며, 다양한 규칙과 제약조건을 정의할 수 있습니다.

예를 들어, 다음과 같은 JSON 스키마를 정의할 수 있습니다:

```json
{
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "age": { "type": "number", "minimum": 0 }
  },
  "required": ["name"]
}

```

이 스키마는 JSON 객체여야 하며, name 필드는 문자열이어야 하고 필수 필드이며, age 필드는 0 이상의 숫자여야 함을 정의합니다.

## pg\_jsonschema 소개

pg\_jsonschema는 PostgreSQL에서 JSON 데이터에 대한 스키마 유효성 검사를 지원하는 확장 기능입니다. 이 확장 기능을 사용하면 JSON 스키마를 정의하고, 해당 스키마에 따라 JSON 데이터의 유효성을 검사할 수 있습니다. pg\_jsonschema는 오픈 소스 프로젝트로, Supabase에서 개발 및 유지보수하고 있습니다.

## pg\_jsonschema의 설치

### Supabase의 경우

만약 supabase를 사용하고 있다면, 다음의 명령어만으로 간단하게 확장을 활성화 할 수 있습니다.

```sql
CREATE EXTENSION pg_jsonschema;
```

만약 supabase가 아니라면 설치가 필요합니다. 

### Supabase가 아닌 경우

1. 사전 요구사항

    - PostgreSQL 12 이상 버전

    - PostgreSQL 데이터베이스 관리자 권한

1. 소스 코드를 다운로드 합니다. pg\_jsonschema 소스 코드는 GitHub 저장소에서 다운로드할 수 있습니다.

    ```shell
    git clone https://github.com/supabase/pg_jsonschema.git
    ```

1. 빌드 및 설치 소스 코드 디렉터리로 이동한 후, 다음 명령어를 실행하여 확장 기능을 빌드하고 설치합니다.이 명령은 pg\_jsonschema 확장 기능을 PostgreSQL 데이터베이스에 설치합니다.

    ```shell
    cd pg_jsonschema
    make install
    ```

1. 확장 기능 활성화 PostgreSQL 데이터베이스에 접속한 후, 다음 SQL 문을 실행하여 pg\_jsonschema 확장 기능을 활성화합니다.

    ```sql
    CREATE EXTENSION pg_jsonschema;
    ```

    성공적으로 활성화되면 다음과 같은 메시지가 표시됩니다.

    ```plain text
    CREATE EXTENSION
    ```

이제 pg\_jsonschema 확장 기능을 사용할 수 있게 되었습니다.

## pg\_jsonschema 기능 및 활용

pg\_jsonschema는 다음과 같은 두 가지 주요 함수를 제공합니다:

- `json_matches_schema(schema json, instance json)`: JSON 데이터와 JSON 스키마를 입력받아, 해당 데이터가 스키마를 만족하는지 여부를 반환합니다.

- `jsonb_matches_schema(schema json, instance jsonb)`: JSONB 데이터와 JSON 스키마를 입력받아, 해당 데이터가 스키마를 만족하는지 여부를 반환합니다.

이 함수들을 사용하여 PostgreSQL 테이블의 CHECK 제약조건을 정의할 수 있습니다. 예를 들어, 다음과 같이 제약조건을 추가할 수 있습니다:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    data JSONB NOT NULL,
    CHECK (jsonb_matches_schema(
        schema => '{
            "type": "object",
            "properties": {
                "name": { "type": "string" },
                "age": { "type": "number", "minimum": 0 }
            },
            "required": ["name"]
        }'::json,
        instance => data
    ))
);

```

이렇게 하면 users 테이블의 data 열에 저장되는 JSONB 데이터가 반드시 지정된 스키마를 만족해야 합니다. 스키마를 위반하는 데이터를 INSERT 또는 UPDATE하려고 하면 오류가 발생합니다.

## pg\_jsonschema를 활용한 실제 예제

실제 예제를 통해 pg\_jsonschema를 어떻게 활용할 수 있는지 살펴보겠습니다. 여기서는 게시물 데이터를 저장하는 posts 테이블을 예로 들겠습니다.

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    author_id UUID NOT NULL,
    content JSONB NOT NULL,
    CHECK (jsonb_matches_schema(
        schema => '{
            "type": "object",
            "properties": {
                "title": { "type": "string" },
                "body": { "type": "string" },
                "tags": {
                    "type": "array",
                    "items": { "type": "string" },
                    "minItems": 1
                }
            },
            "required": ["title", "body", "tags"]
        }'::json,
        instance => content
    ))
);

```

이 테이블에서는 content 열에 JSONB 데이터가 저장되며, 이 데이터는 반드시 title(문자열), body(문자열), tags(문자열 배열, 최소 1개 이상의 항목) 필드를 포함해야 합니다.

다음과 같이 유효한 데이터를 INSERT할 수 있습니다:

```sql
INSERT INTO posts (author_id, content)
VALUES (
    '8c189b74-9865-4e0b-9bfd-a1a6a3e7fcc0',
    '{ "title": "Hello, World!", "body": "This is my first post.", "tags": ["hello", "world"] }'::jsonb
);

```

하지만 다음과 같이 스키마를 위반하는 데이터를 INSERT하려고 하면 오류가 발생합니다:

```sql
INSERT INTO posts (author_id, content)
VALUES (
    '8c189b74-9865-4e0b-9bfd-a1a6a3e7fcc0',
    '{ "title": 123, "body": "Invalid data" }'::jsonb
);

-- ERROR:  23514: check constraint "posts_content_check" of relation "posts" is violated by some row

```

이처럼 pg\_jsonschema를 활용하면 PostgreSQL 데이터베이스 내에서 JSON 데이터에 대한 스키마 유효성 검사를 수행할 수 있습니다. 이를 통해 데이터 무결성을 높이고, 잘못된 데이터로 인한 애플리케이션 오류를 방지할 수 있습니다.

## 마무리

이 글에서는 PostgreSQL 확장 기능인 pg\_jsonschema에 대해 알아보았습니다. JSON 데이터의 스키마 유효성 검사 필요성과 JSON 스키마의 개념, pg\_jsonschema의 기능과 활용 방법, 실제 예제를 통해 실습해보는 과정을 다루었습니다.

pg\_jsonschema를 사용하면 유연한 JSON 데이터의 장점을 그대로 활용하면서도, 데이터 무결성을 보장할 수 있습니다. 이를 통해 데이터베이스 애플리케이션의 안정성과 신뢰성을 높일 수 있습니다.

