---
IDX: "NUM-196"
tags:
  - AWS
  - BackEnd
description: "AWS 활용 테스트 서버 구현 1. AWS Server Infra 구성"
update: "2024-06-09T07:26:00.000Z"
date: "2024-06-09"
상태: "Ready"
title: "AWS 활용 테스트 서버 구현 (1)"
---
![](image1.png)
## 서론

[ngrok](https://sharknia.github.io/ngrok-로컬-서버를-쉽게-공개하는-도구)을 이용하는 등 로컬에서만 개발을 하다 테스트 서버를 꾸며야 하는 때가 왔습니다. AWS를 이용하기로 했고, 그 과정을 진행했던 기록을 남기려고 합니다. 

외부에서는 API Gateway에만 접근이 가능하며, DB나 백엔드 서버는 직접 접근이 불가능하도록 구현하려고 합니다. 

## VPC (Virtual Private Cloud)

### VPC 생성

1. VPC 서비스로 이동: 상단 검색창에서 "VPC"를 검색하여 VPC 대시보드로 이동합니다.

1. VPC 생성:

    - VPC 대시보드에서 "Your VPCs"를 클릭합니다.

    - "Create VPC" 버튼을 클릭합니다.

    - VPC 이름을 입력합니다 (예: `MyVPC`).

    - IPv4 CIDR 블록을 입력합니다 (예: `10.0.0.0/16`). 

    - 기타 옵션은 기본값으로 두고 "Create" 버튼을 클릭합니다.

### IPv4 CIDR 블록이란?

#### CIDR (Classless Inter-Domain Routing)

- CIDR은 IP 주소를 효율적으로 할당하고 라우팅하기 위한 방법입니다.

- CIDR 표기법은 IP 주소 뒤에 슬래시(`/`)와 숫자를 붙여서 사용합니다. 예를 들어, `10.0.0.0/16`입니다.

#### 구성 요소

- 네트워크 주소: `10.0.0.0`

    - 네트워크의 시작 주소를 나타냅니다.

- 서브넷 마스크: `/16`

    - 네트워크와 호스트 부분을 구분하는 비트 수를 나타냅니다.

    - `/16`은 처음 16비트가 네트워크 주소를, 나머지 16비트가 호스트 주소를 의미합니다.

#### 의미

- 10.0.0.0/16은 IP 주소 범위를 나타냅니다.

    - 시작 주소는 `10.0.0.0`입니다.

    - 끝 주소는 `10.0.255.255`입니다.

    - 총 65,536개의 IP 주소를 포함합니다 (2^16).

#### 예시

- 공용 서브넷: `10.0.1.0/24`

    - 시작 주소: `10.0.1.0`

    - 끝 주소: `10.0.1.255`

    - 총 256개의 IP 주소를 포함합니다 (2^8).

    - `/24`는 처음 24비트가 네트워크 주소를, 나머지 8비트가 호스트 주소를 의미합니다.

- 프라이빗 서브넷: `10.0.2.0/24`

    - 시작 주소: `10.0.2.0`

    - 끝 주소: `10.0.2.255`

    - 총 256개의 IP 주소를 포함합니다.

#### 설정 예시

- VPC CIDR 블록: `10.0.0.0/16`

- 공용 서브넷 CIDR 블록: `10.0.1.0/24`

- 프라이빗 서브넷 CIDR 블록: `10.0.2.0/24`

이와 같이 CIDR 블록을 설정하면 VPC 내에서 사용할 수 있는 IP 주소의 범위를 지정하고, 공용 및 프라이빗 서브넷을 나누어 관리할 수 있습니다.

### 서브넷 생성

- VPC 대시보드에서 "Subnets"를 클릭합니다.

- "Create Subnet" 버튼을 클릭합니다.

- 서브넷 이름을 입력합니다 (예: `PublicSubnet1`, `PrivateSubnet1`).

- VPC를 선택합니다 (예: `MyVPC`).

- 서브넷의 가용 영역을 선택합니다.

- 각 서브넷의 IPv4 CIDR 블록을 입력합니다 (예: 공용 서브넷은 `10.0.1.0/24`, 프라이빗 서브넷은 `10.0.2.0/24`).

- "Create Subnet" 버튼을 클릭합니다.

#### 인터넷 게이트웨이 생성 및 연결

1. 인터넷 게이트웨이 생성:

    - VPC 대시보드에서 "Internet Gateways"를 클릭합니다.

    - "Create Internet Gateway" 버튼을 클릭합니다.

    - 이름을 입력합니다 (예: `MyInternetGateway`).

    - "Create" 버튼을 클릭합니다.

1. 인터넷 게이트웨이 연결:

    - 생성한 인터넷 게이트웨이를 선택하고 "Actions"를 클릭한 후 "Attach to VPC"를 선택합니다.

    - VPC를 선택하고 "Attach" 버튼을 클릭합니다.

#### 라우팅 테이블 설정

1. 라우팅 테이블 생성:

    - VPC 대시보드에서 "Route Tables"를 클릭합니다.

    - "Create Route Table" 버튼을 클릭합니다.

    - 이름을 입력합니다 (예: `PublicRouteTable`).

    - VPC를 선택하고 "Create" 버튼을 클릭합니다.

1. 라우팅 테이블 편집:

    - 생성한 라우팅 테이블을 선택하고 "Routes" 탭으로 이동합니다.

    - "Edit Routes" 버튼을 클릭합니다.

    - "Add route"를 클릭하여 다음을 추가합니다:

        - 목적지: `0.0.0.0/0`

        - 타겟: 인터넷 게이트웨이 (예: `MyInternetGateway`)

    - "Save routes" 버튼을 클릭합니다.

1. 서브넷 연결:

    - 라우팅 테이블을 선택하고 "Subnet associations" 탭으로 이동합니다.

    - "Edit subnet associations" 버튼을 클릭합니다.

    - 공용 서브넷을 선택하고 "Save" 버튼을 클릭합니다.

## 보안 그룹 및 네트워크 ACL

### 보안 그룹 생성

1. EC2 대시보드로 이동: 상단 검색창에서 "EC2"를 검색하여 EC2 대시보드로 이동합니다.

1. 보안 그룹 생성:

    - EC2 대시보드에서 "Security Groups" 클릭:

        - "Security Groups"를 클릭합니다.

        - "Create security group" 버튼을 클릭합니다.

    - 보안 그룹 이름 및 설명 입력:

        - 보안 그룹 이름을 입력합니다.

        - 설명을 입력합니다.

        - VPC를 선택합니다 (예: `MyVPC`).

        - 보안 그룹은 서비스 별로 구분하여 만들어주는게 관리하기가 쉽습니다. 

1. 인바운드 규칙 설정:

    - API Gateway 보안 그룹 (APIGatewaySG):

        - Type: HTTP, Protocol: TCP, Port range: 80, Source: 0.0.0.0/0 (모든 IP 허용)

        - Type: HTTPS, Protocol: TCP, Port range: 443, Source: 0.0.0.0/0 (모든 IP 허용)

    - Auth Server 보안 그룹 (TestServerSG):

        - Type: Custom TCP, Protocol: TCP, Port range: 8000 (예제 포트), Source: APIGatewaySG (API Gateway 보안 그룹에서만 접근 허용)

    - DB 보안 그룹 (DBSG):

        - **Type**: PostgreSQL, **Protocol**: TCP, **Port range**: 5432, **Source**: TestServerSG (TestServerSG 보안 그룹에서만 접근 허용)

1. 아웃바운드 규칙 설정:

    - 기본적으로 모든 아웃바운드 트래픽이 허용되지만, 필요에 따라 아웃바운드 규칙을 설정할 수 있습니다.

#### 네트워크 ACL 설정

1. VPC 대시보드로 이동: 상단 검색창에서 "VPC"를 검색하여 VPC 대시보드로 이동합니다.

1. 네트워크 ACL 생성:

    - VPC 대시보드에서 "Network ACLs" 클릭:

        - "Network ACLs"를 클릭합니다.

        - "Create Network ACL" 버튼을 클릭합니다.

    - 네트워크 ACL 이름 및 설명 입력:

        - 네트워크 ACL 이름을 입력합니다 (예: `PublicSubnetACL`, `PrivateSubnetACL`).

        - 설명을 입력합니다.

        - VPC를 선택합니다 (예: `MyVPC`).

    - **서브넷 연결**:

        - 생성한 네트워크 ACL을 선택하고 "Subnet associations" 탭으로 이동합니다.

        - "Edit subnet associations" 버튼을 클릭합니다.

        - 공용 서브넷 및 프라이빗 서브넷을 각각 적절한 네트워크 ACL에 연결합니다.

1. 인바운드 및 아웃바운드 규칙 설정:

    #### Public 서브넷 (PublicSubnetACL)

    - **인바운드 규칙**:

        - Rule #100: `Allow`, Protocol: `TCP`, Port range: `80`, Source: `0.0.0.0/0` (모든 IP)

        - Rule #110: `Allow`, Protocol: `TCP`, Port range: `443`, Source: `0.0.0.0/0` (모든 IP)

    - **아웃바운드 규칙**:

        - Rule #100: `Allow`, Protocol: `TCP`, Port range: `80`, Destination: `0.0.0.0/0`

        - Rule #110: `Allow`, Protocol: `TCP`, Port range: `443`, Destination: `0.0.0.0/0`

        - Rule #120: `Allow`, Protocol: `TCP`, Port range: `1024-65535`, Destination: `0.0.0.0/0`

    #### Private 서브넷 (PrivateSubnetACL)

    - **인바운드 규칙**:

        - Rule #100: `Allow`, Protocol: `TCP`, Port range: `5432`, Source: `10.0.0.0/16` (프라이빗 서브넷이 위치한 VPC의 CIDR 블록, 내부 통신 허용)

        - Rule #110: `Allow`, Protocol: `TCP`, Port range: `1024-65535`, Source: `10.0.0.0/16` (이 규칙은 내부 통신을 위한 포트 범위 허용)

    - **아웃바운드 규칙**:

        - Rule #100: `Allow`, Protocol: `TCP`, Port range: `80`, Destination: `0.0.0.0/0`

        - Rule #110: `Allow`, Protocol: `TCP`, Port range: `443`, Destination: `0.0.0.0/0`

        - Rule #120: `Allow`, Protocol: `TCP`, Port range: `1024-65535`, Destination: `0.0.0.0/0`

## RDS

### PostgreSQL RDS 인스턴스 생성

1. RDS 서비스로 이동: 상단 검색창에서 "RDS"를 검색하여 RDS 대시보드로 이동합니다.

1. DB 인스턴스 생성:

    - RDS 대시보드에서 "Databases"를 클릭합니다.

    - "Create database" 버튼을 클릭합니다.

    - **데이터베이스 생성 방법**: "Standard Create"를 선택합니다.

    - **엔진 옵션**: "PostgreSQL"을 선택합니다.

    - **버전**: 원하는 PostgreSQL 버전을 선택합니다.

1. DB 인스턴스 설정:

    - DB 인스턴스 식별자: 인스턴스의 이름을 입력합니다 (예: `MyPostgreSQLDB`).

    - 마스터 사용자 이름: 기본 관리자 계정 이름을 입력합니다 (예: `admin`).

    - 마스터 암호: 관리자 계정의 암호를 입력합니다.

1. 인스턴스 사양:

    - DB 인스턴스 클래스: 요구사항에 맞는 인스턴스 클래스를 선택합니다 (예: `db.t3.micro`).

    - 스토리지 유형: 일반적인 용도로 "General Purpose (SSD)"를 선택합니다.

    - 할당된 스토리지: 필요에 따라 스토리지 크기를 설정합니다 (예: 20GB).

1. 가용성 및 내구성:

    - 필요에 따라 Multi-AZ 배포를 설정합니다 (고가용성을 위해 권장).

1. 네트워크 및 보안:

    - VPC 선택: 생성한 VPC (`MyVPC`)를 선택합니다.

    - 서브넷 그룹: RDS 서브넷 그룹을 선택합니다 (RDS 인스턴스를 배치할 서브넷 그룹).

    - 퍼블릭 액세스 가능성: "No"를 선택하여 인스턴스를 프라이빗 서브넷에 배치합니다.

    - 보안 그룹: 이전에 생성한 `DBSG` 보안 그룹을 선택합니다.

1. 데이터베이스 옵션:

    - 기본 설정을 사용하거나, 필요에 따라 데이터베이스 이름을 입력합니다.

1. 백업 설정:

    - 자동 백업: 자동 백업을 활성화하고 보관 기간을 설정합니다 (예: 7일).

1. 모니터링:

    - 필요에 따라 Amazon CloudWatch Enhanced Monitoring을 활성화합니다.

1. 암호화:

    - 필요에 따라 암호화를 활성화합니다.

1. 추가 구성:

    - 필요에 따라 로깅 및 유지관리 옵션을 설정합니다.

1. DB 인스턴스 생성:

    - 모든 설정을 확인한 후 "Create database" 버튼을 클릭합니다.

## 마무리

이로써 DB를 포함한 서버 인프라가 일단 구성됐습니다. 다음 글에서는 이미지를 빌드해서 Push한 후, ECS에 배포를 해보겠습니다. 



