---
IDX: "NUM-238"
tags:
  - GraphQL
  - Django
  - Python
description: "GraphQL 고급 활용법"
update: "2025-01-24T01:04:00.000Z"
date: "2025-01-24"
상태: "Ready"
title: "GraphQL과 Django 2"
---
![](image1.png)
## 서론

[지난 시간](https://sharknia.github.io/GraphQL과-Django)에 이어 GraphQL에 대해 좀 더 자세히 알아보는 시간을 가지려고 합니다. 

## GraphQL의 장점과 현실적인 문제

GraphQL은 기존 REST API의 단점을 보완하고, 클라이언트가 원하는 데이터만 요청할 수 있는 유연한 구조를 제공합니다. 하지만 현실적으로 적용할 때는 여러 문제를 고려해야 합니다.

### GraphQL의 이점

#### 유연한 데이터 요청

- REST API는 백엔드에서 정해준 응답 형식을 그대로 받아야 합니다.

- 하지만 GraphQL은 클라이언트가 필요한 데이터만 선택적으로 요청할 수 있어 불필요한 데이터(Over-fetching)를 줄이고, 필요한 데이터를 추가 요청해야 하는 문제(Under-fetching)도 해결할 수 있습니다.

#### 단일 엔드포인트

- REST는 리소스마다 엔드포인트가 다릅니다. (`/users`, `/users/1/posts`, `/users/1/comments` 등)

- GraphQL은 단일 엔드포인트(`/graphql`)에서 다양한 요청을 처리할 수 있어 API 유지보수가 간편합니다.

### 실제 프로젝트에서 GraphQL을 도입할 때 겪는 문제들

#### 쿼리가 복잡해지면 성능 이슈 발생

GraphQL은 유연한 데이터 요청이 가능하지만, 필요한 데이터를 어떻게 가져올지 효율적으로 설계하지 않으면 오히려 성능이 저하될 수 있습니다. 이에 대한 내용은 [이전 블로그 글](https://sharknia.github.io/GraphQL과-Django)에서 다룬바가 있습니다. 

#### 캐싱의 어려움

단일 엔드포인트에서 다양한 요청을 처리하고 응답의 꼴이 정해져 있지 않기 때문에 기본적인 HTTP 캐싱이 어렵고, 별도의 캐싱 전략을 고려해야 합니다.

#### 쿼리 비용 제한 필요

GraphQL은 클라이언트가 원하는 데이터를 자유롭게 요청할 수 있기 때문에, 너무 깊거나 복잡한 쿼리 요청이 들어올 경우 서버 부담이 커질 수 있습니다. 

```graphql
{
  user(id: 1) {
    name
    posts {
      comments {
        author {
          name
          posts {
            comments {
              author {
                ...
              }
            }
          }
        }
      }
    }
  }
}
```

예를 들어 이처럼 너무 깊은 중첩 쿼리는 서버 성능에 심각한 영향을 미칠 수 있습니다. 이를 위해 GraphQL 쿼리의 Depth 제한 및 요청 복잡도 제한(Query Cost Analysis) 설정이 가능합니다. 예를 들어, 5단계 이상 중첩된 요청은 막거나, 특정 요청당 비용을 계산해 제한하는 방식을 사용할 수 있습니다. 

```python
from graphql.validation.rules import QueryDepthLimitRule

class CustomGraphQLView(GraphQLView):
    def get_validation_rules(self):
        return [QueryDepthLimitRule(5)]  # 최대 5단계 중첩 제한
```

이런 식으로 적용하면 GraphQL의 무분별한 깊은 요청을 막아 서버 부하를 줄일 수 있습니다.

### GraphQL을 무조건 적용하면 안 되는 경우

이와 같이 발생할 수 있는 문제 때문에 항상 모든 서비스에 적합한 것은 아닙니다. 다음과 같은 경우에는 REST API 가 더 나을 수 있습니다. 

#### 단순한 API

CRUD API가 단순한 경우, GraphQL의 유연성이 오히려 필요 없을 수 있습니다. 예를 들어 "게시판 서비스"처럼 정형화된 API는 REST API가 더 효율적일 수 있습니다. 

#### 캐싱이 중요한 경우

위에서 설명한 것처럼 캐싱에 있어서는 REST API가 더 유리합니다. 정적 데이터를 많이 제공하는 등 캐싱이 성능에 영향을 크게 미칠 수 있는 서비스라면 REST API가 훨씬 효율적입니다. 

#### **API 소비자가 다양할 경우**

GraphQL은 프론트엔드 개발자가 직접 원하는 데이터를 요청하는 방식이므로, "내부 서비스"에서 사용하기에는 좋지만 공개 API(예: 오픈 API)에서는 REST API가 더 적절할 수 있습니다. 

## GraphQL의 성능 최적화: DataLoader 패턴을 적용하기

[Django ORM 최적화](https://sharknia.github.io/GraphQL과-Django)만으로도 성능이 좋아질 수 있지만, GraphQL 특성상 동일한 요청을 중복 실행하는 문제가 발생할 수 있습니다. 이를 해결하기 위해 DataLoader 패턴을 적용하면 더욱 효율적으로 데이터 조회가 가능합니다.

### DataLoader란?

GraphQL은 resolver마다 개별적으로 데이터베이스를 조회하는데, 이 과정에서 동일한 데이터에 대한 중복 쿼리가 발생할 수 있습니다. DataLoader는 동일한 요청을 한 번에 처리(Batching)하고, 결과를 캐싱하는 패턴을 말합니다. 

### DataLoader 적용 예시 (Django + Graphene)

```python
from promise import Promise
from promise.dataloader import DataLoader
from Django.db.models import Prefetch
from .models import Post

class PostLoader(DataLoader):
    def batch_load_fn(self, user_ids):
        posts = Post.objects.filter(user_id__in=user_ids)
        post_map = {user_id: [] for user_id in user_ids}

        for post in posts:
            post_map[post.user_id].append(post)

        return Promise.resolve([post_map[user_id] for user_id in user_ids])
```

`batch_load_fn()`을 사용해 한 번의 쿼리로 여러 사용자의 `posts` 데이터를 가져오고 있습니다. 

```python
from .dataloaders import PostLoader

class UserType(DjangoObjectType):
    class Meta:
        model = User

    posts = graphene.List(PostType)

    def resolve_posts(self, info):
        loader = PostLoader()
        return loader.load(self.id)  # DataLoader를 통해 batch 처리
```

GraphQL Resolver에서 DataLoader를 사용해 각 사용자별 개별 쿼리 실행 없이 한 번의 쿼리로 처리할 수 있습니다. 

DataLoader 적용 전에는 

```sql
SELECT * FROM user;
SELECT * FROM post WHERE user_id = 1;
SELECT * FROM post WHERE user_id = 2;
SELECT * FROM post WHERE user_id = 3;
...
```

이와 같이 쿼리가 발생해 사용자가 많을수록 쿼리 수 증가하지만 적용 후에는

```sql
SELECT * FROM user;
SELECT * FROM post WHERE user_id IN (1, 2, 3, ...);
```

쿼리 실행 횟수가 감소해 성능이 최적화됩니다. 

### IN 절 개수 제한을 고려한 DataLoader 최적화

다만 이 방법도 만능은 아닙니다. Oracle을 포함한 일부 DBMS에서는 IN 절에 들어갈 수 있는 값의 개수가 제한되어 있습니다. 예를 들어 Oracle은 1000개 제한이 걸려있으며, 사실상 제한이 없는 DBMS라고 하더라도 너무나 긴 IN 절은 성능 저하를 유발할 수 있습니다. 이에 IN 절이 너무 커지지 않도록 방지하는 로직을 추가하는 것이 중요합니다. 

```python
from promise import Promise
from promise.dataloader import DataLoader
from Django.db.models import Prefetch
from .models import Post

# DBMS IN 절 제한 (Oracle 기준 1000개)
BATCH_SIZE = 1000  

class PostLoader(DataLoader):
    def batch_load_fn(self, user_ids):
        results = {}

        # IN 절 개수 제한을 고려한 배치 처리
        for i in range(0, len(user_ids), BATCH_SIZE):
            batch_ids = user_ids[i : i + BATCH_SIZE]
            posts = Post.objects.filter(user_id__in=batch_ids)

            for post in posts:
                results.setdefault(post.user_id, []).append(post)

        # DataLoader는 user_id 순서대로 반환해야 함
        return Promise.resolve([results.get(user_id, []) for user_id in user_ids])

```

이렇게 작성하면

```sql
SELECT * FROM post WHERE user_id IN (1, 2, ..., 1000);
SELECT * FROM post WHERE user_id IN (1001, 1002, ..., 2000);
...

```

쿼리가 1000개씩 나눠서 실행됩니다. 

## GraphQL API 설계: 유지보수 가능한 스키마 설계 원칙

### Pagination(페이지네이션) 패턴 (Offset vs Cursor)

GraphQL에서 대량의 데이터를 처리할 때는 **Pagination(페이지네이션)**이 필수입니다. 페이지네이션을 구현하는 방법에는 Offset 방식과 Cursor 방식이 있습니다.

#### Offset Pagination (기본적인 방식)

`OFFSET`을 이용해 페이지를 이동하는 방식으로 SQL의 LIMIT + OFFSET을 그대로 사용 가능합니다. 

```graphql
query {
  users(limit: 10, offset: 20) {
    id
    name
  }
}
```

Django에서는 아래와 같이 구현 할 수 있습니다. 

```python
class Query(graphene.ObjectType):
    users = graphene.List(UserType, limit=graphene.Int(), offset=graphene.Int())

    def resolve_users(self, info, limit=10, offset=0):
        return User.objects.all()[offset: offset + limit]  # OFFSET 적용
```

이 방법은 구현이 SQL의 OFFESET/LIMIT을 활용하므로 직관적으로 이해가 쉽고 구현이 간단합니다. 다만 데이터가 많아질수록 성능 저하가 일어날 수 있고, 새로운 데이터가 추가되면 데이터가 밀려서 중복 조회가 발생할 수 있습니다. 

#### Cursor Pagination (Relay-style)

Relay는 GraphQL에서 Cursor 기반 Pagination을 공식적으로 정의한 방식입니다. Cursor Pagination은 특정 필드를 기준으로 다음 데이터를 가져오는 방식으로 OFFSET이 아니라 고유한 Cursor값을 사용합니다. 페이징이 빠르고 안정적으로 Offset 방식의 단점을 개선할 수 있지만 구현이 상대적으로 복잡합니다. 

```graphql
query {
  users(first: 10, after: "cursor123") {
    edges {
      node {
        id
        name
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

Django에서는 아래와 같이 구현 가능합니다. 

```python
import graphene
from graphene_Django.types import DjangoObjectType
from graphene.relay import Node
from graphene_Django.filter import DjangoFilterConnectionField
from .models import User

class UserNode(DjangoObjectType):
    class Meta:
        model = User
        interfaces = (Node,)  # Relay Node 사용

class Query(graphene.ObjectType):
    users = DjangoFilterConnectionField(UserNode)  # Relay-style Pagination 적용
```

이 방식은 대량 데이터에서도 성능이 뛰어나고 데이터가 추가/삭제되어도 안전하지만, 정렬 기준이 필요한 필드가 있어야 하는 문제점이 있습니다.  정식 GraphQL 표준 방식으로 Cursor 기반이라 성능 최적화가 가능하면서도 `pageInfo.hasNextPage` 등을 제공하여 클라이언트에서 쉽게 처리 가능합니다. 

### GraphQL API 변경 관리(버전 관리 vs Deprecated 필드)

GraphQL API는 REST처럼 v1, v2 버전을 사용하지 않는 것이 일반적입니다. 대신 Deprecated 필드를 활용하여 API 변경을 최소화하는 방식을 추천합니다.

```graphql
type User {
  id: ID!
  name: String!
  email: String @deprecated(reason: "Use contactEmail instead")
  contactEmail: String
}
```

기존의 `email` 필드는 유지하면서, 새로운 `contactEmail` 필드를 추가했습니다. 클라이언트가 점진적으로 변경할 수 있도록 유도할 수 있습니다. 

Django에서는 다음과 같이 구현이 가능합니다. 

```python
class UserType(DjangoObjectType):
    class Meta:
        model = User

    email = graphene.String(deprecation_reason="Use contactEmail instead")
```

하지만 대규모 변경이 필요한 경우, 특정 API 그룹만 버전 관리를 하는 것이 유리할 수도 있습니다. 상황에 따라 유연한 대처가 필수입니다. 

## GraphQL과 캐싱

### HTTP 캐싱이 어려운 이유

GraphQL은 REST API보다 강력한 데이터 요청 기능을 제공하지만, 기본적인 HTTP 캐싱이 어렵다는 단점이 있습니다. 단일 엔드포인트를 사용하므로 REST API처럼 URL 기반의 캐싱이 불가능하기 때문입니다. 

REST API의 경우에는 URL이 정해져 있기 때문에, 브라우저나 CDN에서 `Cache-Control`, `ETag` 등을 활용하여 응답을 캐싱할 수 있습니다. 

```plain text
GET /users/1  → 캐싱 가능
GET /posts/10 → 캐싱 가능
```

GraphQL에서는 모든 요청이 같은 엔드포인트(`/graphql`)에서 처리되므로, URL 기반의 HTTP 캐싱이 불가능합니다.

```plain text
POST /graphql   → 캐싱 불가능
```

모든 요청이 POST method로 이루어지며, 요청 본문(Body)에 있는 GraphQL Query 내용이 다를 수 있기 때문입니다.

이 문제를 해결하기 위해 다음의 방법들을 활용해 캐싱할 수 있습니다. 

### GraphQL의 `Persisted Queries` 활용법

#### Persisted Queries란?

GraphQL 쿼리를 사전에 저장해두고, 해시(hash) 값을 이용해 요청하는 방식입니다. 즉, 쿼리 내용을 생략하고 해시값만 보내므로, 동일한 요청을 쉽게 캐싱할 수 있습니다. 

```graphql
query {
  user(id: 1) {
    name
    email
  }
}
```

이 쿼리를 아래처럼 쿼리 해시값을 사용한 요청으로 변환합니다.

```json
{
  "id": "e3cbbd2f0f5e4b80b4",
  "variables": { "id": 1 }
}
```

해시 값이 동일하면 캐싱이 가능해집니다. 클라이언트와 서버에서 동일한 해시값을 가진 요청을 캐싱할 수 있습니다. 

#### Django의 Persisted Queries

클라이언트에서 Query 해시값 생성을 생성합니다. 클라이언트는 Apollo Client등을 활용해 해시값을 생성할 수 있습니다. Django에서는 해시값 기반으로 응답을 캐싱합니다. 

```python
from django.core.cache import cache
from graphql.execution.executors.sync import SyncExecutor

class CachedGraphQLView(GraphQLView):
    def execute_graphql_request(self, request, data, query, *args, **kwargs):
        cache_key = f"graphql:{hash(query)}"
        response = cache.get(cache_key)

        if response is None:
            response = super().execute_graphql_request(request, data, query, *args, **kwargs)
            cache.set(cache_key, response, timeout=3600)  # 1시간 캐싱

        return response
```

쿼리를 해시값으로 변환하여 동일한 요청을 캐싱 가능합니다. 

### Apollo Client의 캐싱 전략

GraphQL에서 클라이언트 캐싱(Apollo Client) 을 활용하면 프론트엔드에서 불필요한 네트워크 요청을 줄일 수 있습니다. Apollo Client의 캐싱을 활용하면, 동일한 요청을 여러 번 보낼 필요 없이 프론트엔드에서 데이터를 재사용할 수 있습니다. 

### Redis를 활용한 GraphQL 응답 캐싱

Persisted Queries와 같은 개념으로 결국 같은 쿼리에 대해 동일한 응답을 반복해서 제공할 수 있기 때문에, Redis를 활용하는 방법으로도 캐싱을 해 성능 향상을 꾀할 수 있습니다. 

#### Django + Redis 기반 GraphQL 캐싱 구현

```python
import redis
import hashlib
from django.core.cache import cache
from graphene_django.views import GraphQLView

redis_client = redis.StrictRedis(host="localhost", port=6379, db=0)

class CachedGraphQLView(GraphQLView):
    def execute_graphql_request(self, request, data, query, *args, **kwargs):
        query_hash = hashlib.sha256(query.encode()).hexdigest()
        cache_key = f"graphql_cache:{query_hash}"
        
        # Redis에서 캐시 확인
        cached_response = redis_client.get(cache_key)
        if cached_response:
            return cached_response

        # 캐시 없으면 원래 GraphQL 요청 실행
        response = super().execute_graphql_request(request, data, query, *args, **kwargs)
        
        # Redis에 캐싱 (60분 유지)
        redis_client.setex(cache_key, 3600, response)

        return response
```

이제 동일한 GraphQL 요청이 오면, Django가 데이터베이스가 아니라 Redis에서 즉시 응답하는 것이 가능합니다. 

