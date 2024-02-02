---
IDX: "NUM-42"
tags:
  - Python
update: "2024-02-02T16:32:00.000Z"
date: "2023-08-23"
상태: "Ready"
title: "print와 pprint"
---
print와 pprint는 모두 Python에서 사용되는 출력 관련 함수이다. 

### print

값을 터미널에 간단히 출력할 때 사용한다. 

```python
x = 10
y = "Hello"
print(x, y)  # 출력: 10 Hello
```

### pprint

pprint는 pretty print의 약자로, 복잡한 데이터 구조를 읽기 쉽게 출력하는 데에 사용된다. 데이터 구조를 계층적으로 출력하고 들여쓰기를 적용하여 가독성을 향상시킨다. 

```python
import pprint

data = {'name': 'John', 'age': 30, 'city': 'New York'}
pprint.pprint(data)
```

예를 들어 위와 같은 코드를 실행하면,

```python
{'age': 30,
 'city': 'New York',
 'name': 'John'}
```

로 출력이 된다. 

딕셔너리, 리스트, 중첩된 데이터 구조 등을 보기 좋게 출력해주어 디버깅이나 데이터 분석시에 유리하다. 

