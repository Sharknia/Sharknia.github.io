---
IDX: "NUM-157"
tags:
  - Supabase
  - Postgresql
  - RLS
description: "Supabase의 RLS에 대해 알아보자"
update: "2024-03-12T03:02:00.000Z"
date: "2024-03-12"
상태: "Ready"
title: "Supabase와 Row-Level Security (RLS) "
---
![](image1.png)
## Row-Level Security (RLS) 란?

Row-Level Security (RLS)는 데이터베이스 시스템에서 제공하는 보안 기능 중 하나로, 데이터베이스의 각 행(row)에 대한 접근을 제어하는 세밀한 방법을 제공합니다. RLS를 사용하면 데이터베이스 사용자나 응용 프로그램이 특정 조건을 만족하는 행에만 접근하거나 그러한 행을 변경할 수 있도록 정책을 설정할 수 있습니다. 이는 사용자별 데이터 접근 제한, 다중 사용자 환경에서의 데이터 격리, 그리고 민감한 정보의 보호와 같은 목적을 위해 중요합니다.

### 특징

#### 세밀한 접근 제어

RLS는 사용자 또는 사용자 그룹별로 데이터베이스 행에 대한 접근 권한을 설정할 수 있게 해줍니다. 이를 통해 특정 데이터에 대한 접근을 정밀하게 제한할 수 있습니다.

#### 정책 기반 보안

RLS는 데이터베이스 레벨에서 정책(policy)을 정의하여 작동합니다. 이 정책들은 데이터에 대한 `SELECT`, `INSERT`, `UPDATE`, `DELETE`와 같은 작업 수행 시 적용되어, 해당 작업이 정책에 정의된 규칙을 충족시킬 때만 수행될 수 있습니다.

#### 투명성

RLS는 데이터베이스 시스템 내에서 자동으로 적용되므로, 응용 프로그램 코드를 변경하지 않고도 데이터 접근 제어를 강화할 수 있습니다. 사용자는 자신이 접근 권한을 가진 데이터만 볼 수 있으며, RLS 정책에 의해 접근이 제한된 데이터는 *존재하지 않는 것처럼* 처리됩니다.

## 예제 파헤쳐보기

먼저 RLS를 하나 설정해보고, 이 구문이 뭘 의미하는지 한 번 살펴보겠습니다. 

우선, 특정 테이블에 대해 RLS를 활성화 해줍니다. 

```sql
ALTER TABLE my_table ENABLE ROW LEVEL SECURITY;
```

특정 조건에 따라 데이터 접근을 제어하는 정책을 추가합니다. 예를 들어, 사용자가 자신의 데이터에만 접근할 수 있도록 하는 정책을 추가할 수 있습니다.

```sql
CREATE POLICY my_policy ON my_table FOR SELECT
USING (user_id = current_user_id());
```

이러한 방식으로 RLS를 구성함으로써, 데이터베이스 관리자는 데이터의 보안과 무결성을 유지하면서도 필요한 사용자에게 필요한 데이터에 대한 접근을 허용할 수 있습니다.

이는 모든 쿼리에 WHERE 절을 추가하는 것으로 생각하면 좀 더 쉽습니다. 예를 들어 위 정책의 경우, 

```sql
select * from my_table
where current_user_id = my_table.user_id;
-- Policy is implicitly added.
```

이렇게 정책이 where 절에 암묵적으로 추가된다고 생각하면 이해하기 쉽습니다. 

### Supabase에서의 RLS

좀 더 복잡하고, Supabase에 특화된 예제를 보면 다음과 같습니다. 

```python
create policy "Allow authenticated uploads"
on storage.objects
for insert
to authenticated
with check (
  (storage.foldername(name))[1] = auth.uid()::text
);
```

여기서 설정한 정책은 "Allow authenticated uploads"라는 이름으로, `storage.objects` 테이블에 대한 `insert` 작업에 적용됩니다. 이 정책은 다음과 같은 조건을 충족시키는 경우에만 삽입 작업을 허용합니다:



객체의 이름에서 파생된 폴더 이름 (`storage.foldername(name)` 함수를 통해 추출)의 첫 번째 요소가 현재 인증된 사용자의 ID (`auth.uid()`)와 일치해야 합니다. 이는 사용자가 자신의 고유 폴더에만 파일을 업로드할 수 있도록 제한하는 조건입니다.



요약하자면, 이 RLS 정책은 인증된 사용자가 'profile\_images' 버킷 내에 자신의 사용자 ID에 해당하는 폴더에만 파일을 업로드할 수 있게 합니다.

## Using과 Check

위의 두 예제를 살펴보면 각각 Using과 Check를 사용한 것을 볼 수 있습니다. 

PostgreSQL의 행 수준 보안(Row-Level Security, RLS) 정책을 사용할 때, `WITH CHECK`와 `USING` 절은 서로 다른 목적으로 사용됩니다. 이들의 주요 차이점은 정책이 적용되는 시점과 데이터 조작 작업(DML) 유형에 따라 달라집니다.

### `WITH CHECK`

- `WITH CHECK` 절은 `INSERT`와 `UPDATE` 작업에 대해 사용됩니다.

- 이 절은 새로운 데이터가 삽입되거나 기존 데이터가 업데이트될 때 적용되어야 하는 조건을 정의합니다.

- `WITH CHECK`은 해당 작업이 실행된 후 최종적으로 데이터가 정책에 정의된 조건을 만족하는지 검증합니다. 만약 조건을 만족하지 않는다면, 작업은 거부됩니다.

- 예를 들어, 사용자가 자신의 폴더에만 파일을 삽입하거나 업데이트할 수 있도록 제한하는 정책을 설정할 때 사용됩니다.

### `USING`

- `USING` 절은 `SELECT`와 `DELETE` 작업에 대해 사용됩니다.

- 이 절은 해당 작업을 수행할 때 접근이 허용되는 행을 결정하는 조건을 정의합니다.

- `SELECT` 작업에 대해, `USING` 조건을 만족하는 행만 조회될 수 있습니다.

- `DELETE` 작업에 대해, `USING` 조건을 만족하는 행만 삭제될 수 있습니다.

- 예를 들어, 사용자가 자신의 폴더에 있는 파일만 조회하거나 삭제할 수 있도록 제한하는 정책을 설정할 때 사용됩니다.

### 예시

결론적으로, `WITH CHECK`와 `USING`의 차이는 적용되는 데이터 조작 작업의 유형과 해당 조건이 검증되는 시점에 있습니다.

## Roles

### to authenticated

`to authenticated` 구문은 해당 정책이 인증된 사용자, 즉 로그인한 사용자에게만 적용됨을 의미합니다. Supabase 및 PostgreSQL의 Row-Level Security (RLS) 정책에서는, 특정 데이터베이스 작업(예: 삽입, 조회, 수정, 삭제)에 대한 접근 권한을 세밀하게 제어할 수 있습니다. 여기서 `to authenticated`는 인증 과정을 성공적으로 마친 사용자, 즉 시스템이 식별하고 인증한 사용자에게만 해당 작업을 허용한다는 것을 명시합니다.

### to anon

이와 반대되는 개념으로는 `to anon`이 있습니다. Supabase Auth는 모든 요청을 `authenticated` 또는 `anon` 에 매핑합니다. `authenticated` 이 로그인한 유저라면, anon은 로그인 하지 않은 유저를 의미합니다. 

### 예제

```sql
create policy "Profiles are viewable by everyone"
on profiles for select
to authenticated, anon
using ( true );

-- OR

create policy "Public profiles are viewable only by authenticated users"
on profiles for select
to authenticated
using ( true );

```

“Profiles are viewable by everyone” 정책은 인증된 유저와 인증되지 않은 유저 모두에게 읽기를 허용하며, "Public profiles are viewable only by authenticated users" 정책은 오직 인증된 유저에게만 읽기를 허용합니다. 

### service\_role

Supabase는 `service_role` 같은 특별한 "서비스" 키를 제공하는데, 이는 RLS를 우회할 수 있어 관리 작업에 유용하나, 클라이언트 사이드나 고객에게 노출되어서는 안 됩니다. 이는 반드시 백엔드 단에서만 사용해야 합니다. 



**Supbase는 클라이언트 라이브러리가 Service Key로 초기화된 경우에도 서명한 사용자의 RLS 정책을 준수합니다.**

## auth 모듈

### auth.uid()

`auth.uid()`는 현재 인증된 사용자의 고유 식별자(ID)를 반환하는 함수입니다. Supabase는 내부적으로 사용자 인증 시스템을 갖추고 있으며, 사용자가 로그인할 때 각 사용자에게 고유한 ID를 할당합니다. 이 ID는 사용자의 세션 및 인증 정보와 연결되어 있으며, `auth.uid()`를 통해 현재 세션의 사용자 ID를 쉽게 조회할 수 있습니다. 이를 통해 데이터베이스 내에서 사용자별 데이터 접근을 제어하거나 사용자 식별 정보를 기반으로 한 작업을 수행할 수 있습니다.

예를 들어, 사용자가 파일을 업로드할 때, `auth.uid()`를 사용하여 해당 파일이 특정 사용자에게 속함을 식별하거나, 사용자별로 데이터를 격리하여 보안을 강화하는 데 활용할 수 있습니다. 사용자가 로그인 상태가 아니라면, `auth.uid()`는 `null`이나 유효하지 않은 값을 반환할 수 있으므로, `to authenticated` 구문과 함께 사용하여 인증된 사용자에게만 특정 작업을 허용하는 방식으로 보안을 관리합니다.

### auth.jwt()

현재 요청을 하는 사용자의 JWT(Json Web Token)를 반환합니다. JWT 내에는 사용자의 `app_metadata` 및 `user_metadata` 컬럼에 저장된 정보가 포함될 수 있습니다.

- `user_metadata`: 사용자가 `supabase.auth.update()` 함수를 통해 업데이트할 수 있는 메타데이터입니다. 사용자 관련 정보(예: 선호 설정)를 저장하는 데 적합하지만, 인증 데이터를 저장하기에는 적합하지 않습니다.

- `app_metadata`: 사용자가 업데이트할 수 없기 때문에 인증 데이터를 저장하는 데 적합한 메타데이터입니다. 예를 들어, 사용자가 특정 팀에 속해 있는지 여부를 판단하는 데 사용할 수 있는 팀 데이터를 여기에 저장할 수 있습니다.

```sql
create policy "User is in team"
on my_table
to authenticated
using ( team_id in (select auth.jwt() -> 'app_metadata' -> 'teams'));
```

예를 들어, 사용자가 어떤 팀에 속해 있는지 확인하는 데 `app_metadata` 내에 저장된 팀 데이터(팀 ID 배열 등)를 사용할 수 있습니다. 

## 결론

Supabase에서는 항상 RLS를 사용할 것을 권장하고 있습니다. 만약 모두의 접근이 필요한 경우에도, RLS를 활성화하고 인증된 유저와 인증되지 않은 유저에게 권한을 부여하는 방식을 사용할 것을 권장합니다. 

## 참고

[https://supabase.com/docs/guides/database/postgres/row-level-security](https://supabase.com/docs/guides/database/postgres/row-level-security)

