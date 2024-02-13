---
IDX: "NUM-130"
tags:
  - SqlAlchemy
description: "Sqlalchemy에서 index 선언하기"
update: "2024-02-13T13:19:00.000Z"
date: "2024-02-13"
상태: "Ready"
title: "Sqlalchemy의 index"
---
## 서론

DB 설계를 하다보면 index를 정의해야 하는 경우가 많습니다.

만약 Sqlalchemy의 [create\_all()](https://sharknia.github.io/Sqlalchemy의-create_all) 메소드를 사용해 테이블을 생성하고 있다면 Sqlalchemy에서 동시에 index를 정의할 수 있습니다. 

## Sqlalchemy에서의 인덱스 정의

### 단일 컬럼 인덱스

Sqlalchemy의 `Column` 객체를 생성할 때에 `index=True`  플래그를 설정하면 간단하게 단일 컬럼에 대한 인덱스를 생성할 수 있습니다. 

```python
Column('user_id', Integer, index=True)
```

### 복합 인덱스

두 개 이상의 컬럼을 포함하는 복합 인덱스는 `Index` 객체를 사용하여 생성할 수 있습니다. 

```python
from sqlalchemy import Index
Index('my_index', MyModel.column1, MyModel.column2.desc())
```

여기서 `desc()` 메소드는 해당 컬럼을 내림차순으로 정렬하도록 지시합니다. 오름차순은 기본값이므로 `asc()`는 일반적으로 생략됩니다.

### Unique 인덱스

Unique 인덱스는 값의 중복을 방지하고 해당 컬럼의 각 값이 유일하다는 것을 보장합니다. 

```python
Column('email', String, unique=True)
```

### 복합 Unique 인덱스

SqlAlchemy에서는 복합 인덱스에 대해서도 유니크 제약 조건을 적용할 수 있습니다. 이를 통해 테이블 내에서 특정 컬럼 조합의 유니크성을 강제할 수 있습니다. 

SqlAlchemy에서 복합 유니크 인덱스를 정의하는 방법은 두 가지입니다. 

1. `UniqueConstraint` 사용

    `UniqueConstraint`는 테이블의 컬럼에 대한 유니크 제약 조건을 정의하는 데 사용됩니다. 이는 `__table_args__` 속성 내부에서 사용될 수 있습니다.

    ```python
    from sqlalchemy import Column, Integer, String, create_engine
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.schema import UniqueConstraint
    
    Base = declarative_base()
    
    class MyModel(Base):
        __tablename__ = 'my_model'
        id = Column(Integer, primary_key=True)
        column1 = Column(Integer)
        column2 = Column(String)
        __table_args__ = (UniqueConstraint('column1', 'column2', name='my_unique_constraint'),)
    ```

1. `Index` 사용 시 `unique=True` 설정

    `Index` 객체를 생성할 때 `unique=True` 플래그를 설정하여 복합 유니크 인덱스를 생성할 수 있습니다. 이 방법은 인덱스 생성과 동시에 컬럼 조합의 유니크성도 보장합니다.

    ```python
    from sqlalchemy import Column, Integer, String, MetaData, Table
    from sqlalchemy.schema import Index
    
    metadata = MetaData()
    my_table = Table('my_table', metadata,
        Column('id', Integer, primary_key=True),
        Column('column1', Integer),
        Column('column2', String),
    )
    
    # 복합 유니크 인덱스 생성
    my_unique_index = Index('my_unique_index', my_table.c.column1, my_table.c.column2, unique=True)
    ```

두 방법 모두 데이터베이스에서 해당 컬럼 조합의 유니크성을 강제합니다. `UniqueConstraint`는 주로 제약 조건을 명시적으로 표현할 때 사용되며, `Index`의 `unique=True` 설정은 검색 최적화와 유니크 제약 조건을 동시에 적용하고자 할 때 유용합니다.

### **_\_table\_args\__**

  `__table_args__` 클래스 속성은 튜플이나 사전 형태로 제공될 수 있으며, 테이블 생성 시 추가적인 SQL 제약 조건이나 인덱스를 정의하는데 사용됩니다.  

예를 들어 복합 인덱스를 `__table_args__` 클래스 속성을 사용해 모델 내부에서 정의한 예제 코드는 다음과 같습니다. 

```python
class Event(Base):
    __tablename__ = 'events'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    timestamp = Column(DateTime)
    user_id = Column(Integer)
    is_active = Column(Boolean)

    # 복합 인덱스 정의
    __table_args__ = (
        Index('ix_events_user_id_timestamp', 'user_id', timestamp.desc()),
    )
```

이 방식으로 정의된 인덱스는 데이터베이스에 해당 모델을 반영할 때 자동으로 생성됩니다. 



