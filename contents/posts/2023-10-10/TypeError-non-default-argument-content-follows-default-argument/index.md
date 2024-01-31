---
tags:
  - Python
update: "2024-01-31"
date: "2023-10-10"
상태: "Ready"
title: "TypeError: non-default argument 'content' follows default argument"
---
`TypeError: non-default argument 'content' follows default argument`



이 오류는 Python 클래스 또는 함수에서 기본값이 설정된 인자 뒤에 기본값이 설정되지 않은 인자가 위치할 때 발생한다. 예를 들어, 나는 다음과 같았다. 

```python
@dataclass
class CsDetailQueryReturnDto:
    ...
    occurrence_datetime: Optional[str] = None
    ...
    content: str
    cs_type: CsType
    ...
```

`occurrence_datetime` 필드에 기본값 `None`이 설정되어 있고, 이 필드 뒤에 기본값이 설정되지 않은 `content`와 `cs_type` 필드가 위치하고 있다.

### 해결 방법

이 문제가 발생한 이유는 기본값이 설정된 매개변수 뒤에 기본값이 설정되지 않은 매개변수가 올 수 없다는 규칙 때문이다. 

함수를 호출할 때, Python은 매개변수를 위치나 키워드로 판별할 수 있어야 한다. 만약 기본값이 없는 매개변수가 기본값이 있는 매개변수 뒤에 온다면, Python은 어떤 매개변수에 값을 할당해야 하는지 혼동할 수 있다.



