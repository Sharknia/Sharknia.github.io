---
tags:
  - DataBase
  - AWS
  - Python
  - Work
update: "2024-01-31"
date: "2023-12-08"
상태: "Ready"
title: "PynamoDB와 boto3, PynamoDB의 커넥션"
---
회사에서 DynamoDB를 사용할 때에, PynamoDB ORM을 사용하여 연결을 하고 있다. 

Postgresql을 도입했을 때에 커넥션 문제가 지속적으로 발생해 (연결이.. 끊어지지 않는다!) 이를 처리하던 도중 PynamoDB는 어떻게 커넥션을 관리하는지 궁금해서 찾아보게 되었다. 

### PynamoDB란? 

PynamoDB는 DynamoDB를 위한 Python 라이브러리이다. 

#### Pythonic Interface

파이썬 개발자들이 익숙한 객체 지향 접근 방식을 제공한다. 

#### 간편한 데이터 모델링

PynamoDB를 사용하면 DynamoDB 테이블과 항목을 Python 클래스로 모델링 할 수 있다. 

#### CRUD Operation

PynamoDB는 Create, Read, Update, Delete 작업을 간편하게 수행할 수 있는 메소드를 제공한다. 

### PynamoDB의 커넥션 관리

PynamoDB에서 커넥션은 boto3를 사용해서 관리하며, Low Level의 API를 사용하면 커넥션을 직접 연결할 수 있지만 기본적으로는 자동으로 관리를 한다. 설정 몇가지만 해주면(또는 하지 않아도) 필요에 따라서 알아서 커넥션을 맺고 끊는다. 

[https://pynamodb.readthedocs.io/en/stable/settings.html](https://pynamodb.readthedocs.io/en/stable/settings.html)

### boto3란? 

AWS를 위한 AWS 공식 Python SDK이다. 이 라이브러리를 통해 Python 어플리케이션에서 AWS를 쉽게 사용 및 관리 할 수 있다. 

#### 광범위한 AWS 서비스 지원

S3, EC2, DynamoDB, IAM 등 AWS에서 제공하는 대부분의 서비스를 지원한다. 

#### 높은 수준의 객체 지향 API와 low level API 제공

`Resource API`는 높은 수준의 객체 지향 인터페이스를 제공하여 사용자가 AWS 리소스를 직관적으로 다룰 수 있게 해준다. 

`Client API` 는 낮은 수준, 서비스 중심의 인터페이스를 제공하여 더 세밀한 제어가 필요할 때 사용할 수 있다. 

#### 자격 증명 관리와 보안

AWS의 자격 증명을 효과적으로 관리할 수 잇는 기능을 제공한다.    

