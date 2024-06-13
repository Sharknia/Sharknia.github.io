---
IDX: "NUM-198"
tags:
  - AWS
  - BackEnd
  - VSCode
description: "AWS에서 EC2 인스턴스 생성 및 Visual Studio Code로 원격 연결하기"
update: "2024-06-13T12:00:00.000Z"
date: "2024-06-13"
상태: "Ready"
title: "AWS에서 EC2 인스턴스 생성 및 Visual Studio Code로 원격 연결하기"
---
## 서론

이번 글에서는 AWS에서 VPC를 생성하고, EC2 인스턴스를 설정하며, 터미널과 Visual Studio Code를 통해 원격으로 연결하는 과정을 기록하려고 합니다. 

## VPC 생성

### 1. VPC 생성

1. AWS Management Console에 로그인합니다.

1. 상단 검색창에 "VPC"를 입력하고 VPC 대시보드로 이동합니다.

1. "VPCs"를 선택한 후 "Create VPC" 버튼을 클릭합니다.

1. VPC 이름, IPv4 CIDR 블록(예: 10.0.0.0/16), IPv6 CIDR 블록 설정, 테넌시 등을 입력합니다.

1. "Create VPC" 버튼을 클릭하여 VPC를 생성합니다.

### 2 서브넷 생성

1. VPC 대시보드에서 "Subnets"를 선택합니다.

1. "Create Subnet" 버튼을 클릭합니다.

1. 서브넷 이름, VPC 선택, 가용 영역(AZ) 선택, IPv4 CIDR 블록(예: 10.0.1.0/24)을 입력합니다.

1. "Create Subnet" 버튼을 클릭하여 서브넷을 생성합니다.

### 3. 인터넷 게이트웨이 생성 및 연결

1. VPC 대시보드에서 "Internet Gateways"를 선택합니다.

1. "Create internet gateway" 버튼을 클릭합니다.

1. 인터넷 게이트웨이 이름을 입력하고 "Create internet gateway" 버튼을 클릭합니다.

1. 생성된 인터넷 게이트웨이를 선택하고 "Actions" 메뉴에서 "Attach to VPC"를 선택합니다.

1. 앞서 생성한 VPC를 선택하고 "Attach internet gateway"를 클릭합니다.

### 4. 라우팅 테이블 생성 및 설정

1. VPC 대시보드에서 "Route Tables"를 선택합니다.

1. "Create route table" 버튼을 클릭합니다.

1. 라우팅 테이블 이름과 VPC를 선택한 후 "Create route table" 버튼을 클릭합니다.

1. 생성된 라우팅 테이블을 선택하고 "Routes" 탭을 클릭한 후 "Edit routes" 버튼을 클릭합니다.

1. 기본 경로(0.0.0.0/0)에 인터넷 게이트웨이를 대상으로 추가합니다.

1. "Save routes" 버튼을 클릭합니다.

1. "Subnet associations" 탭을 클릭하고 "Edit subnet associations" 버튼을 클릭합니다.

1. 앞서 생성한 서브넷을 선택하고 "Save" 버튼을 클릭합니다.

## EC2 인스턴스 생성

### 1. SSH 키 페어 생성

1. 상단 검색창에 "EC2"를 입력하고 EC2 대시보드로 이동합니다.

1. 왼쪽 사이드바에서 "Key Pairs"를 선택합니다.

1. "Create key pair" 버튼을 클릭합니다.

1. 키 페어 이름을 입력하고, 키 파일 형식을 선택한 후 "Create key pair" 버튼을 클릭합니다. 이때, PEM 형식을 선택하는 것을 권장합니다.

1. 키 페어가 생성되면 자동으로 .pem 파일이 다운로드됩니다. 이 파일을 안전한 위치에 저장합니다.

### 2. EC2 인스턴스 생성

1. EC2 대시보드에서 "Instances"를 선택합니다.

1. "Launch instances" 버튼을 클릭합니다.

1. "Name and tags" 섹션에서 인스턴스 이름을 입력합니다.

1. "Application and OS Images (Amazon Machine Image)" 섹션에서 사용할 AMI를 선택합니다.

1. "Instance type" 섹션에서 원하는 인스턴스 유형을 선택합니다.

1. "Key pair (login)" 섹션에서 앞서 생성한 키 페어를 선택합니다.

1. "Network settings" 섹션에서 편집을 눌러 1에서 생성한 VPC와 서브넷을 선택합니다.

1. 보안 그룹 설정에서 기존 보안 그룹을  새로 생성합니다. SSH (22번 포트)를 허용하는 규칙이 포함되어야 SSH로 접근할 수 있습니다. 허용할 IP는 내 아이피를 선택하거나 IPv4 Anywhere을 선택하는데, IPv4 Anywhere을 선택한 경우에는 모든 IP에서 접근이 가능함에 주의해야 합니다. 

1. "Launch instance" 버튼을 클릭하여 인스턴스를 생성합니다.

## 터미널에서 연결

### 1. SSH 키 파일 권한 설정

1. 터미널을 엽니다 (Windows의 경우 Git Bash 또는 PuTTY 사용).

1. 다운로드한 .pem 파일의 권한을 설정합니다.

    ```shell
    chmod 400 path/to/your-key-pair.pem
    ```

### 2. SSH를 사용하여 인스턴스에 연결

1. SSH를 사용하여 인스턴스에 연결합니다. 아래 명령어를 사용하시면 됩니다. `instance-public-dns`는 EC2 인스턴스의 퍼블릭 DNS 이름입니다.

    ```shell
    ssh -i path/to/your-key-pair.pem ec2-user@instance-public-dns
    ```

### 3. EC2 인스턴스의 퍼블릭 DNS 확인 방법

1. EC2 대시보드에서 "Instances"를 선택합니다.

1. 연결하려는 인스턴스를 선택합니다.

1. 하단에 인스턴스 세부 정보가 나타나며, "Public DNS (IPv4)" 항목에서 퍼블릭 DNS 이름을 확인할 수 있습니다.

## VS Code에서 연결

### 1. Visual Studio Code 설치 및 설정

#### Remote - SSH 확장 설치

- Visual Studio Code를 열고, 왼쪽 사이드바에서 확장 아이콘(네모 모양)을 클릭합니다.

- 검색창에 `Remote - SSH`를 입력하고 `Remote - SSH` 확장을 설치합니다.

### 2. Visual Studio Code에서 SSH 구성

1. VS Code에서 Command Palette 열기

    - `Ctrl+Shift+P` (또는 `Cmd+Shift+P` on Mac)을 눌러 Command Palette를 엽니다.

1. SSH 구성 파일 편집

    - Command Palette에서 `Remote-SSH: Open SSH Configuration File...`을 입력하고 선택합니다.

    - 로컬의 SSH 설정 파일 경로를 선택합니다 (보통 `~/.ssh/config`).

1. SSH 호스트 추가

    - 구성 파일에 다음 내용을 추가합니다. `Host` 부분은 원하는 이름으로 설정하고, `HostName`, `User`, `IdentityFile` 부분을 자신의 정보로 변경합니다.

        ```plain text
        Host your-instance-name
            HostName instance-public-dns
            User ec2-user
            IdentityFile /path/to/your-key-pair.pem
        ```

### 3. Visual Studio Code에서 원격 연결

1. VS Code의 Remote Explorer 열기

    - Visual Studio Code의 왼쪽 사이드바에서 "Remote Explorer" 아이콘을 클릭합니다.

1. SSH Targets 섹션에서 호스트 선택

    - `SSH Targets` 섹션에서 추가한 호스트를 선택하고 `Connect to Host in New Window`를 클릭합니다.

1. 호스트 키 확인

    - 연결할 때 처음으로 호스트 키를 신뢰할 것인지 묻는 창이 나타날 수 있습니다. "Yes"를 클릭하여 신뢰합니다.

1. 원격 서버에 연결

    - 연결이 성공하면 새로운 VS Code 창이 열리며, 원격 서버의 파일 시스템에 접근할 수 있습니다.

