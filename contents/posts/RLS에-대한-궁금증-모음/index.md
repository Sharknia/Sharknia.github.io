---
IDX: "NUM-167"
tags:
  - RLS
  - Postgresql
description: "작업하면서 생긴 RLS에 대한 QnA"
update: "2024-03-22T01:44:00.000Z"
date: "2024-03-21"
상태: "Ready"
title: "RLS에 대한 궁금증 모음"
---
## 1. 조인

postgresql RLS에서 A테이블은 모두가 읽을 수 있고, B테이블은 나만 읽을 수 있다고 합시다. 
그럼 A와 B를 조인해서 만들어진 뷰는 어떻게 될까요? 내가 아닌 유저가 이 뷰를 읽으려고 시도하면 어떻게 될까요? 

### 정답

이 사용자는 A 테이블에는 접근할 수 있지만, B 테이블에 대한 접근 권한이 없습니다. 따라서, RLS 정책에 의해 B 테이블에서의 데이터 접근이 제한됩니다. 결과적으로, 조인된 뷰를 조회하려고 할 때 두 가지 가능한 결과가 있습니다:

- 접근 거부: B 테이블에 접근할 수 없기 때문에, RLS 정책이 매우 엄격하게 설정되어 있어 전체 뷰에 대한 접근이 거부될 수 있습니다.

- 부분적 결과: RLS 정책이나 쿼리 구조에 따라, B 테이블과 관련 없는 데이터(즉, A 테이블에서만 오는 데이터)만 조회할 수 있게 될 수도 있습니다. 이는 RLS 정책 구현 방법과 쿼리의 구체적인 세부 사항에 따라 달라집니다.

### 해설

#### RLS와 뷰의 작동 원리

PostgreSQL에서 RLS는 테이블 레벨에서 설정되며, 해당 테이블에 대한 쿼리가 실행될 때마다 RLS 정책이 적용됩니다. 이 정책은 `SELECT`, `INSERT`, `UPDATE`, `DELETE` 명령에 대해 설정할 수 있으며, 사용자가 명령을 실행할 때마다 정책에 정의된 조건에 따라 접근이 허용되거나 거부됩니다.

뷰는 기본적으로 하나 이상의 테이블에서 데이터를 검색하기 위한 쿼리를 저장한 객체입니다. 뷰 자체에는 데이터가 저장되어 있지 않으며, 뷰를 쿼리할 때마다 저장된 쿼리가 실행되어 결과가 반환됩니다.

#### 조인된 뷰의 접근 제어

A와 B를 조인해서 만들어진 뷰에 대한 접근 권한은, 뷰를 구성하는 기본 테이블의 RLS 정책과 해당 사용자의 권한에 따라 달라집니다. 즉, 뷰를 통해 데이터에 접근하려 할 때, 뷰에 포함된 각 테이블에 대한 접근 권한이 평가되고, 모든 관련 RLS 정책이 적용됩니다.

## 2. 아무 권한 설정도 없다면? 

PostgreSQL의 행 레벨 보안(Row-Level Security, RLS)가 enable 되어있는데, 예를 들어 INSERT에 대해서 아무 policy가 정의되어 있지 않은데 내가 이 테이블에 Insert를 시도한다면 어떻게 될까요?

### 정답

PostgreSQL에서 행 레벨 보안(Row-Level Security, RLS)이 활성화되어 있고, 특정 작업(예를 들어 INSERT)에 대한 정책이 정의되어 있지 않다면, 그 작업에 대한 기본 정책은 "DENY"가 됩니다. 즉, 아무런 INSERT 정책이 설정되어 있지 않으면, 해당 테이블에 대한 INSERT 시도는 기본적으로 거부됩니다.

## 3. 지금까지 설정한 RLS를 조회하는 방법은? 

다음의 쿼리문으로 여태 설정한 RLS들을 확인할 수 있습니다. 

```sql
SELECT
  pol.polname AS policy_name,
  nsp.nspname AS schema_name,
  tab.relname AS table_name,
  pol.polcmd AS command,
  pg_get_expr(pol.polqual, pol.polrelid) AS using,
  pg_get_expr(pol.polwithcheck, pol.polrelid) AS with_check
FROM
  pg_policy pol
  JOIN pg_class tab ON pol.polrelid = tab.oid
  JOIN pg_namespace nsp ON tab.relnamespace = nsp.oid
WHERE
  tab.relname = '테이블 이름';
```



