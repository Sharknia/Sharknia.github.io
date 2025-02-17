---
IDX: "NUM-242"
tags:
  - BackEnd
  - Nginx
description: "Nginx를 활용한 Reverse Proxy 구현"
update: "2025-02-17T06:41:00.000Z"
date: "2025-02-17"
상태: "Ready"
title: "Nginx를 활용한 Reverse Proxy 구현"
---
![](image1.png)
## 서론

현재 근무하는 회사의 API Gateway 역할을 하는 서버의 유지보수/고도화 업무를 맡게 됐습니다. API Gateway 역할을 하고 있다고는 하지만 사실은 아직 아주 일부의 기능만 구현된 상태로, 일단은 각 마이크로서비스의 리버스 프록시 역할을 할 수 있도록 Nginx를 활용하려고 합니다. 

이번 글에서는 프록시와 리버스 프록시의 개념부터 Nginx의 설치 및 설정, 환경변수를 활용한 동적 구성 방법 등을 순서대로 다뤄보려고 합니다. 

## 프록시와 리버스 프록시란? 

### 프록시

클라이언트와 서버 사이에서 중계 역할을 수행하며, 클라이언트의 요청을 대신 서버에 전달하고 서버의 응답을 다시 클라이언트로 전달합니다.

#### 주요 기능

- 캐싱: 자주 요청되는 데이터를 임시 저장해서 서버의 부하를 줄이고 응답 속도를 개선합니다. 

- 익멱성 제공: 클라이언트의 실제 IP 주소를 숨기고 프록시 서버의 IP를 사용해 요청을 전달합니다. 

- 접근 제어 및 필터링: 특정 웹사이트나 콘텐츠에 대한 접근 제한을 설정할 수 있습니다. 

#### 활용 사례

- 기업이나 학교 등에서 내부 사용자의 인터넷 접근을 제어하고 모니터링 할 때 사용합니다. 

- 개인이 익명성을 확보하거나 지역 제한 우회를 위해 활용하기도 합니다. 

### 리버스 프록시 

서버 앞단에 위치해 클라이언트의 요청을 받아 이를 내부의 여러 백엔드 서버로 분배하는 역할을 합니다. 클라이언트는 리버스 프록시와만 통신하며 내부 서버 구조는 외부에 노출되지 않습니다.

#### 주요 기능

- 보안 강화: 내부 서버의 직접 노출을 방지해 외부 공격으로부터 보호합니다. 

- 부하 분산: 다수의 백엔드 서버에 요청을 효율적으로 분산시켜 서버 과부하를 예방하고 서비스 안정성을 향상시킵니다. 

- 캐싱 및 압축: 정적 콘텐츠 캐싱 및 데이터 압축을 통해 응답 속도를 개선하고 네트워크 대역폭을 최적화합니다. 

#### 활용 사례

- 대규모 웹사이트나 API 서비스에서 하나의 접점을 통해 여러 서버를 관리할 때 사용합니다. 

- CDN 및 클라우드 환경에서 효율적이고 안전한 서비스 운영을 지원합니다. 

### 차이점

결국 프록시와 리버스 프록시의 가장 큰 차이점은 `누가 프록시를 이용하는가` 라고 할 수 있습니다. 

## Nginx란?

Nginx는 가볍고 빠른 웹 서버이자 리버스 프록시 서버로, 높은 성능과 확장성을 제공합니다. 기본적으로 정적 콘텐츠 제공에 최적화되어 있으며, 리버스 프록시, 로드 밸런싱, API Gateway 역할도 수행할 수 있습니다.

### 주요 특징

- 고성능 & 확장성: 이벤트 기반 아키텍쳐로 동시 요청 처리가 뛰어나고 적은 리소스로 많은 트래픽을 감당할 수 있습니다. 

- 리버스 프록시 기능: 백엔드 서버로의 트래픽을 중계하는 리버스 프록시 기능을 할 수 있습니다. 

- 유연한 설정: 설정 파일이 직관적이고 다양한 플러그인 및 모듈을 지원합니다. 

- 보안 강화: DDoS 방어, 접근 제어, 인증 및 암호화 처리 기능을 제공합니다. 

### 활용 사례

- 정적 콘텐츠 서빙

- 리버스 프록시 및 API Gateway

- 로드 밸런싱을 통한 트래픽 분산

## Nginx 설치하기

Mac에서 Nginx를 설치하는 가장 쉬운 방법은 Homebrew를 이용하는 것입니다.

```bash
brew install nginx
```

설치 후 실행하고, 상태를 확인합니다. 

```bash
brew services start nginx
brew services list
```

설치가 완료되면 브라우저에서 http://localhost:8080을 열어 Nginx 기본 페이지가 표시되는지 확인합니다.

기본적으로 Mac에서 설치된 Nginx는 8080 포트에서 실행됩니다.

다음의 명령어로 nginx를 중지할 수 있습니다. 

```bash
brew services stop nginx
```

### 설정 파일 경로 **(Homebrew 버전)**

설정 파일은 다음 경로에서 찾을 수 있습니다.

```bash
/opt/homebrew/etc/nginx/nginx.conf
```

## Nginx 설정 파일 구조 및 관리 방법

Nginx 설정 파일(nginx.conf)은 여러 개의 블록으로 구성되어 있으며, 역할별로 나누어 관리할 수 있습니다.

### **기본적인 Nginx 설정 파일 구조**

기본적으로 Nginx 설정 파일은 다음과 같은 구조를 가집니다.

```plain text
worker_processes auto;  # 사용 가능한 CPU 코어 개수만큼 워커 프로세스를 자동 설정

events {
    worker_connections 1024;  # 하나의 워커 프로세스당 처리 가능한 최대 연결 수
}

http {
    include mime.types;  # MIME 타입 설정 포함
    default_type application/octet-stream;

    sendfile on;  # 파일 전송을 최적화하여 성능 향상
    keepalive_timeout 65;  # 연결 유지 시간 설정

    # 로그 설정
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # 압축 설정
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    # 서버 설정 파일 포함
    include /etc/nginx/conf.d/*.conf;
}
```

#### **주요 블록 설명**

- worker\_processes auto; → CPU 코어 개수에 맞춰 자동으로 워커 프로세스 개수 설정

- events 블록 → 동시 연결 수 제한

- http 블록 → HTTP 관련 설정, MIME 타입, 압축, 로그 설정 포함

- server 블록 → 개별 서버 설정 (리버스 프록시, 정적 파일 서빙 등)

- include → 설정 파일을 외부에서 불러오는 방법

### **설정 파일을 여러 개로 분리하여 관리하기**

Nginx 설정이 많아지면 하나의 nginx.conf에 모든 설정을 넣는 것은 비효율적입니다. 이를 방지하기 위해 include 지시어를 활용하여 설정 파일을 분리할 수 있습니다.

예를 들어, `/etc/nginx/conf.d/`폴더에 개별 서버 설정 파일을 저장할 수 있습니다.

#### **nginx.conf 파일에서 설정 불러오기**

```plain text
http {
    include /etc/nginx/conf.d/*.conf;
}
```

#### **/etc/nginx/conf.d/my\_service.conf 파일 생성**

```plain text
server {
    listen 80;
    server_name myapp.example.com;

    location / {
        proxy_pass http://backend:5000;
    }
}
```

이렇게 하면 개별 서비스를 독립적으로 관리할 수 있어 **유지보수성이 높아집니다.**

### **설정 파일 문법 검사 및 적용 방법**

설정 파일을 수정한 후에는 문법 검사를 수행할 수 있습니다. 

```bash
nginx -t  # 문법 검사
nginx -s reload  # 변경 사항 적용
```

만약 설정에 오류가 있다면 nginx -t를 실행했을 때 오류 메시지가 출력됩니다. 오류를 수정한 후 다시 검사한 뒤 적용하면 됩니다.

### **개별 사이트 설정 파일 활용 (sites-available, sites-enabled)**

일반적으로 배포 환경에서는 사이트 설정을 /etc/nginx/sites-available/에 저장하고 활성화할 설정만 /etc/nginx/sites-enabled/에 심볼릭 링크를 걸어 관리하는 방식도 사용합니다.

```plain text
sudo ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/
```

이렇게 하면 특정 사이트만 쉽게 활성화/비활성화할 수 있습니다.

## 리버스 프록시의 구현

### 기본 리버스 프록시 설정

가장 간단한 리버스 프록시 설정은 클라이언트의 요청을 특정 백엔드 서버로 전달하는 것입니다. 다음은 http://localhost:80으로 들어오는 요청을 내부의 **백엔드 서버(8080 포트)**로 프록시하는 설정입니다.

```plain text
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### **다중 서비스 리버스 프록시**

실제 운영 환경에서는 여러 개의 마이크로서비스를 운영할 수 있습니다.

이때, 요청 경로에 따라 적절한 백엔드 서버로 분배하도록 설정할 수 있습니다.

```plain text
server {
    listen 80;
    server_name example.com;

    location /api/ {
        proxy_pass http://backend-service:5000;
    }

    location /auth/ {
        proxy_pass http://auth-service:4000;
    }

    location /static/ {
        root /var/www/html;
    }
}
```

이 설정에서는 

- /api/ 요청 → backend-service:5000으로 전달

- /auth/ 요청 → auth-service:4000으로 전달

- /static/ 요청 → Nginx가 직접 정적 파일 서빙

을 실시합니다. 이렇게 하면 마이크로서비스 구조에서 API 요청을 특정 서비스로 라우팅할 수 있습니다.

### 로드 밸런싱을 활용한 리버스 프록시 

Nginx는 단순한 리버스 프록시 역할뿐만 아니라, 부하 분산(Load Balancing) 기능도 수행할 수 있습니다.

예를 들어, 여러 개의 백엔드 서버가 있을 때, 요청을 분산하여 성능을 최적화할 수 있습니다.

```plain text
upstream backend_servers {
    server backend1:5000;
    server backend2:5000;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

`upstream backend_servers {}`  을 활용해 여러개의 백엔드 서버 그룹을 정의해주었습니다. 요청이 들어오면 자동으로 두 개의 백엔드 서버(backend1, backend2)로 부하를 분산하여 전달합니다. 

부하 분산 알고리즘은 기본적으로 `Round Robin`이 적용되며, 특정 방식으로 변경할 수도 있습니다.

```plain text
upstream backend_servers {
    least_conn;  # 가장 적은 연결 수를 가진 서버로 요청 전달
    server backend1:5000;
    server backend2:5000;
}
```

### **HTTPS(SSL) 리버스 프록시**

보안을 위해 SSL을 적용한 리버스 프록시를 설정할 수도 있습니다.

이 경우, 클라이언트는 HTTPS로 요청을 보내고, Nginx가 내부 서버와는 HTTP로 통신하는 방식입니다.

```plain text
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/example.crt;
    ssl_certificate_key /etc/nginx/ssl/example.key;

    location / {
        proxy_pass http://backend-service:5000;
    }
}
```

제 프로젝트에서는 기본적으로 컨테이너 안에서 작동하게 되므로 이 설정은 적용하지 않았습니다. 

