---
IDX: "NUM-265"
tags:
  - Python
  - Django
  - BackEnd
description: "애플 실리콘에서의 mysqlclient, mysql.h"
series: "Django Upgrade"
update: "2025-07-02T01:22:00.000Z"
date: "2025-06-26"
상태: "Ready"
title: "PyMySQL 소개"
---
![](image1.png)
## **서론**

Apple Silicon 환경에서 Python, 특히 Django 개발 환경을 설정하다 보면 유독 `mysqlclient` 패키지 설치 단계에서 붉은 에러 메시지와 마주치는 경우가 많습니다. (제가 레거시 환경을 다루다보니 많은 것일수도 있습니다. )

```bash
`src/MySQLdb/_mysql.c:29:10: fatal error: 'mysql.h' file not found`
```



경로를 직접 찾거나 C 라이브러리 설치, 환경 변수 설정을 통해 이 문제를 해결할 수도 있지만, 과정이 번거롭고 다른 환경에 배포할 때 또 다른 문제를 낳기도 합니다.

이 글에서는 골치 아픈 컴파일 에러와 싸우는 대신, 순수 파이썬(Pure Python)으로 작성된 `PyMySQL` 라이브러리를 사용하여 이 문제를 5분 안에 빠르게 해결하는 방법을 소개합니다.

## **왜** `mysqlclient`**는 설치가 까다로울까?**

이 에러의 원인을 알면 해결책을 더 잘 이해할 수 있습니다.

1. C-확장 라이브러리: `mysqlclient`는 100% 파이썬 코드가 아닙니다. 빠른 성능을 위해 C언어로 작성된 부분을 포함하고 있어, 설치 시 사용자의 시스템 환경에 맞게 소스 코드를 컴파일하는 과정이 필요합니다.

1. 시스템 의존성: 이 컴파일 과정은 시스템에 MySQL 또는 MariaDB의 C 개발 라이브러리(헤더 파일 `mysql.h` 포함)가 미리 설치되어 있기를 요구합니다.

1. Apple Silicon의 특수성: 애플 실리콘 맥에서는 Homebrew를 통해 라이브러리를 설치해도, 그 경로가 기존 Intel 맥과 다른 `/opt/homebrew` 아래에 위치합니다. 이 때문에 컴파일러가 헤더 파일과 라이브러리를 제대로 찾지 못해 에러가 발생하는 경우가 많습니다.

## **PyMySQL의 등장**

PyMySQL은 이 모든 문제를 한 번에 해결해 주는 대안입니다.

PyMySQL은 C 컴파일 과정이 전혀 필요 없는, 100% 순수 파이썬으로 작성된 MySQL 드라이버입니다.

### **장점과 단점 (Trade-off)**

#### **압도적인 장점**

- **쉬운 설치:** C 컴파일이 없으므로 `pip install pymysql` 한 줄이면 끝납니다. 어떤 OS(macOS, Linux, Windows)나 아키텍처(Intel, Apple Silicon)에서도 동일하게, 의존성 문제 없이 작동합니다.

- **배포 환경 단순화:** Docker 이미지를 만들거나 새로운 서버에 배포할 때, 복잡한 시스템 라이브러리 설치 과정이 사라져 배포 과정이 매우 단순하고 깔끔해집니다.

- **유일한 단점:**

    - C로 최적화된 `mysqlclient`에 비해 아주 약간의 성능 차이가 있을 수 있습니다. 하지만 데이터베이스 통신이 병목의 대부분을 차지하는 일반적인 웹 애플리케이션 환경에서는 이 차이를 체감하기 거의 어렵습니다.

결론적으로, 대부분의 경우 설치 및 배포의 편리함이 미세한 성능 차이를 압도합니다.

## **Django 프로젝트에 PyMySQL 적용하기**

적용 방법은 놀라울 정도로 간단합니다.

### **1. 의존성 변경**

사용하시는 패키지 매니저에 맞게 `mysqlclient`를 제거하고 `pymysql`을 추가합니다.

#### **Poetry 사용 시**

```bash
poetry remove mysqlclient
poetry add pymysql
```

#### **pip 사용 시 (**`requirements.txt`**)**

```plain text
# requirements.txt
# mysqlclient  <-- 이 줄을 삭제하거나 주석 처리
pymysql      # 이 줄을 추가
```

`pip install -r requirements.txt` 를 실행합니다.

### **2. Django에 PyMySQL 인식시키기**

이제 Django가 `mysqlclient` 대신 `PyMySQL`을 사용하도록 알려줘야 합니다. 프로젝트의 메인 `__init__.py` 파일 최상단에 다음 두 줄을 추가하면 됩니다.

보통 `settings.py` 파일과 같은 위치에 있는 `__init__.py` 입니다.

```python
# your_project/__init__.py

import pymysql

# Django가 MySQLdb 모듈을 임포트할 때, 대신 PyMySQL을 사용하도록 설정
pymysql.install_as_MySQLdb()
```



이 코드는 Django의 DB 클라이언트가 기본적으로 찾는 `MySQLdb` 모듈(즉, `mysqlclient`)을 `PyMySQL`이 대신하도록 위장시켜주는 역할을 합니다.

이것으로 모든 설정이 끝났습니다. 이제 `runserver`를 실행하면 Django가 `PyMySQL`을 통해 정상적으로 DB에 연결하는 것을 확인할 수 있습니다.



