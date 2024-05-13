---
IDX: "NUM-179"
tags:
  - Django
  - Docker
  - Docker-compose
  - Postgresql
description: "docker-compose를 활용한 postgresql과 django 커넥션 만들기"
update: "2024-05-13T07:10:00.000Z"
date: "2024-05-11"
상태: "Ready"
title: "docker-compose를 활용한 postgresql과 django 커넥션 만들기"
---
![](image1.png)
[이전 포스팅](https://sharknia.github.io/Django-설치)에서 간단하게 django 프로젝트를 설치하고 구동해봤습니다. 이번 시간에는 postgresql과 django 프로젝트를 연결해보려고 합니다. 

## .env 파일 작성

DB 설정을 위한 환경변수 파일을 작성합니다. 

```shell
POSTGRES_DB=
POSTGRES_USER=
POSTGRES_PASSWORD=
DATABASE_URL=postgres://[username]:[password]@[db-url]:[port]/[db-name]
```

해당 파일을 docker-compose.yml 파일과 같은 곳에 위치시킵니다. 

이 파일은 보안에 유의하여, git과 같은 곳에 올라가지 않도록 주의합니다. 

## Makefile 작성

원활한 테스트를 위해 Makefile을 미리 작성합니다. 

```makefile
up:
	docker-compose -f docker/docker-compose.yml up -d

down:
	docker-compose -f docker/docker-compose.yml down

ps:
	docker-compose -f docker/docker-compose.yml ps

log: 
	docker-compose -f docker/docker-compose.yml logs

```

이제 `make up`, `make down` 명령어를 통해 간단하게 컨테이너를 재시작 할 수 있습니다. 

## **docker-compose.yml 파일 수정**

해당 파일을 다음과 같이 수정합니다. 

```yaml
version: '3.8'

services:
    db:
        image: postgres
        volumes:
            - postgres_data:/var/lib/postgresql/data
        environment:
            POSTGRES_DB: ${POSTGRES_DB}
            POSTGRES_USER: ${POSTGRES_USER}
            POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
        ports:
            - '5432:5432'
    web:
        build:
            context: ..
            dockerfile: docker/Dockerfile
        command: gunicorn project.wsgi:application --bind 0.0.0.0:8000 --reload
        volumes:
            - ../:/app
        ports:
            - '8000:8000'
        depends_on:
            - db
        environment:
            DATABASE_URL: ${DATABASE_URL}
            POSTGRES_DB: ${POSTGRES_DB}
            POSTGRES_USER: ${POSTGRES_USER}
            POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

volumes:
    postgres_data:

```

port는 원하는대로 수정하면 됩니다. 

## dj-database-url 설치

`dj-database-url`는 Django 프로젝트에서 데이터베이스 설정을 간편하게 관리할 수 있도록 도와주는 파이썬 라이브러리입니다. 이 라이브러리를 사용하면, 표준 데이터베이스 연결 URL을 파싱하여 Django의 데이터베이스 설정 형식으로 변환할 수 있습니다. 즉, `DATABASE_URL` 환경 변수에서 데이터베이스 연결 문자열을 읽어 Django에서 사용할 수 있게 설정하는 역할을 합니다. 

간단한 연결을 위해 해당 라이브러리를 설치하겠습니다. 

```shell
poetry shell
poetry add dj-database-url
```

이렇게 설치를 한 후, 이미지를 다시 빌드합니다. 

## Setting.py 수정

DATABASE 부분을 다음과 같이 수정합니다. 

```python
import dj_database_url

DATABASES = {"default": dj_database_url.config()}
```

`DATABASE_URL` 환경변수를 사용하므로, 반드시 해당 환경변수가 설정되어 있어야 합니다. 

## 연결 확인

DB 연결이 잘 됐는지 테스트 해볼 수 있습니다. 

다음의 명령어를 사용해 파이썬 Shell을 실행하고, 

```shell
docker-compose -f docker/docker-compose.yml run web python manage.py shell
```

다음의 파이썬 코드를 입력합니다. 오류가 없다면 연결이 잘 된 것입니다. 

```python
from django.db import connection
cursor = connection.cursor()
```

psql을 이용해 DB에 직접 연결해 DB를 테스트 해볼수도 있습니다. 

```bash
psql -h [url] -U [username] -d [dbname] -p 5432
```



