---
IDX: "NUM-161"
tags:
  - Supabase
description: "Supabase CLI db migration 이력 초기화하기"
update: "2024-03-14T08:57:00.000Z"
date: "2024-03-14"
상태: "Ready"
title: "Supabase CLI db migration 이력 초기화하기"
---
## 서론

supabase에는 DB 마이그레이션 기능이 있어, 연결된 프로젝트의 스키마를 그대로 로컬 supabase DB에 적용할 수 있습니다. 버전관리도 가능합니다. 

근데 이것도 완벽한 기능은 아니어서, 꼬일때가 있고 그래서 다시 마이그레이션을 실행해야 할 수도 있습니다. 그 방법을 기록합니다. 

## 마이그레이션 기록 초기화하기

### 로컬 마이그레이션 파일 삭제

다음의 명령어를 입력해 로컬 마이그레이션 파일들을 삭제합니다. 

```bash
rm supabase/migrations/*
```

### 마이그레이션 기록 삭제

기록을 하나하나 reverted 해줘야 합니다. `supabase migration list` 명령어로 초기화 할 마이그레이션 리스트와 REMOTE 키값을 확인합니다. 

다음의 명령어로 한땀 한땀 삭제합니다. 

```bash
supabase migration repair --status reverted [키값]
```

이렇게 모든 기록을 삭제 후, `supabase db pull` 을 실행해 다시 DB를 덤프하면 됩니다. 



