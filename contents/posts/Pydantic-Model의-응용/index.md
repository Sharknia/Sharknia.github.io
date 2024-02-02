---
IDX: "NUM-113"
tags:
  - Work
  - Python
description: "커스텀된 Base Pydantic Model class 생성 - 네이밍 컨벤션 문제 해결"
update: "2024-02-02T16:32:00.000Z"
date: "2024-01-26"
상태: "Ready"
title: "Pydantic Model의 응용"
---
## 문제

기존에는 프론트와 주고 받을 데이터 모델을 정의하는데에 [dataclass](https://sharknia.github.io/dataclass)를 사용하고 있었다. 그러다가 FastAPI로 넘어오면서 [Pydantic model](https://sharknia.github.io/Pydantic-모델)을 도입했다. 

dataclass로는 해결할 수 없는 문제가 있었다. 프론트엔드 개발에서는 주로 카멜케이스를 사용하기 때문에, 파이썬에서는 주로 스네이크 케이스를 사용하는데 ([네이밍 규칙(naming conventions)](https://sharknia.github.io/네이밍-규칙naming-conventions)), 프론트로 나갈 변수 명을 카멜케이스로 통일 하기 위해 어쩔 수 없이 백엔드의 코드 컨벤션을 해치면서 Response로 나갈 모델의 속성만 Camel 케이스로 선언하고 있었다. 

하지만 난 이런게 너무 싫다. 

## 해결

Pydantic Model을 이를 해결하기 위한 옵션이 있다. 

### alias_generator

필드에 대한 별칭을 자동으로 생성하는 함수를 지정한다. 주로 모델 필드 이름을 snake 케이스에서 camel케이스로 매핑할 때 사용된다. 

### populator_by_name

이 옵션을 True로 설정하면 Pydantic은 별칭 대신 모델 필드의 원래 이름을 사용하여 인스턴스를 생성하고 할당한다. 이는 필드의 별칭과 원래 이름을 혼용하여 사용할 수 있게 한다. 기본적으로는 False로 되어있어 만약 이 설정을 수정하지 않는다면 속성을 snake case로 정의했어도 인스턴스 생성시 변수를 camelCase로 넣어줘야 하는 불상사가 일어난다. 

이 설정을 True로 해두면 모델 인스턴스가 직렬화, 역직렬화 되는 과정에서만 별칭을 이용하게 된다. 

### 두 옵션을 엮어서..

이 두 옵션을 엮어서 Request로 받을 경우, Response로 내려갈 경우에만 별칭을 사용하고, 별칭은 camelCase에 따라 생성되도록 설정을 할 수 있다. 

그리고 이 설정값을 넣은 BastDtoModel 클래스를 생성해서 앞으로 우리가 데이터 통신에 사용할 PydanticModel은 모두 이 클래스를 상속받아 구현하기로 했다. 

<details>
<summary>base_dto_model.py</summary>

```python
from pydantic import BaseModel


def to_camel(string: str) -> str:
    components = string.split('_')
    return components[0] + ''.join(x.title() for x in components[1:])


class BaseDtoModel(BaseModel):
    class Config:
        alias_generator = to_camel
        populate_by_name = True
```


</details>

alias_generator에는 to_camel 메소드를 정의해주었다. to_camel 메소드는 snake_case로 작성된 문자열을 camelCase로 변환해주는 함수이다. 



