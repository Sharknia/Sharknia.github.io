---
IDX: "NUM-188"
tags:
  - Supabase
  - Django
  - Python
description: "JWT 인증을 이용한 Cumstom permission 클래스"
update: "2024-05-22T04:41:00.000Z"
date: "2024-05-22"
상태: "Ready"
title: "Django Rest Framework에서 JWT를 이용한 사용자 인증 구현"
---
![](image1.png)
## 서론

Django Rest Framework(DRF)는 Django에서 RESTful API를 쉽게 구축할 수 있도록 도와주는 도구입니다. JWT(Json Web Token)는 클라이언트와 서버 간의 인증을 안전하게 수행하기 위한 방법으로 널리 사용됩니다. 이번에는 DRF에서 JWT를 이용한 사용자 인증을 구현하고, 이를 위한 커스텀 퍼미션 클래스를 작성해보겠습니다.

## **Django Rest Framework(DRF)란?**

Django Rest Framework(DRF)는 Django를 위한 강력하고 유연한 도구로, RESTful API를 쉽게 구축하고 관리할 수 있게 해줍니다. DRF는 직관적인 API 개발을 위해 다양한 기능을 제공하며, 대표적인 기능으로는 다음과 같은 것들이 있습니다.

- Serializer: 복잡한 데이터 쿼리셋과 모델 인스턴스를 JSON 등으로 직렬화/역직렬화하는 도구.

- Viewsets: 기본 CRUD 작업을 간편하게 구현할 수 있는 뷰 로직.

- Authentication & Permissions: 다양한 인증 및 권한 부여 메커니즘을 제공하여 보안 설정을 강화.

- Browsable API: 개발 중에 API를 쉽게 테스트하고 문서화할 수 있는 웹 브라우저 기반 인터페이스.

이 모든 기능을 통해 DRF는 백엔드 개발자가 빠르고 효율적으로 웹 API를 구축할 수 있도록 도와줍니다.

## **JWT란?**

JWT는 JSON 형식의 데이터를 사용하여 정보의 무결성과 진위를 검증하는 토큰입니다. 주로 인증과 정보 교환에 사용되며, 클라이언트와 서버 간의 통신에서 사용자가 인증되었음을 보증하는 데 사용됩니다.

## 구현

### JWT

JWT는 Supabase를 활용한 인증을 거쳐 로그인을 할 때 발급됩니다. 

[이 페이지](https://supabase.com/docs/guides/auth/jwts#jwts-in-supabase)를 참고하면 Supabase에서 생성되는 JWT의 꼴을 알 수 있습니다. 

```json
{
  "aud": "authenticated",
  "exp": 1615824388,
  "sub": "0334744a-f2a2-4aba-8c8a-6e748f62a172",
  "email": "d.l.solove@gmail.com",
  "app_metadata": {
    "provider": "email"
  },
  "user_metadata": null,
  "role": "authenticated"
}

```

이 중에서 일단 sub, email을 활용해보려고 합니다. 

또한 HS256 알고리즘을 사용하는 것을 확인할 수 있습니다. 

### **테스트 코드 작성**

잘 될지 모르겠지만, 지금 진행하는 프로젝트는 테스트코드를 먼저 작성하는 방향으로 가려고 합니다. 

토큰 검증을 할 것이므로, 예상 시나리오는 

- 올바른 토큰

- 토큰이 없는 경우 

- 토큰이 잘못된 경우

- 토큰이 만료된 경우

의 네 가지입니다. 해당 케이스의 테스트 코드를 작성합니다. 

```python
from datetime import datetime, timedelta, timezone

import jwt
from django.conf import settings
from django.test import RequestFactory, TestCase
from rest_framework.exceptions import AuthenticationFailed

from project.permission import IsAuthenticatedWithJWT


class TestIsAuthenticatedWithJWT(TestCase):
    def setUp(self):
        self.factory = RequestFactory()
        self.secret = settings.SUPABASE_JWT_SECRET

    def test_has_permission_with_valid_token(self):
        request = self.factory.get("/")
        token = jwt.encode(
            {
                "sub": "user123",
                "email": "user@example.com",
            },
            self.secret,
            algorithm="HS256",
        )
        request.META["HTTP_AUTHORIZATION"] = f"Bearer {token}"
        permission = IsAuthenticatedWithJWT()

        self.assertTrue(permission.has_permission(request, None))
        self.assertEqual(
            request.user,
            {
                "user_id": "user123",
                "email": "user@example.com",
            },
        )

    def test_has_permission_with_invalid_token(self):
        request = self.factory.get("/")
        request.META["HTTP_AUTHORIZATION"] = "Bearer invalidtoken"
        permission = IsAuthenticatedWithJWT()

        with self.assertRaises(AuthenticationFailed):
            permission.has_permission(request, None)

    def test_has_permission_without_token(self):
        request = self.factory.get("/")
        permission = IsAuthenticatedWithJWT()

        with self.assertRaises(AuthenticationFailed):
            permission.has_permission(request, None)

    def test_has_permission_with_expired_token(self):
        request = self.factory.get("/")
        token = jwt.encode(
            {
                "sub": "user123",
                "email": "user@example.com",
                "exp": datetime.now(timezone.utc) - timedelta(seconds=1),
            },
            self.secret,
            algorithm="HS256",
        )
        request.META["HTTP_AUTHORIZATION"] = f"Bearer {token}"
        permission = IsAuthenticatedWithJWT()

        with self.assertRaises(AuthenticationFailed):
            permission.has_permission(request, None)

```

### **Custom Permission 클래스 작성**

DRF에서 JWT를 사용한 인증을 처리하기 위해 커스텀 퍼미션 클래스를 작성합니다. 이 클래스는 요청에 포함된 JWT 토큰을 검증하고, 유효한 경우 사용자 정보를 설정합니다.

```python
# project/permissions.py

import jwt
from django.conf import settings
from rest_framework import permissions
from rest_framework.exceptions import AuthenticationFailed

class IsAuthenticatedWithJWT(permissions.BasePermission):
    def has_permission(self, request, view):
        auth_header = request.headers.get("Authorization")
        if not auth_header:
            raise AuthenticationFailed("Authorization header missing")

        try:
            token = auth_header.split(" ")[1]
            payload = jwt.decode(token, settings.SUPABASE_JWT_SECRET, algorithms=["HS256"])
            request.user = {
                "user_id": payload.get("sub"),
                "email": payload.get("email"),
            }
        except jwt.ExpiredSignatureError:
            raise AuthenticationFailed("Token has expired")
        except jwt.InvalidTokenError:
            raise AuthenticationFailed("Invalid token")

        return True

```

### View에 Permission Class 적용

작성한 커스텀 퍼미션 클래스를 뷰에 적용하여, 해당 뷰에 접근하는 모든 요청이 JWT를 통해 인증되도록 설정합니다.

```python
# views.py

from rest_framework import status
from rest_framework.response import Response
from rest_framework.views import APIView
from drf_yasg.utils import swagger_auto_schema

from .models import UserProfile
from .serializers import UserProfileSerializer
from project.permissions import IsAuthenticatedWithJWT

class UserProfileUpdateView(APIView):
    permission_classes = [IsAuthenticatedWithJWT]

    @swagger_auto_schema(
        request_body=UserProfileSerializer,
        responses={200: UserProfileSerializer, 404: "Not Found", 400: "Bad Request"},
    )
    def put(self, request, *args, **kwargs):
        user_id = request.user["user_id"]
        try:
            user_profile = UserProfile.objects.get(pk=user_id)
        except UserProfile.DoesNotExist:
            return Response(
                {"error": "UserProfile not found"}, status=status.HTTP_404_NOT_FOUND
            )

        serializer = UserProfileSerializer(
            user_profile, data=request.data, partial=True
        )
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_200_OK)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

```



