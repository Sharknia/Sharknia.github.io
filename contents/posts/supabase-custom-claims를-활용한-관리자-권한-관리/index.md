---
IDX: "NUM-173"
tags:
  - Supabase
  - Postgresql
description: "supabase-custom-claims를 활용해 일반 유저와 관리자의 권한을 구분하여 관리하자. "
update: "2025-01-08T16:06:00.000Z"
date: "2024-04-25"
상태: "Ready"
title: "supabase-custom-claims를 활용한 관리자 권한 관리"
---
![](image1.png)
## 서론

Supabase에서 특정 edge function이나 db function, 또는 뷰나 테이블의 CRUD에 있어서도 계정에 따라 다른 결과물을 내놔야 하는 경우가 아주 많습니다. 쉽게 예를 들자면 특정 유저에게 내부 재화를 부여하는 기능이 관리자 계정은 가능하지만 일반 사용자 계정은 불가능해야 하는 경우가 있을 수 있겠네요. 

User 데이터가 저장된 테이블에 해당 유저의 권한을 저장해두고 이를 읽어서 사용한다면 쉽게 구현할 수 있겠지만, Supabase에서는 이를 더 효과적으로 구현하기 위한 방법을 준비해두었습니다. 

## supabase-custom-claims?

### 소개

supabase-custom-claims를 사용해 인증된 사용자가 앱에 로그인할 때 받는 액세스 토큰에 JSON 데이터를 임의로 추가 또는 삭제할 수 있습니다. 

사용자의 계획, 레벨, 그룹, 가입 일자, 권한 등 다양한 정보를 포함시킬 수 있어 이를 사용해 애플리케이션의 특정 부분에 대한 Access 권한을 세밀하게 조정할 수 있습니다. 

문자열/숫자/Boolean/배열/JSON 객체등을 모두 저장할 수 있습니다. 저장된 데이터는 사용자 테이블의 raw\_app\_meta\_data 컬럼에 저장됩니다. 

#### 장점

사용자 정의 클레임은 사용자가 로그인할 때 받는 보안 토큰에 저장되며 이 클레임은 PostgreSQL 데이터베이스가 즉시 접근할 수 있도록 구성 매개변수로 제공됩니다. 따라서 데이터베이스의 디스크 입출력 없이 즉시 이 값을 사용할 수 있습니다. 특히 RLS에서 클레임을 사용하면 데이터베이스 호출을 크게 줄일 수 있어 크게 성능을 개선할 수 있습니다. 

#### 단점

자동으로 업데이트 않으며 로그아웃 후 다시 로그인 해야 새 클레임이 적용됩니다. 따라서 자주 변경되는 데이터를 저장하기에는 적합하지 않습니다. supabase.auth.refreshSession()를 호출하여 강제로 새로 고칠 수는 있습니다. 다만 서버나 클레임 관리자의 수동 변경 이후 이를 알릴 수 있는 방법은 없습니다. 

#### 주의

Supabase 인증 시스템에서 `provider`와 `providers`라는 사용자 정의 클레임을 이미 사용하므로 이 이름은 사용하지 않아야 합니다. 

## 특정 유저에게 권한 부여하기

`set_claim` 함수는 특정 유저의 권한 정보를 사용자 정의 클레임(custom claims)으로 추가하거나 업데이트하는 데 사용됩니다. 이 함수는 Supabase에서 제공하는 사용자 관리 기능과 연동하여, `raw_app_meta_data` 필드를 업데이트합니다.

```sql
-- 특정 유저에게 claims_admin 권한 부여
SELECT auth.set_claim(
  user_id := '<user_id>',        -- 유저의 UUID
  claim := 'app_metadata.claims_admin',
  value := 'true'               -- 권한 값 (true/false, 숫자 등)
);
```

#### **매개변수 설명**

1. `user_id`: 권한을 부여할 사용자의 고유 ID(UUID). Supabase의 `auth.users` 테이블에서 확인할 수 있습니다.

1. `claim`: 부여할 클레임의 이름입니다. 예를 들어, `app_metadata.claims_admin`은 관리자로 설정할 때 사용됩니다.

1. `value`: 클레임의 값으로, `true`, `false`, 숫자, 문자열 등 다양한 데이터 타입을 사용할 수 있습니다.

## 특정 유저의 권한 확인하기

`is_claims_admin()` 함수는 현재 사용자 세션에서 `claims_admin` 권한이 있는지를 확인하는 데 사용됩니다. 이를 통해 관리 권한이 있는 사용자만 특정 기능을 사용할 수 있도록 제한할 수 있습니다.

```sql
CREATE OR REPLACE FUNCTION is_claims_admin()
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
  -- 세션의 JWT 클레임에서 claims_admin 값 확인
  RETURN coalesce(
    (current_setting('request.jwt.claims', true)::jsonb)->'app_metadata'->>'claims_admin',
    'false'
  )::boolean;
END;
$$;
```



## DB Function에서의 활용

```sql
    IF NOT is_claims_admin() THEN
        RAISE EXCEPTION 'AccessDenied';
    END IF;
```

## RLS에서의 활용

이 두 기능은 Supabase의 RLS 정책과 함께 사용되며, 아래와 같이 `is_claims_admin()`를 활용해 RLS 정책을 정의할 수 있습니다.

```sql
CREATE POLICY "Allow claims_admin"
ON sensitive_table
FOR ALL
TO authenticated
USING (
  is_claims_admin() = true
);
```

## 기타 : Supabase 대시보드에서의 is\_claims\_admin()

```sql
CREATE OR REPLACE FUNCTION is_claims_admin()
    RETURNS "bool"
    LANGUAGE "plpgsql"
    AS $$
  BEGIN
    IF session_user = 'authenticator' THEN
      --------------------------------------------
      -- To disallow any authenticated app users
      -- from editing claims, delete the following
      -- block of code and replace it with:
      -- RETURN FALSE;
      --------------------------------------------
      IF extract(epoch from now()) > coalesce((current_setting('request.jwt.claims', true)::jsonb)->>'exp', '0')::numeric THEN
        return false; -- jwt expired
      END IF;
      If current_setting('request.jwt.claims', true)::jsonb->>'role' = 'service_role' THEN
        RETURN true; -- service role users have admin rights
      END IF;
      IF coalesce((current_setting('request.jwt.claims', true)::jsonb)->'app_metadata'->'claims_admin', 'false')::bool THEN
        return true; -- user has claims_admin set to true
      ELSE
        return false; -- user does NOT have claims_admin set to true
      END IF;
      --------------------------------------------
      -- End of block 
      --------------------------------------------
    ELSE -- not a user session, probably being called from a trigger or something
      return true;
    END IF;
  END;
$$;
```

Supabase 대시보드에서 is\_claims\_admin()을 실행하면 반드시 true로 떨어집니다! Role을 바꿔서 시뮬레이션 하더라도 그렇습니다. 왜냐하면 대시보드에서 실행한 경우, `session_user` 가 `postgres` 이기 때문입니다. 실제 작동은 원하는대로 작동하지만, 대시보드에서의 테스트가 실제 실행 결과값과 달라져 테스트가 어려워지게 됩니다. Role을 바꾸는게 어떻게 작동하길래 이런 결과가 나올까요?

### Supabase의 Role 변경 기능

Supabase SQL Editor에서는 데이터베이스의 다양한 Role을 사용하여 쿼리를 실행할 수 있는 기능이 제공됩니다. 이는 PostgreSQL의 Role-Based Access Control (RBAC) 기능을 활용한 것으로, 서로 다른 Role을 적용하여 데이터베이스 접근을 시뮬레이션하고, 특정 Role이 어떤 데이터에 접근할 수 있는지 테스트할 수 있습니다.

이 Role 변경 기능은 주로 보안 설정을 테스트하거나 다양한 사용자 권한을 시뮬레이션하는 데 사용됩니다. 예를 들어, 어떤 Role이 특정 테이블에 대한 접근 권한을 가지고 있는지, 또는 특정 작업을 수행할 수 있는지 등을 확인할 수 있습니다.

### Role 변경과 session\_user

하지만 이때 Role을 변경하더라도 `session_user`는 변경되지 않습니다. `session_user`는 데이터베이스에 연결된 사용자의 로그인 이름을 반영하며, 세션의 생애 주기 동안 고정됩니다. Role을 변경한다고 해서 세션을 시작한 사용자가 변경되는 것은 아닙니다. Role은 현재 세션에서 실행 권한을 임시로 변경하는 것에 불과하므로, 이는 `session_user`의 값을 바꾸지 않습니다.

이 기능은 데이터베이스의 보안 설정을 테스트하거나 다른 사용자의 권한으로 쿼리를 실행해 보려는 개발자에게 유용합니다. 하지만 `session_user`는 데이터베이스 세션의 보안 컨텍스트를 유지하는 중요한 요소로, 이를 통해 누가 실제로 데이터베이스에 접근하고 있는지 확인할 수 있습니다.

### `session_user` 와 `current_user`

Edge funcion이나 DB Function을 사용한 경우, 관리자 계정 또는 사용자 계정 둘 모두 `session_user`는 `authenticator` 이며, `current_user`는 `authenticated` 입니다. 

#### session\_user vs current\_user

- `session_user`: 데이터베이스 세션을 시작할 때 설정된 사용자 신원을 나타냅니다. 이 값은 세션 동안 변경되지 않습니다.

- `current_user`: 현재 실행 중인 명령이나 쿼리의 맥락에서 적용되는 사용자를 나타냅니다. `SET ROLE` 또는 `SET SESSION AUTHORIZATION`과 같은 명령을 사용하여 세션 중에 변경될 수 있습니다.

결국 큰 차이는 변경할 수 있냐, 없냐에 있다고 할 수 있습니다. 

하지만 current\_user를 변경하는 것은 클라이언트에서 거의 불가능한 작업입니다. `SET ROLE`이나 `SET SESSION AUTHORIZATION` 같은 명령을 통해 변경을 시도한다고 해도, 본인이 가지지 않은 권한으로의 변경은 사실상 불가능합니다. 

 특히 Supabase는 인증 토큰과 서버 사이드의 보안 설정을 통해 사용자 신원을 관리하며, 클라이언트에서 `current_user`를 임의로 변경할 수 없도록 합니다. 따라서, Supabase를 사용하는 경우 `current_user`를 기반으로 하는 보안 체크도 충분히 안전합니다. 



