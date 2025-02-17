---
IDX: "NUM-243"
tags:
  - BackEnd
  - Nginx
description: "리버스 프록시 구현하기"
update: "2025-02-17T11:35:00.000Z"
date: "2025-02-17"
상태: "Ready"
title: "Nginx를 활용한 Reverse Proxy 구현(2)"
---
![](image1.png)
## 서론

이번에는 리버스 프록시를 구현해보고, 동적으로 환경변수를 삽입하는 방법에 대해 알아보겠습니다. 

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

### **WebSocket 리버스 프록시**

실시간 데이터 처리가 필요한 WebSocket 서비스도 Nginx에서 프록시할 수 있습니다.

```plain text
server {
    listen 80;
    server_name example.com;

    location /ws/ {
        proxy_pass http://websocket-server:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
}
```

- proxy\_http\_version 1.1; → WebSocket을 위해 HTTP 1.1을 사용

    nginx는 기본적으로 http 통신에 1.0 버전을 사용하므로 웹소켓 통신을 위해서는 1.1버전 사용을 설정해줘야 합니다. 

- proxy\_set\_header Upgrade $http\_upgrade; → WebSocket 연결을 위한 Upgrade 헤더 설정

- proxy\_set\_header Connection "Upgrade"; → 연결을 유지하기 위한 설정

이렇게 하면 Nginx를 통해 WebSocket 요청도 정상적으로 처리할 수 있습니다.

## envsubst를 활용한 환경변수 적용

실제 운영 환경에서는 환경 변수를 활용하여 설정을 동적으로 관리하는 경우가 많습니다. 특히 Docker 컨테이너 환경에서는 환경 변수로 설정값을 주입하는 것이 일반적입니다. Nginx에서는 envsubst를 사용하면 Nginx 설정 파일에서 환경 변수를 동적으로 적용할 수 있습니다.

### envsubst?

envsubst는 환경 변수를 치환하는 유틸리티로 쉘 스크립트에서 사용되며 템플릿 파일 내의 변수를 실제 값으로 대체할 수 있습니다. 예를 들어, Nginx 설정 파일 내에서 환경 변수 ${VAR\_NAME}을 포함시킨 후 envsubst를 실행하면, 해당 변수 값이 적용된 새로운 설정 파일을 생성할 수 있습니다.

### gettext 설치

envsubst는 `gettext` 패키지에 포함되어 있습니다. 따라서 envsubst를 사용하려면 `gettext`를 먼저 설치해야 합니다.

```bash
brew install gettext
```

Ubuntu 및 Debian 계열에서는 gettext 패키지를 설치하면 됩니다.

```bash
sudo apt update
sudo apt install gettext -y
```

### **환경 변수를 적용한 Nginx 설정**

#### **환경 변수가 포함된 Nginx 설정 파일 (nginx.conf.template)**

`nginx.conf.template` **파일을 생성해 다음의 내용을 작성합니다.** 

```plain text
server {
    listen 80;
    server_name ${SERVER_NAME};

    location / {
        proxy_pass http://${BACKEND_HOST}:${BACKEND_PORT};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

여기서 `${SERVER_NAME}`, `${BACKEND_HOST}`, `${BACKEND_PORT}` 값은 **실제 실행 시점에 환경 변수로 치환될 예정**입니다.

#### envsubst를 이용한 환경 변수 적용

envsubst를 활용해 환경 변수를 포함한 템플릿 파일(nginx.conf.template)을 실제 Nginx 설정 파일(nginx.conf)로 변환합니다.

```bash
envsubst '$SERVER_NAME $BACKEND_HOST $BACKEND_PORT' < nginx.conf.template > nginx.conf
```

당연히 환경변수는 미리 정의되어 있어야 합니다. 

#### **Nginx 실행**

치환된 설정 파일을 적용한 후 Nginx를 실행합니다.

```bash
nginx -t  # 문법 검사
nginx -s reload  # 설정 적용
```

### Docker에서의 활용

envsubst를 활용해 Docker 환경에서 Nginx 설정을 동적으로 주입할 수 있습니다.

#### Dockerfile 작성

```docker
FROM nginx:latest

# 기본 환경 변수 설정
ENV SERVER_NAME=myapp.example.com
ENV BACKEND_HOST=backend-service
ENV BACKEND_PORT=5000

# 환경 변수 템플릿 복사
COPY nginx.conf.template /etc/nginx/templates/nginx.conf.template

# 컨테이너 실행 시 envsubst를 이용해 환경 변수를 적용
CMD envsubst < /etc/nginx/templates/nginx.conf.template > /etc/nginx/nginx.conf && nginx -g 'daemon off;'
```



