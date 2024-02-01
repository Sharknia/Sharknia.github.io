---
tags:
  - SqlAlchemy
  - Python
  - DataBase
description: "Sqlalchemy의 func 사용법"
update: "2024-02-01"
date: "2024-01-30"
상태: "Ready"
title: "Sqlalchemy의 func"
---
## func란? 

SQL 함수를 생성하고 호출하는 데 사용되는 기능이다. SQL 표준 함수 뿐 아니라 데이터베이스 별 특정 함수까지 다룰 수 있다. 

Sqlalchemy의 유연한 기능으로 다양한 데이터베이스 작업을 보다 Pythonic한 방식으로 작성할 수 있게 해준다. 

## func의 특징

- 함수 생성기

    func는 데이터베이스의 내장 함수나 사용자 정의 함수를 파이썬 코드 내에서 호출하기 위한 함수 생성기이다. 

- 동적 생성

    func 객체에 접근할 때 Python의 속성 접근 매커니즘을 통해 동적으로 SQL 함수 호출을 생성한다. 예를 들어 func.count() 는 SQL의 COUNT() 함수 호출을 생성한다. 

- 데이터베이스 독립성

    다양한 데이터베이스 시스템에서 사용할 수 있는 표준 SQL 함수를 추상화한다. 따라서 특정 데이터베이스에 종속적이지 않은 코드를 작성할 수 있다. 

- 다양한 함수 지원

    func는 거의 모든 종류의 SQL 함수를 호출할 수 있도록 지원한다. 

## `__getattr__` method

func 객체의 __getattr__ 메소드는 특정 속성에 접근할 때 호출된다. 이 메소드는 동적으로 SQL 함수 호출을 생성한다. 

예를 들어, func.pg_try_advisory_lock에 접근하면 pg_try_advisory_lock 이름으로 _FunctionGenerator 객체를 생성한다. 이 객체는 최종적으로 SQL 쿼리 내에서 해당 함수 호출을 나타낸다. 

## 사용방법

### 기본 사용

함수 이름을 func 객체의 속성으로 접근하여 사용한다. (__getattr__ 메소드로 연결된다.)

```python
from sqlalchemy import func

# COUNT 함수 호출 예시
query = select([func.count()]).select_from(my_table)
```

### 함수 매개변수 전달

함수 호출 시 매개변수를 전달할 수 있다. 

```python
# LENGTH 함수에 문자열 전달
query = select([func.length('some string')])
```

### 복잡한 표현식

복잡한 SQL 표현식을 만드는 데도 사용할 수 있다. 

```python
# 조건부 SQL 함수 호출
query = select([func.coalesce(my_table.column, 'default value')])
```



