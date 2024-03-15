---
IDX: "NUM-162"
tags:
  - Postgresql
description: "쿼리문 날려서 함수, 트리거 생성 쿼리문 확인하기"
update: "2024-03-15T02:17:00.000Z"
date: "2024-03-15"
상태: "Ready"
title: "Postgresql Function, Trigger 생성 쿼리문 확인하기"
---
## 서론

어떤 함수를 만들었었는지, 어떤 트리거를 만들었었는지가 관리하기가 어렵다고 느낍니다. 현재 개발 상황 상 따로 백엔드 서버도 없는 상황이어서 쿼리문을 관리하려고 합니다. 

Supabase Dashboard에서도 확인이 가능하긴 하지만, 제한적인 부분이 있어 깔끔하게 쿼리문으로 확인하는 방법을 기록해두려고 합니다. 

## Function 생성 쿼리문 확인하기

```sql
SELECT pg_get_functiondef(f.oid)
FROM pg_catalog.pg_proc f
JOIN pg_catalog.pg_namespace n ON (f.pronamespace = n.oid)
WHERE n.nspname = '스키마명'  -- 함수가 위치한 스키마명
AND f.proname = '함수명';    -- 확인하고 싶은 함수명
```

## Trigger 생성 쿼리문 확인하기

```sql
SELECT 
    t.tgname AS trigger_name,
    pg_catalog.pg_get_triggerdef(t.oid) AS trigger_definition,
    p.proname AS trigger_function,
    c.relname AS table_name,
    n.nspname AS schema_name
FROM 
    pg_catalog.pg_trigger t
    JOIN pg_catalog.pg_proc p ON t.tgfoid = p.oid
    JOIN pg_catalog.pg_class c ON t.tgrelid = c.oid
    JOIN pg_catalog.pg_namespace n ON c.relnamespace = n.oid
WHERE 
    c.relname = '테이블명'  -- 트리거가 설정된 테이블명
    AND n.nspname = '스키마명';  -- 테이블이 위치한 스키마명
```

## 뷰 생성 쿼리문 확인하기

사실 뷰 생성 쿼리문은 전문을 supabase에서 확인할 수 있습니다. 근데 그냥 서비스로 같이 기록합니다. 

```sql
SELECT pg_get_viewdef('"스키마명"."뷰명"', true);
```

