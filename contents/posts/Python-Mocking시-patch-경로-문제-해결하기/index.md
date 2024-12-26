---
IDX: "NUM-225"
tags:
  - Python
  - FastAPI
  - TestCode
description: "Mocking과 경로 문제: 함수 선언 위치 vs 사용 위치"
update: "2024-12-26T10:43:00.000Z"
date: "2024-12-19"
상태: "Ready"
title: "Python Mocking시 patch 경로 문제 해결하기"
---
![](image1.png)
## 서론

테스트 코드를 작성하다 보면 `Mocking`을 자주 사용하게 됩니다. 특히 `pytest-mock`의 `mocker.patch`는 외부 의존성을 대체하고 독립적인 테스트를 작성할 때 매우 유용합니다. 그런데 최근에 테스트를 작성하면서 Mocking을 적용했음에도 불구하고 원래 함수가 호출되는 문제를 겪었습니다.

결론부터 말하면, Mocking할 함수의 "사용 경로"와 "선언 경로"를 혼동한 것이 문제였습니다. 이 블로그에서는 이 문제의 원인과 해결 방법을 공유하겠습니다.

## 문제 상황

### 테스트 대상 코드

테스트하려는 서비스 함수는 아래와 같이 특정 `Diary`와 관련된 자산 정보를 조회합니다.

```python
from app.domains.diaries.repositories import get_diary_by_id
from app.domains.diary_assets.repositories import get_assets_by_diary_id
from app.domains.diary_assets.schemas import AssetDetailResponse

def get_assets_by_diary_service(db: Session, diary_id: int, user_id: int) -> list[AssetDetailResponse]:
    diary = get_diary_by_id(db, diary_id)
    if not diary:
        raise HTTPException(status_code=404, detail="Diary not found")

    if diary.user_id != user_id:
        raise HTTPException(status_code=403, detail="You do not have permission to access this diary")

    diary_assets = get_assets_by_diary_id(db, diary_id)
    return [
        AssetDetailResponse(
            id=asset.id,
            diary_id=asset.diary_id,
            asset_id=asset.asset_id,
            amount=asset.amount,
            buy_price=asset.buy_price,
            created_at=asset.created_at,
            updated_at=asset.updated_at,
        )
        for asset in diary_assets
    ]

```

이 함수에서 `get_diary_by_id`와 `get_assets_by_diary_id`는 외부 레포지토리를 호출합니다. 이를 Mocking하여 독립적인 테스트를 작성하려고 했습니다.

### **잘못된** Mocking

처음에는 아래와 같이 Mocking을 작성했습니다.

```python
mocker.patch("app.domains.diaries.repositories.get_diary_by_id", return_value=mock_diary)
mocker.patch("app.domains.diary_assets.repositories.get_assets_by_diary_id", return_value=mock_assets)

```

이 코드가 올바르게 작동해 `get_diary_by_id` 메소드가 mock\_diary을 리턴할 것이라고 예상했지만, 테스트 중 mocker가 아니라 실제 `get_diary_by_id` 함수가 호출되었습니다(이 부분은 몇 번의 실패 끝에 실제 함수에 로그를 남겨 확인할 수 있었습니다.) 

### **문제의 원인**

Mocking의 대상은 함수가 선언된 위치가 아니라, 테스트 대상 함수에서 해당 함수를 가져오는 경로여야 합니다.

- `get_assets_by_diary_service` 함수는 `app.domains.diary_assets.services` 모듈에서 `get_diary_by_id`를 import하여 사용합니다.

- 따라서, Mocking할 때도 `services` 모듈에서 `get_diary_by_id`를 사용하는 경로를 지정해야 합니다.

## 해결: 올바른 Mocking

아래와 같이 경로를 수정해야 Mocking이 제대로 적용됩니다.

```python
mocker.patch("app.domains.diary_assets.services.get_diary_by_id", return_value=mock_diary)
mocker.patch("app.domains.diary_assets.services.get_assets_by_diary_id", return_value=mock_assets)
```

이제 테스트 대상 함수가 사용하는 의존성을 올바르게 Mocking할 수 있습니다.

## 정리

### Mocking의 핵심: "사용 경로를 기준으로"

1. 테스트 대상 함수에서 import된 경로를 기준으로 설정해야 합니다.

1. 함수가 선언된 원래 위치를 Mocking하면, 테스트 대상 함수에서는 여전히 원래 함수를 호출하게 됩니다.

### **왜 이런 문제가 발생할까?**

Python에서 `import`는 모듈 간의 의존성을 설정합니다. 테스트 대상 함수가 의존성을 import한 순간, Python은 해당 모듈의 네임스페이스에 함수를 바인딩합니다. 따라서 Mocking은 실제 사용된 모듈의 네임스페이스에서 함수 대체를 수행해야 효과가 있습니다.

이 문제는 `unittest.mock`, `pytest-mock` 등 모든 Mocking 라이브러리에서 동일하게 발생할 수 있습니다.

## 마무리

이 문제로 몇 시간을 헤매면서 Mocking의 원리를 제대로 이해하게 되었습니다. 지금은 당연하게 느껴지지만, 처음 겪을 때는 정말 답답했습니다. 알게된 후에도 자주 하게 되는 실수로, 제일 먼저 잘못된건 아닌지 확인하게 되는 습관이 들었습니다. 

이 경우에는 실제 함수에 로그를 남겨보는 것이 가장 알기 쉬웠습니다. 

