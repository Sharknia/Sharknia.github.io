---
IDX: "NUM-224"
tags:
  - Python
  - Docker
description: "load_dotenv와 Docker 환경 변수 충돌 해결"
update: "2024-12-02T15:35:00.000Z"
date: "2024-11-29"
상태: "Ready"
title: "Docker에서 Python 환경 변수 관리"
---
## 문제 상황

최근 Python 프로젝트에서 Docker를 사용해 컨테이너를 실행하면서 환경 변수가 제대로 반영되지 않는 문제를 겪었습니다. Docker의 `--env-file` 옵션을 사용해 환경 변수를 전달했지만, 코드 실행 결과는 Docker가 전달한 값이 아닌 로컬 `.env` 파일의 값이 사용되는 이상한 상황이 발생했습니다.

```shell
docker run -d --env-file .env --name my_app my_image
```

Docker Run에서 사용한 .env 파일에서 `KEYWORD=스타일러`를 설정했지만, 프로그램은 여전히 `KEYWORD=콜라`를 사용하고 있었습니다.

## 원인 분석

Python에서 환경 변수를 관리하기 위해 `python-dotenv` 라이브러리의 `load_dotenv`를 사용하고 있었습니다. 이 라이브러리는 `.env` 파일의 내용을 Python 환경 변수로 로드해주는 편리한 도구입니다.

하지만, Docker와 `load_dotenv`를 함께 사용할 때 충돌이 발생했습니다.

### 원인의 핵심

- Docker 환경 변수: `-env-file` 옵션으로 전달된 값은 컨테이너 실행 시 시스템 환경 변수로 설정됩니다.

- `load_dotenv`의 동작:

    - `.env` 파일의 내용을 Python 환경 변수로 설정.

    - 기본적으로 이미 존재하는 환경 변수는 덮어쓰지 않음.

    - 하지만 코드가 `.env` 파일을 명시적으로 읽어, Docker에서 설정된 환경 변수를 덮어쓰는 문제가 발생.

이로 인해 프로그램은 Docker가 전달한 `KEYWORD=스타일러`가 아닌 `.env` 파일의 `KEYWORD=콜라`를 사용했습니다.

## 해결 방법

### `load_dotenv` 제거

`load_dotenv`는 로컬 `.env` 파일을 로드하기 위한 도구로, Docker와 함께 사용할 때는 불필요합니다. Docker는 `--env-file`로 전달한 환경 변수를 컨테이너 내부에 자동으로 설정하므로, Python 코드에서는 `os.getenv`만 사용하면 됩니다.

```python
import os

# 환경 변수 읽기
keyword = os.getenv("KEYWORD", "default_value")
print(f"Keyword: {keyword}")
```

이렇게 `load_dotenv`를 제거하니, Docker에서 전달한 환경 변수(`KEYWORD=스타일러`)가 정상적으로 동작했습니다.

<hr style="border: none; height: 1px; background-color: #e0e0e0; margin: 16px 0;" />
### Docker에서 환경 변수 확인

Docker 컨테이너가 환경 변수를 올바르게 전달했는지 확인하려면 아래 명령어를 사용합니다.

```shell
docker exec -it my_app env | grep KEYWORD
```

결과:

```plain text
KEYWORD=스타일러
```

이 명령어를 통해 컨테이너 내부에서 설정된 환경 변수를 확인할 수 있습니다.

## 로컬에서는..?

로컬에서 실행할 때는 `.env` 파일을 사용해야 합니다. 

```python
from dotenv import load_dotenv
import os

load_dotenv()
keyword = os.getenv("KEYWORD", "default_value")
print(f"Keyword: {keyword}")
```

## 교훈

1. Docker와 Python 환경 변수 관리의 중복은 주의해야 합니다:

    - Docker에서 환경 변수를 관리한다면, Python 코드에서 추가로 `.env` 파일을 읽지 않아야 합니다.

1. 환경 변수 우선순위를 명확히 해야 합니다:

    - Docker > 시스템 환경 변수 > `.env` 파일 순으로 환경 변수가 설정되도록 설계하면 혼란을 줄일 수 있습니다.

