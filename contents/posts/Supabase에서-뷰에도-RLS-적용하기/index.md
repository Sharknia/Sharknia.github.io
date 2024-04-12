---
IDX: "NUM-171"
tags:
  - Supabase
  - Postgresql
  - DataBase
description: "security invoker 옵션을 활용해 뷰에 RLS 적용"
update: "2024-04-12T10:25:00.000Z"
date: "2024-04-12"
상태: "Ready"
title: "Supabase에서 뷰에도 RLS 적용하기"
---
Supabase 특성 상, RLS와 뷰를 적극 활용해야 합니다. 그런데 create or replace 명령어를 사용해서 뷰를 생성하면 뷰의 생성에 사용된 테이블들의 RLS가 적용되지 않는 문제가 있습니다. 
예를 들어, 

```sql
CREATE OR REPLACE VIEW v_user_data AS
SELECT
  id,
  user_id,
  purchase_date,
  amount
FROM
  purchases
WHERE
  valid_until > now() 
ORDER BY
  purchase_date;

```

purchases등의 테이블에 RLS를 설정해서 본인의 것만 조회를 할 수 있도록 설정해두고 뷰를 생성한다면, 

```sql
select * from v_user_data 
```

를 사용해서 조회한다면 RLS가 적용되지 않고 모든 ROW가 조회되는 것을 볼 수 있습니다. 

```sql
select
  *
from
  (
    SELECT
      id,
      user_id,
      purchase_date,
      amount
    FROM
      purchases
    WHERE
      valid_until > now()
    ORDER BY
      purchase_date
  ) T
```

이렇게 조회를 한다면 정상적으로 RLS가 적용되어 본인의 것만 조회가 됨을 확인할 수 있습니다. 

이는 제가 의도한 작동이 아니며, 뷰에도 RLS가 적용되게 할 필요가 있습니다. 

## security invoker

이를 위해 security invoker 옵션이 PostgreSQL v15부터 도입되었습니다. 이를 통해 뷰를 통해 데이터에 접근할 때 원본 테이블의 행 레벨 보안(Row-Level Security, RLS) 정책이 호출자(invoker) 기준으로 적용됩니다. 사용법은 다음과 같습니다. 

뷰를 생성한 후에, 

```sql
alter view v_user_data set (security_invoker=on);
```

이렇게 `security_invoker` 옵션을 on으로 설정해주면 원본 테이블들의 RLS가 뷰에도 적용된 것을 확인할 수 있습니다. 



