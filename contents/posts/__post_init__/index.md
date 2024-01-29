---
tags:
  - Python
update: "2024-01-29"
date: "2023-10-17"
상태: "POST"
title: "__post_init__"
---
`__post_init__` 메소드는 Python의 `dataclasses` 모듈에 있는 `dataclass` 데코레이터를 사용하여 클래스를 정의할 때 사용된다. 

`__post_init__` 메소드는 객체가 초기화된 직후에 자동으로 호출된다. 즉, `__init__` 메소드가 호출된 후에 `__post_init__` 메소드가 호출된다.

클래스의 인스턴스가 생성될 때 `__init__` 메소드가 호출되어 인스턴스 변수들을 초기화하고, 바로 이어서 `__post_init__` 메소드가 호출되어 추가적인 초기화 작업을 수행할 수 있다. 이러한 방식으로, 객체의 초기화 과정 중에 추가적인 로직을 실행할 수 있다.

```python
dto = ContentListQueryDto(payload)
```

`ContentListQueryDto` 클래스의 인스턴스를 생성하면서 `__init__` 메소드가 자동으로 호출되고, 이어서 `__post_init__` 메소드도 자동으로 호출된다. `__post_init__` 메소드는 `__init__` 메소드가 완료된 직후에 실행되므로, `__init__` 메소드에서 설정된 필드 값을 기반으로 추가적인 초기화 작업을 수행할 수 있다.

이 경우에, `dto = ContentListQueryDto(payload)` 코드를 실행하면 `__init__` 메소드가 호출되어 `payload` 딕셔너리의 키-값 쌍을 이용하여 인스턴스 변수들을 초기화하고, 이어서 `__post_init__` 메소드가 호출되어 해당 메소드에 정의된 작업들을 수행한다. 



