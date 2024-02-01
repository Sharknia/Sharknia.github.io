---
tags:
  - Python
update: "2024-02-01"
date: "2023-10-17"
상태: "Ready"
title: "dataclass"
---
dataclass는 [[파이썬]]3.7부터 표준 라이브러리의 일부로 도입된 데코레이터이다.
주로 클래스를 정의할 때 반복적으로 필요한 특수 메서드들(`__init__`, `__repr__`, `__eq__` 등)을 자동으로 생성해주는 역할을 한다. 이를 통해 데이터를 저장하고 처리하는 클래스를 보다 간결하게 정의할 수 있다.

## 주요 특징

1. 자동 생성된 생성자

1. 자동 생성된 표현 메서드

1. 자동 생성된 비교 메서드

1. 불변성 옵션

## 예제

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float


```

이 코드는 아래의 코드를 간결하게 표현한 것이다.

```python
class Point:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point(x={self.x}, y={self.y})"

    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y


```

## 결론

dataclass는 데이터 구조나 간단한 객체를 정의할 때 유용하게 사용된다. 그러나 복잡한 로직이나 특별한 초기화가 필요한 경우에는 일반 클래스를 사용하는 것이 더 나을 수 있다.



