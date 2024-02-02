---
IDX: "NUM-74"
tags:
  - Python
update: "2024-02-02T16:32:00.000Z"
date: "2023-11-02"
상태: "Ready"
title: "Pydantic 모델"
---
`Pydantic` 은 파이썬의 타입 힌트 시스템을 기반으로 데이터 검증 및 설정 관리를 제공하는 라이브러리이다. 특히 FastAPI에서는 Pydantic을 주로 요청 및 응답 객체의 데이터 검증, 직렬화 및 역직렬화에 사용한다. 

## 주요 특징

### 타입 힌트 기반

Pydantic 모델은 파이썬의 타입 힌트를 활용하여 선언된다. 이를 통해 코드 내에서 명확하게 데이터의 구조와 타입을 지정할 수 있다. 

### 자동 데이터 검증

모델에 데이터를 할당할 때에 자동으로 데이터 검증이 수행된다. 유효하지 않은 데이터는 오류를 발생시킨다. 

### 데이터 변환

입력 데이터를 적절한 타입으로 자동 변환한다. 예를 들어 문자열로 들어온 숫자 데이터를 정수로 변환할 수 있다. 

### 기본값 및 유효성 검사

모델 필드에 기본값을 제공하거나 유효성 검사 규칙을 추가할 수 있다. 

### JSON 직렬화 및 역직렬화

자동으로 JSON 형태로 직렬화되며 JSON 데이터를 역직렬화 하여 모델 객체로 변환할 수 있다. 

## 예시

```python
from pydantic import BaseModel, ValidationError

class User(BaseModel):
    id: int
    name: str
    age: int
    email: str

# 사용 예
user_data = {
    "id": 1,
    "name": "John Doe",
    "age": 30,
    "email": "johndoe@example.com"
}

user = User(**user_data)
print(user.json())  # 모델 객체를 JSON 형태로 출력

try:
    invalid_data = {
        "id": "invalid",
        "name": "John",
        "age": "invalid",
        "email": "johndoe"
    }
    User(**invalid_data)
except ValidationError as e:
    print(e.errors())  # 데이터 검증 오류 출력
```



