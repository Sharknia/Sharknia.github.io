---
IDX: "NUM-158"
tags:
  - Supabase
description: "테스트를 위한 supabase local 개발 환경 db 세팅"
update: "2024-03-13T09:11:00.000Z"
date: "2024-03-13"
상태: "Ready"
title: "Supabase Local 개발 환경 꾸미기 - DB 세팅"
---
## 개요

[지난 시간](https://sharknia.github.io/Supabase-Local-Dev-환경-꾸미기)에 Egde Function을 만들어서 배포하는 것까지 진행해봤습니다. 하지만 저기서 끝낸다면 로컬에 DB가 없으므로 DB에 접근을 필요로 하는 Edge Function은 로컬에서의 정상적인 테스트가 불가능합니다. 

따라서 이번에는 테스트에 필요한 환경을 구성하는 방법을 정리합니다. 

## 사전 작업

테스트를 한다는 것은 실제 개발 또는 Production 서버와 같은 환경을 꾸며야 한다는 것이고, 이는 데이터나 스키마가 동일해야 한다는 것으로, 현재로서는 개발 환경 프로젝트와 동일한 DB 환경을 꾸미는 것을 목표로 합니다. 따라서, `supabase link ..` 작업까지는 완료가 되어있어야 합니다. 

## DB 스키마 맞추기

### supabase db pull

다음의 명령어를 통해 연결된 프로젝트의 스키마를 모두 생성하는 쿼리문을 생성할 수 있습니다. 

```bash
supabase db pull --linked
```

이렇게 하면 supbase/migrations 디렉토리 안에 `<timestamp>_remote_schema.sql` 꼴의 쿼리문 파일이 생성됩니다. 해당 쿼리문은 자동으로 실행되지는 않으며, 직접 쿼리문을 실행해줘야 합니다. 

초기 설정 이후에는 `supabase db pull --linked`를 실행하면 변경된 사항만 자동으로 쿼리문을 생성합니다. 스키마를 지정해 특정 스키마만 pull 할 수도 있습니다. 

### supabase migration

버전 관리 기능도 갖추고 있습니다. 

- supabase migration list : 마이그레이션 상태를 확인합니다. 

- supabase migration up : 보류 중인 마이그레이션을 로컬 데이터베이스에 적용합니다. 

### supabase db reset

여기까지 해두면, supabase db reset을 할 때마다 데이터베이스를 지웠다가 스키마를 다시 생성합니다. 언제든지 초기 상태로 돌아올 수 있습니다. 

## 테스트 데이터 맞추기

원활한 테스트를 위해서는 스키마는 물론이고 테스트 데이터도 들어있어야 합니다. db reset를 할 때마다 테스트 데이터를 넣기가 여간 번거로운 일이 아닐텐데, seed.sql 파일에 insert 구문을 미리 작성해두면 db reset에서 데이터베이스 삭제 후, migration 쿼리문을 통해 스키마를 맞춘 후 seed.sql의 insert 구문을 실행해 테스트 데이터까지 삽입한 깔끔한 테스팅 상태로 맞춰줍니다. 

## 참고

[https://supabase.com/docs/guides/cli/local-development](https://supabase.com/docs/guides/cli/local-development)

[https://supabase.com/docs/reference/cli/supabase-db-pull](https://supabase.com/docs/reference/cli/supabase-db-pull)

[https://supabase.com/docs/guides/cli/seeding-your-database](https://supabase.com/docs/guides/cli/seeding-your-database)

