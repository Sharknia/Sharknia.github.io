---
IDX: "NUM-234"
tags:
  - Python
  - Django
  - GraphQL
description: "GraphQL과 Django"
update: "2025-01-21T00:45:00.000Z"
date: "2025-01-20"
상태: "Ready"
title: "GraphQL과 Django"
---
![](image1.png)
## GraphQL의 특징

### 클라이언트 주도형 데이터 요청

REST API는 백엔드가 정해준 엔드포인트를 그대로 사용해야 하지만, GraphQL은 클라이언트가 원하는 데이터만 요청할 수 있습니다. 

#### REST API 요청 예시 (불필요한 데이터 포함)

```plain text
GET /users/1
```

```json
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com",
  "phone": "123-456-7890"
}
```

#### GraphQL 요청 예시 (원하는 데이터만 선택)

```plain text
{
  user(id: 1) {
    name
    email
  }
}
```

```json
{
  "data": {
    "user": {
      "name": "Alice",
      "email": "alice@example.com"
    }
  }
}
```

REST API는 필요하지 않은 데이터까지 포함되지만, GraphQL은 원하는 필드만 반환합니다. 

### 하나의 엔드포인트로 모든 요청 처리

REST API는 엔드포인트가 여러 개(GET /users, GET /users/1/posts 등) 필요하지만, GraphQL은 단 하나의 엔드포인트(/graphql)만 사용하여 모든 요청을 처리할 수 있습니다.

GraphQL에서는 Mutation이 데이터 변경(Create, Update, Delete) 작업이지만, HTTP 요청 방식은 기본적으로 POST를 사용합니다. 왜냐하면 요청 본문(Body)에 쿼리를 포함해야 하기 때문입니다. 

### 타입 시스템 & 스키마 정의

GraphQL은 스키마 기반으로 동작하며, 데이터의 타입을 명확하게 정의합니다.

```plain text
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}
type Post {
  id: ID!
  title: String!
  content: String!
}
type Query {
  user(id: ID!): User
  posts: [Post]
}
```

### 정리

GraphQL은 원하는 데이터만 요청이 가능하고, 하나의 엔드포인트로 모든 요청을 처리하며 강력한 타입 시스템으로 별도의 문서화가 필요 없어 프론트엔드에 최적화되어 있습니다. 

다만 쿼리가 복잡해질 경우 성능 이슈가 발생할 수 있음에 유의해야 합니다. 

## Django의 GraphQL

Django에서 GraphQL을 사용하려면 보통 `graphene-django` 라이브러리를 활용합니다.

### 설치

다음의 명령어를 통해 설치합니다. 

```bash
pip install graphene-django
```

그리고 Django 프로젝트의 `settings.py`에 GraphQL를 추가합니다. 

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "graphene_django",  # GraphQL 추가
    "myapp",
]

GRAPHENE = {
    "SCHEMA": "myapp.schema.schema"  # GraphQL 스키마 경로 지정
}
```



### `graphene.ObjectType`

Django에서 GraphQL을 사용할 때, `graphene.ObjectType`을 상속받아 **GraphQL에서 응답 데이터의 구조를 정의**합니다.

```python
import graphene

class UserType(graphene.ObjectType):
    id = graphene.ID()
    name = graphene.String()
    email = graphene.String()
```

- GraphQL에서 `UserType`을 정의.

- `id`, `name`, `email`이 응답에 포함될 필드임을 지정.

즉, GraphQL API에서 클라이언트가 어떤 데이터를 받을 수 있는지 정의하는 것입니다.

### `DjangoObjectType`

Django 모델을 GraphQL 타입으로 변환할 때는 `graphene_django.types.DjangoObjectType`을 사용합니다.

```python
from graphene_django.types import DjangoObjectType
from .models import User

class UserType(DjangoObjectType):
    class Meta:
        model = User  # Django 모델과 자동 매핑
```

- `User` 모델과 자동으로 연결됨.

- Django 모델 필드를 자동으로 GraphQL 필드로 변환 (`CharField → String`, `IntegerField → Int`  등).

### GraphQL과 CSRF

#### CSRF 보호란?

Django는 CSRF(Cross-Site Request Forgery) 공격 방지를 위해 POST 요청을 보호합니다. 하지만 GraphQL은 기본적으로 POST 요청을 사용하기 때문에 CSRF 보호 정책과 충돌할 수 있습니다. 따라서 `csrf_exempt`를 사용하면 Django의 CSRF 미들웨어가 해당 뷰를 검사하지 않도록 설정할 수 있습니다. 

GraphQL은 일반적으로 REST API처럼 클라이언트가 직접 요청을 보내는 방식입니다. 즉, 웹 브라우저에서 실행되는 폼(form) 기반 요청이 아니므로 CSRF 토큰을 사용할 필요가 없음.

#### 보안 강화를 위한 대체 방법

`csrf_exempt` 대신 `@csrf_protect`를 사용하는 방법이 있습니다. 

```python
from django.views.decorators.csrf import csrf_protect

urlpatterns = [
    path("graphql/", csrf_protect(GraphQLView.as_view(graphiql=True))),  # CSRF 보호 적용
]
```

이렇게 하면 CSRF 보호를 유지하면서도 GraphQL API를 안전하게 사용할 수 있습니다. 



또, JWT 또는 Token 기반 인증을 사용할 수 있습니다. 

GraphQL API는 보통 JWT(Json Web Token) 또는 OAuth 토큰을 사용하여 인증합니다. 이렇게 하면 CSRF 보호 없이도 API 요청을 안전하게 관리할 수 있습니다.

```python
from graphql_jwt.decorators import login_required
import graphene

class Query(graphene.ObjectType):
    user = graphene.Field(UserType)

    @login_required
    def resolve_user(self, info):
        return info.context.user
```

CSRF 공격은 "브라우저가 자동으로 요청을 보내는 방식"을 악용하는 공격입니다. 그러나 JWT 인증 방식은 CSRF 보호가 필요 없는 구조입니다. 즉, GraphQL + JWT 인증을 사용하면 `csrf_exempt` 없이도 보안 문제를 예방할 수 있습니다. 

### Query

GraphQL에서 데이터를 조회할 때는 `Query`를 사용합니다. Django에서는 다음과 같이 구현할 수 있습니다. 

```python
import graphene
from graphene_django.types import DjangoObjectType
from .models import User

class UserType(DjangoObjectType):
    class Meta:
        model = User

class Query(graphene.ObjectType):
    user = graphene.Field(UserType, id=graphene.Int())

    def resolve_user(self, info, id):
        return User.objects.get(pk=id)

schema = graphene.Schema(query=Query)
```

### Mutation

GraphQL은 조회뿐만 아니라, 데이터를 추가/수정/삭제할 수도 있습니다. 이를 위해 Mutation을 사용합니다.

요청을 이렇게 보낸다면, 

```plain text
mutation {
  createUser(name: "Alice", email: "alice@example.com") {
    user {
      id
      name
      email
    }
  }
}
```

아래와 같이 구현할 수 있습니다. 

```python
class CreateUser(graphene.Mutation):
    class Arguments:
        name = graphene.String()
        email = graphene.String()

    user = graphene.Field(UserType)

    def mutate(self, info, name, email):
        user = User.objects.create(name=name, email=email)
        return CreateUser(user=user)

class Mutation(graphene.ObjectType):
    create_user = CreateUser.Field()
```

이런식으로 `Update`, `Delete`도 구현이 가능합니다. 

### `resolve_<필드명>` 규칙

GraphQL에서 `Query`나 `Mutation`을 처리할 때, `graphene`은 자동으로 해당 필드를 처리하는 `resolve_<필드명>` 메서드를 찾습니다. 따라서 필드명이 `user`라면 `resolve_user()`라는 메서드를 만들면 자동으로 연결됩니다.

```python
import graphene
from graphene_django.types import DjangoObjectType
from .models import User

class UserType(DjangoObjectType):
    class Meta:
        model = User

class Query(graphene.ObjectType):
    user = graphene.Field(UserType, id=graphene.Int())  # GraphQL 필드 정의

    def resolve_user(self, info, id):  # 필드명과 매칭된 resolver 함수
        return User.objects.get(pk=id)

schema = graphene.Schema(query=Query)
```

이 코드에서 GraphQL의 `user` 필드를 요청하면 `resolve_user()`가 자동 실행됩니다. `resolve_user(self, info, id)`를 만들지 않으면, `user` 필드의 데이터를 가져올 방법이 없기 때문에 오류가 발생합니다. 

`resolve_`를 사용하지 않고, `Field()`에서 직접 `resolver`를 지정할 수도 있지만, 일반적으로는 `resolve_`를 사용하는 것이 가독성이 좋고 유지보수가 쉽습니다. 

```python
class Query(graphene.ObjectType):
    user = graphene.Field(UserType, id=graphene.Int(), resolver=lambda self, info, id: User.objects.get(pk=id))

```

### GraphQL의 성능 최적화 (N+1 문제 해결)

GraphQL은 기본적으로 [N+1 문제](https://sharknia.github.io/N1-문제)가 발생할 가능성이 큽니다. Django ORM에서는 `select_related()`와 `prefetch_related()`를 활용해 최적화할 수 있습니다.

예를 들어, 

```python
class Query(graphene.ObjectType):
    users = graphene.List(UserType)

    def resolve_users(self, info):
        return User.objects.all()  # 각 user마다 profile을 조회하는 추가 쿼리 발생!
```

위와 같이 작성하면 `users`를 조회할 때 각 `profile`을 개별 쿼리로 조회해 N+1 문제가 발생합니다. 

```python
class Query(graphene.ObjectType):
    users = graphene.List(UserType)

    def resolve_users(self, info):
        return User.objects.select_related("profile").all()  # SQL JOIN 사용
```

이렇게 하면 한 번의 SQL 쿼리로 해결 가능합니다. ManyToMany 또는 1:N의 경우에는 `prefetch_related()`를 사용해 최적화 할 수 있습니다. 

## 예고편

다음에는 GraphQL의 캐싱 전략, batch queries, Persisted Queries와 함께 GraphQL의 강점 중 하나인 Subscription(실시간 데이터 스트리밍) 기능에 대해 알아보겠습니다. 또 django의 `graphene-file-upload`를 이용한 파일 업로드 방법이나 `relay-style pagination` 에 대해서도 알아보려고 합니다. 



