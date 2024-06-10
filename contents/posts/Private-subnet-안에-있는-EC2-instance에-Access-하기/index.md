---
IDX: "NUM-197"
tags:
  - AWS
  - BackEnd
  - VSCode
description: "Bastion Host, VSCode Remote - SSH 확장을 사용해 Private subnet 안에 있는 EC2 instance에 Access 하기"
update: "2024-06-10T08:49:00.000Z"
date: "2024-06-10"
상태: "Ready"
title: "Private subnet 안에 있는 EC2 instance에 Access 하기"
---
![](image1.png)
## 서론

프라이빗 서브넷에 있는 EC2 인스턴스에 접근하는 가장 쉬운 방법 중 하나는 VSCode Remote - SSH를 사용하여 Bastion Host를 통한 포트 포워딩을 설정하는 것입니다. 이 방법을 사용하면 VSCode에서도 원격 개발 환경을 구성할 수 있습니다. 다음은 이미 Private Subnet에 접근을 해야 하는 EC2 Instance가 존재하며, Private Subnet, Public Subnet등 네트워크 인프라는 모두 설정되어 있음을 전제로 해당 내용의 구현을 단계별로 설명한 내용입니다. 

## 1. Bastion Host 설정

퍼블릭 서브넷에 배치된 Bastion Host(Jump Box)를 통해 프라이빗 서브넷의 인스턴스에 접근할 수 있습니다. 이를 위해서 퍼블릭 서브넷에 새로운 EC2 인스턴스를 생성하고, 이 인스턴스를 Bastion Host로 사용합니다.

### EC2 인스턴스 생성

- AWS Management Console에 로그인하고 EC2 Dashboard로 이동합니다.

- Launch Instance를 클릭하여 새로운 인스턴스를 생성합니다.

- Amazon Linux 2 AMI를 선택하고, 인스턴스 유형으로 `t2.micro`를 선택합니다.

- Configure Instance Details 단계에서 퍼블릭 서브넷을 선택하고, Auto-assign Public IP를 `Enable`로 설정합니다.

- 나머지 설정을 완료하고 인스턴스를 시작합니다.

### 보안 그룹 설정

Bastion Host와 프라이빗 서브넷의 EC2 인스턴스 간의 보안 그룹 설정을 구성합니다.

#### Bastion Host 보안 그룹

퍼블릭 서브넷에 있는 Bastion Host의 보안 그룹을 설정하여 SSH 접근을 허용합니다.

1. Security Groups에서 Bastion Host의 보안 그룹을 선택합니다.

1. Inbound rules에서 다음 규칙을 추가합니다:

    - Type: SSH

    - Protocol: TCP

    - Port Range: 22

    - Source: My IP (또는 필요에 따라 특정 IP 범위)

#### 프라이빗 서브넷 EC2 인스턴스 보안 그룹

프라이빗 서브넷에 있는 EC2 인스턴스의 보안 그룹을 설정하여 Bastion Host의 SSH 접근을 허용합니다.

1. Security Groups에서 프라이빗 서브넷 EC2 인스턴스의 보안 그룹을 선택합니다.

1. Inbound rules에서 다음 규칙을 추가합니다:

    - Type: SSH

    - Protocol: TCP

    - Port Range: 22

    - Source: Bastion Host의 보안 그룹 ID

## 2. SSH 구성 파일 설정

SSH 구성 파일(`~/.ssh/config`)에 Bastion Host를 통해 프라이빗 서브넷의 인스턴스로 포트 포워딩을 설정합니다.

#### SSH 구성 파일 예시

```bash
Host bastion
    HostName <Bastion Host Public IP>
    User ec2-user
    IdentityFile /path/to/your-key-pair.pem

Host private-instance
    HostName <Private IP of EC2 in private subnet>
    User ec2-user
    IdentityFile /path/to/your-key-pair.pem
    ProxyJump bastion
```

이 설정에서 `bastion`은 퍼블릭 서브넷에 있는 Bastion Host를 나타내며, `private-instance`는 프라이빗 서브넷에 있는 EC2 인스턴스를 나타냅니다. `ProxyJump` 옵션을 사용하여 Bastion Host를 경유하도록 설정합니다.

## 3. VSCode 설정

VSCode에서 Remote - SSH 확장을 사용하여 프라이빗 서브넷의 인스턴스에 접근합니다.

#### VSCode Remote - SSH 설정

1. VSCode를 열고 Remote-SSH: Connect to Host... 명령을 실행합니다.

1. `private-instance`를 선택합니다.

VSCode는 `~/.ssh/config` 파일을 읽고, Bastion Host를 통해 프라이빗 서브넷의 인스턴스에 연결합니다.

## 4. 기타 



