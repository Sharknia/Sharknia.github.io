---
IDX: "NUM-195"
tags:
  - Django
  - Python
description: "AbstractUser와 AbstractBaseUser "
update: "2024-06-05T05:53:00.000Z"
date: "2024-06-03"
상태: "Ready"
title: "Django의 사용자 인증 시스템 커스터마이징"
---
![](image1.png)
## 서론

Django에서 사용자 인증 시스템을 커스터마이징해야 하는 경우가 있습니다. 이때 Django는 두 가지 주요 클래스(`AbstractUser`와 `AbstractBaseUser`)를 제공하여 커스텀 유저 모델을 쉽게 만들 수 있게 해줍니다.  이 글에서는 이 두 클래스의 차이점과 각각의 사용 예시를 소개하겠습니다.

## Django 사용자 인증 시스템

### 기본 모델과 필드

Django는 기본적으로 `User` 모델을 제공하여 사용자 정보를 저장하고 관리합니다. `User` 모델에는 다음과 같은 필드가 포함되어 있습니다:

- `username`: 사용자 이름 (필수)

- `password`: 암호화된 비밀번호 (필수)

- `email`: 이메일 주소 (선택)

- `first_name`: 이름 (선택)

- `last_name`: 성 (선택)

- 기타 인증 및 권한 관련 필드 (`is_staff`, `is_superuser`, `is_active` 등)

### 인증 뷰와 URL

Django는 사용자 인증과 관련된 기본 뷰를 제공합니다. 이 뷰들은 `django.contrib.auth.views` 모듈에 포함되어 있으며, 다음과 같은 뷰들이 있습니다:

- `LoginView`: 사용자가 로그인할 수 있는 뷰

- `LogoutView`: 사용자가 로그아웃할 수 있는 뷰

- `PasswordChangeView`: 사용자가 비밀번호를 변경할 수 있는 뷰

- `PasswordResetView`: 사용자가 비밀번호를 재설정할 수 있는 뷰

이 뷰들은 설정된 URL에 매핑하여 사용할 수 있습니다.

### Form Class

Django는 사용자 인증과 관련된 여러 폼 클래스를 제공하여 사용자 입력을 처리합니다. 대표적인 폼 클래스로는 다음이 있습니다:

- `UserCreationForm`: 새로운 사용자를 생성하는 폼

- `AuthenticationForm`: 사용자가 로그인할 때 사용하는 폼

- `PasswordChangeForm`: 사용자가 비밀번호를 변경할 때 사용하는 폼

- `PasswordResetForm`: 사용자가 비밀번호를 재설정할 때 사용하는 폼

## AbstractUser

`AbstractUser`는 Django의 기본 User 모델을 확장하는 가장 간단한 방법입니다. 이 클래스는 기본 User 모델의 모든 필드와 메서드를 상속하므로, 추가적인 필드를 정의하거나 메서드를 추가하는 데 적합합니다.

### 장점

- 기본 필드 제공: `AbstractUser`는 Django의 기본 User 모델에서 제공하는 모든 필드를 포함합니다. 이를 통해 사용자 이름, 이메일, 비밀번호, 이름 및 성과 같은 필드를 쉽게 사용할 수 있습니다.

- 편리한 확장성: 기존 필드를 유지하면서 필요한 추가 필드만 확장할 수 있어, 대부분의 기본적인 사용자 정보 요구 사항을 충족할 수 있습니다.

- 간단한 설정: 커스터마이징이 쉬우며, 기본적인 인증 기능을 사용하기 위한 추가적인 설정이 거의 필요하지 않습니다.

### 단점

- 제한된 유연성: 기본 User 모델의 필드와 구조를 유지해야 하므로, 사용자 모델을 처음부터 완전히 재정의하고자 할 때는 적합하지 않습니다.

- 불필요한 필드: 모든 기본 필드가 포함되어 있으므로, 불필요한 필드가 있을 수 있습니다. 예를 들어, 이메일을 기본 로그인 필드로 사용하고 싶을 때 사용자 이름 필드가 불필요하게 포함될 수 있습니다.

### 예제

```python
# models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    date_of_birth = models.DateField(null=True, blank=True)
    profile_picture = models.ImageField(upload_to='profiles/', null=True, blank=True)

# settings.py
AUTH_USER_MODEL = 'yourapp.CustomUser'
```

## AbstractBaseUser

`AbstractBaseUser`는 사용자 모델을 처음부터 완전히 커스터마이징하고자 할 때 사용합니다. 이 클래스는 인증에 필요한 기본 필드와 메서드만 제공하며, 나머지 필드는 직접 정의해야 합니다.

### 장점

- 완전한 커스터마이징: `AbstractBaseUser`를 사용하면 사용자 모델의 모든 필드를 처음부터 완전히 정의할 수 있습니다. 이를 통해 로그인 필드, 필수 필드 등을 자유롭게 설정할 수 있습니다.

- 유연성: 사용자 모델의 구조와 필드를 자유롭게 설정할 수 있어, 복잡한 요구 사항이나 특정 비즈니스 로직을 구현하는 데 매우 유용합니다.

- 효율성: 필요한 필드만 포함시킬 수 있으므로, 모델의 간결성과 효율성을 유지할 수 있습니다.

### 단점

- 복잡한 설정: 모든 필드와 매니저를 직접 정의해야 하므로, 설정 과정이 복잡하고 시간이 많이 걸릴 수 있습니다. 특히, 커스텀 사용자 매니저를 작성해야 합니다.

- 높은 진입 장벽: 초보자에게는 설정과 구현이 어려울 수 있으며, 기본 User 모델을 사용하는 것보다 더 많은 학습이 필요합니다.

### 예제

```python
# models.py
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager
from django.db import models

class CustomUserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        if not email:
            raise ValueError('The Email field must be set')
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)

        return self.create_user(email, password, **extra_fields)

class CustomUser(AbstractBaseUser):
    email = models.EmailField(unique=True)
    first_name = models.CharField(max_length=30, blank=True)
    last_name = models.CharField(max_length=30, blank=True)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)
    is_superuser = models.BooleanField(default=False)

    objects = CustomUserManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = []

    def __str__(self):
        return self.email

# settings.py
AUTH_USER_MODEL = 'yourapp.CustomUser'
```



