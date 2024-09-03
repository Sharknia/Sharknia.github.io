---
IDX: "NUM-200"
tags:
  - Django
  - DRF
  - Python
description: "Django REST Framework에서 가상 필드 추가하기: SerializerMethodField"
update: "2024-09-03T08:52:00.000Z"
date: "2024-06-17"
상태: "Ready"
title: "Django REST Framework에서 가상 필드 추가하기"
---
![](image1.png)
## 서론

Django REST Framework(DRF)를 사용하다 보면, 모델에 없는 가상의 필드를 시리얼라이저에 추가해야 할 때가 있습니다. 이때 유용하게 사용할 수 있는 것이 `SerializerMethodField`입니다. `SerializerMethodField`를 통해 동적으로 계산된 값을 시리얼라이저에 포함할 수 있습니다. 이번 글에서는 이를 구현하는 방법을 간단히 살펴보겠습니다.

## 예제: 가상의 필드 추가하기

예를 들어, `Photos` 모델이 아래와 같이 정의되어 있다고 가정해봅시다:

```python
from django.db import models
import uuid

class Photos(models.Model):
    id = models.UUIDField(default=uuid.uuid4, primary_key=True, editable=False)
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name="photos")
    match_type = models.CharField(max_length=10)
    file_name = models.TextField()
    updated_at = models.DateTimeField(auto_now=True)
    created_at = models.DateTimeField(auto_now_add=True)
```

이 모델을 기반으로 하는 시리얼라이저에 `id`와 `match_type`을 조합한 가상의 필드 `id_match_type`을 추가해보겠습니다.

## `SerializerMethodField` 사용하기

먼저, `SerializerMethodField`를 사용하여 시리얼라이저에 가상의 필드를 추가합니다:

```python
from rest_framework import serializers
from .models import Photos

class PhotoSerializer(serializers.ModelSerializer):
    id_match_type = serializers.SerializerMethodField()  # 가상의 필드 정의

    class Meta:
        model = Photos
        fields = ["id", "match_type", "file_name", "updated_at", "created_at", "id_match_type"]

    def get_id_match_type(self, obj):  # 가상의 필드를 계산하는 메서드 정의
        return f"{obj.id}-{obj.match_type}"


```

위 코드에서 `SerializerMethodField`는 `id_match_type`이라는 이름의 필드를 시리얼라이저에 추가합니다. DRF는 자동으로 `get_id_match_type` 메서드를 찾아 이 필드의 값을 계산합니다. 이 메서드는 객체(`obj`)를 인자로 받아, `id`와 `match_type`을 조합한 문자열을 반환합니다.

## 결과

이제 `PhotoSerializer`를 사용하여 직렬화된 데이터에는 `id_match_type` 필드가 포함됩니다:

```json
{
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "match_type": "CUTE",
    "file_name": "photo.jpg",
    "updated_at": "2024-06-17T10:26:19Z",
    "created_at": "2024-06-17T10:26:19Z",
    "id_match_type": "123e4567-e89b-12d3-a456-426614174000-CUTE"
}


```

## 결론

`SerializerMethodField`는 모델에 존재하지 않는 가상의 필드를 시리얼라이저에 추가할 때 매우 유용한 도구입니다. 이를 통해 동적으로 계산된 값을 클라이언트에게 전달할 수 있습니다. 위 예제에서는 `id`와 `match_type`을 조합한 가상의 필드를 추가하는 방법을 살펴보았습니다. 

