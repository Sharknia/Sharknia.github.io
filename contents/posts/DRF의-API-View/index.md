---
IDX: "NUM-194"
tags:
  - Django
  - Python
  - DRF
description: "Django REST Framework API 뷰에 대한 정리"
update: "2024-06-05T04:37:00.000Z"
date: "2024-06-05"
상태: "Ready"
title: "DRF의 API View"
---
![](image1.png)
DRF에서 다양한 HTTP 메서드(`GET`, `POST`, `PUT`, `PATCH`, `DELETE`)를 처리하기 위한 여러 가지 API 뷰 클래스들을 소개하고, 이를 효과적으로 사용하는 방법을 안내하겠습니다. 또한, `GenericAPIView`를 활용하여 직접 뷰를 구현하는 방법도 다룹니다.

## DRF의 기본 뷰 클래스들을 활용한 구현

### 공통 속성들에 대한 설명

#### serializer\_class

`serializer_class`는 뷰에서 사용할 시리얼라이저 클래스를 지정합니다. 시리얼라이저는 Django 모델 인스턴스를 JSON으로 변환하거나, 클라이언트로부터 받은 JSON 데이터를 Django 모델 인스턴스로 변환하는 역할을 합니다.

```python
class CustomUserSerializer(serializers.ModelSerializer):
    class Meta:
        model = CustomUser
        fields = ['id', 'username', 'nickname', 'dob', 'avatar']
```

#### queryset

`queryset`은 뷰에서 처리할 기본 쿼리셋을 지정합니다. 이는 데이터베이스에서 특정 모델의 인스턴스를 선택하는 데 사용됩니다.

```python
queryset = CustomUser.objects.all()
```

#### permission\_classes

`permission_classes`는 뷰에 대한 접근 권한을 제어합니다. 예를 들어, 인증된 사용자만 접근할 수 있도록 설정할 수 있습니다.

```python
permission_classes = [IsAuthenticated]
```

### UpdateAPIView: PUT, PATCH 요청 처리

`UpdateAPIView`는 객체를 업데이트하는 데 사용됩니다. `PUT` 요청은 객체의 모든 필드를 업데이트하고, `PATCH` 요청은 일부 필드만 업데이트합니다.

```python
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from .models import CustomUser
from .serializers import CustomUserSerializer

class UpdateCustomUserView(generics.UpdateAPIView):
    queryset = CustomUser.objects.all()
    serializer_class = CustomUserSerializer
    permission_classes = [IsAuthenticated]

    def get_object(self):
        return self.request.user
```

### CreateAPIView: POST 요청 처리

`CreateAPIView`는 새로운 객체를 생성하는 데 사용됩니다.

```python
from rest_framework import generics
from .models import CustomUser
from .serializers import CustomUserSerializer

class CreateCustomUserView(generics.CreateAPIView):
    queryset = CustomUser.objects.all()
    serializer_class = CustomUserSerializer
```

### RetrieveAPIView: GET 요청 처리

`RetrieveAPIView`는 단일 객체를 가져오는 데 사용됩니다.

```python
from rest_framework import generics
from .models import CustomUser
from .serializers import CustomUserSerializer

class RetrieveCustomUserView(generics.RetrieveAPIView):
    queryset = CustomUser.objects.all()
    serializer_class = CustomUserSerializer
    permission_classes = [IsAuthenticated]

    def get_object(self):
        return self.request.user
```

### DestroyAPIView: DELETE 요청 처리

`DestroyAPIView`는 객체를 삭제하는 데 사용됩니다.

```python
from rest_framework import generics
from .models import CustomUser

class DestroyCustomUserView(generics.DestroyAPIView):
    queryset = CustomUser.objects.all()
    permission_classes = [IsAuthenticated]

    def get_object(self):
        return self.request.user
```

### ListAPIView: 다수의 객체를 GET 요청으로 가져오기

`ListAPIView`는 여러 객체를 리스트 형태로 가져오는 데 사용됩니다.

```python
from rest_framework import generics
from .models import CustomUser
from .serializers import CustomUserSerializer

class ListCustomUserView(generics.ListAPIView):
    queryset = CustomUser.objects.all()
    serializer_class = CustomUserSerializer
```

## GenericAPIView와 Mixins를 활용한 구현

DRF는 `GenericAPIView`와 여러 `mixins`를 제공하여 보다 유연하게 뷰를 구성할 수 있습니다. 이를 통해 기본 제공되는 뷰 클래스를 사용하지 않고도 비슷한 기능을 구현할 수 있습니다.

### mixins?

Django REST Framework(DRF)에서 `mixins`는 뷰 클래스에 특정한 기능을 추가하는 작은 모듈입니다. 여러 `mixins`를 조합하여 하나의 뷰에서 다양한 기능을 구현할 수 있습니다. DRF의 `mixins`는 특히 `GenericAPIView`와 함께 사용되어 CRUD(Create, Read, Update, Delete) 작업을 수행하는 데 유용합니다.

### UpdateAPIView를 GenericAPIView로 구현

`GenericAPIView`와 `UpdateModelMixin`을 사용하여 `PUT`과 `PATCH` 요청을 처리합니다.

```python
from rest_framework import generics, mixins
from rest_framework.permissions import IsAuthenticated
from .models import CustomUser
from .serializers import CustomUserSerializer

class UpdateCustomUserView(generics.GenericAPIView, mixins.UpdateModelMixin):
    queryset = CustomUser.objects.all()
    serializer_class = CustomUserSerializer
    permission_classes = [IsAuthenticated]

    def get_object(self):
        return self.request.user

    def patch(self, request, *args, **kwargs):
        return self.partial_update(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)
```

### CreateAPIView를 GenericAPIView로 구현

`GenericAPIView`와 `CreateModelMixin`을 사용하여 `POST` 요청을 처리합니다.

```python
from rest_framework import generics, mixins
from .models import CustomUser
from .serializers import CustomUserSerializer

class CreateCustomUserView(generics.GenericAPIView, mixins.CreateModelMixin):
    queryset = CustomUser.objects.all()
    serializer_class = CustomUserSerializer

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

### RetrieveAPIView를 GenericAPIView로 구현

`GenericAPIView`와 `RetrieveModelMixin`을 사용하여 `GET` 요청을 처리합니다.

```python
from rest_framework import generics, mixins
from .models import CustomUser
from .serializers import CustomUserSerializer

class RetrieveCustomUserView(generics.GenericAPIView, mixins.RetrieveModelMixin):
    queryset = CustomUser.objects.all()
    serializer_class = CustomUserSerializer
    permission_classes = [IsAuthenticated]

    def get_object(self):
        return self.request.user

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)
```

### DestroyAPIView를 GenericAPIView로 구현

`GenericAPIView`와 `DestroyModelMixin`을 사용하여 `DELETE` 요청을 처리합니다.

```python
from rest_framework import generics, mixins
from .models import CustomUser

class DestroyCustomUserView(generics.GenericAPIView, mixins.DestroyModelMixin):
    queryset = CustomUser.objects.all()
    permission_classes = [IsAuthenticated]

    def get_object(self):
        return self.request.user

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

### ListAPIView를 GenericAPIView로 구현

`GenericAPIView`와 `ListModelMixin`을 사용하여 여러 객체를 가져오는 `GET` 요청을 처리합니다.

```python
from rest_framework import generics, mixins
from .models import CustomUser
from .serializers import CustomUserSerializer

class ListCustomUserView(generics.GenericAPIView, mixins.ListModelMixin):
    queryset = CustomUser.objects.all()
    serializer_class = CustomUserSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
```

