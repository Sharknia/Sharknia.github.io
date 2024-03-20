---
IDX: "NUM-164"
tags:
  - Postgresql
description: "Postgresql에서 ENUM 타입에서 값 삭제하기"
update: "2024-03-20T08:24:00.000Z"
date: "2024-03-19"
상태: "Ready"
title: "Postgresql에서 ENUM 타입에서 값 삭제하기"
---
![](image1.png)
## 서론

PostgreSQL의 ENUM 타입은 유연성이 제한적이라는 단점이 있습니다. 일단 생성되면 ENUM 타입에 값을 직접 추가하거나 삭제할 수 없기 때문입니다. 하지만 간단한 우회 방법을 통해 ENUM 타입 값 삭제가 가능합니다.

이 포스트에서는 기존에 정의된 ENUM 타입에서 하나의 값만 삭제하는 방법에 대해 알아보겠습니다.

### 시나리오

기존에 정의된 `mood` ENUM 타입이 아래와 같이 있다고 가정해봅시다.

```sql
sqlCopy codeCREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');

```

여기서 'ok' 값을 삭제하고자 한다면 어떻게 해야할까요?

## 값 삭제 절차

1. **새 ENUM 타입 생성**: 삭제하려는 값을 제외한 새 ENUM 타입을 생성합니다.

    ```sql
    CREATE TYPE mood_new AS ENUM ('sad', 'happy');
    ```

1. **컬럼 타입 변경**: 기존 컬럼의 타입을 새로 생성한 ENUM 타입으로 변경합니다.

    ```sql
    ALTER TABLE your_table
    ALTER COLUMN your_column TYPE mood_new
    USING your_column::text::mood_new;
    ```

1. **기존 ENUM 타입 삭제**: 기존 ENUM 타입을 DROP 합니다.

    ```sql
    DROP TYPE mood;
    ```

1. **새 ENUM 이름 변경(옵션)**: 필요하다면 새 ENUM 타입의 이름을 기존 이름으로 변경할 수 있습니다.

    ```sql
    ALTER TYPE mood_new RENAME TO mood;
    ```

## 주의사항

- 삭제하려는 ENUM 값이 기존 데이터에 사용되고 있다면, 이 절차를 진행하기 전에 해당 데이터를 먼저 변경하거나 삭제해야 합니다.

- 이 작업은 ENUM 타입을 사용하는 모든 테이블과 뷰에 영향을 미칠 수 있으므로, 실행 전 데이터베이스 백업을 진행하는 것이 안전합니다.

