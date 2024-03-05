---
IDX: "NUM-152"
tags:
  - SqlAlchemy
  - SQLModel
description: "SQLModel에서 unique_constraints 설정하기(Feat. SQLAlchemy)"
update: "2024-03-05T07:48:00.000Z"
date: "2024-03-05"
상태: "Ready"
title: "SQLModel에서 unique_constraints 설정하기"
---
## unique\_constraints란? 

테이블의 특정 컬럼 또는 컬럼의 조합에 대해 유니크 제약조건을 설정하는 데 사용됩니다. 이는 데이터베이스에 동일한 값을 가진 중복 레코드가 없도록 보장하는 데 유용합니다.

## unique\_constraints 설정하기

클래스 내부에 `__table_args__` 속성을 정의하고, unique\_constraints 튜플을 이 속성에 할당합니다. 이 튜플 내에서, 각 유니크 제약조건은 컬럼 이름을 담은 튜플로 표현됩니다.

### 예시 코드

다음은 User 모델에 대해 `email`과 `username` 컬럼 조합에 유니크 제약조건을 설정하는 예시입니다.

```python
from sqlmodel import Field, SQLModel

class User(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    username: str
    email: str
    
    __table_args__ = (
        UniqueConstraint('username', 'email'),
    )

```

위 코드는 같은 `username`과 `email` 값을 가진 두 개의 `User` 레코드가 데이터베이스에 존재할 수 없음을 의미합니다.

### 주의 사항

`__table_args__` 는 SQLAlchemy의 기능을 직접 사용합니다. 따라서, SQLModel을 사용할 때 SQLAlchemy에 대한 기본적인 이해가 필요할 수 있습니다.



