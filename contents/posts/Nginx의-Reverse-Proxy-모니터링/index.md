---
IDX: "NUM-249"
tags:
  - BackEnd
  - DevOps
description: "모니터링과 Filebeat"
update: "2025-02-21T07:41:00.000Z"
date: "2025-02-21"
상태: "Ready"
title: "Nginx의 Reverse Proxy 모니터링"
---
## Nginx 로깅의 중요성

Nginx는 웹 서버 및 리버스 프록시 역할을 수행하면서 모든 HTTP 요청과 응답을 기록합니다. 이 로그를 효과적으로 모니터링하면 다음과 같은 장점을 얻을 수 있습니다.

- 트래픽 분석: 요청량이 가장 많은 API는 무엇인가?

- 응답 코드 모니터링: 4xx, 5xx 오류가 많이 발생하는가?

- 장애 감지: 502, 504 에러가 급증하고 있는가?

- 보안 이슈 탐지: 특정 IP에서 비정상적인 요청이 들어오는가?

따라서 Nginx 로그를 실시간으로 수집 및 분석하는 시스템이 필요합니다.

## Filebeat를 선택한 이유

현재 백엔드 서버에서 이미 로그를 수집하는데에 Elasticsearch를 사용하고 있습니다. Filebeat는 Nginx 로그를 실시간으로 수집하고 Elasticsearch로 전송하는 데 가장 적합한 도구입니다.

## Filebeat의 장점 및 작동 원리

### 장점

- 실시간 로그 수집: Nginx 로그가 추가되면 즉시 감지

- 경량 실행: CPU 및 메모리 사용량이 적음

- 네트워크 장애 대비: Elasticsearch가 다운되면 재전송 가능

- 구성 간편: 설정 파일(`filebeat.yml`)만으로 빠르게 적용 가능

### Filebeat의 작동 방식

1. 로그 파일을 지속적으로 모니터링 (`tail -f` 역할)

1. 읽은 위치를 저장 (`registry file` 사용)하여 중복 수집 방지

1. 로그를 Elasticsearch로 전송 (JSON 형태로 저장)

## Filebeat 설정하기

### Filebeat 설치

다음의 명령어로 설치합니다. 

```bash
sudo apt update && sudo apt install filebeat
```

### Filebeat 설정 파일 수정 (`/etc/filebeat/filebeat.yml`)

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
    - /var/log/nginx/error.log
  fields:
    service: nginx

output.elasticsearch:
  hosts: ["http://localhost:9200"]
  index: "nginx-logs-%{+yyyy.MM.dd}"
```

## 컨테이너 위에서 실행되는 Nginx와 Filebeat의 연동

### Docker 환경에서 Nginx 로그 마운트 설정

Filebeat가 Nginx 로그를 읽을 수 있도록 Docker 볼륨 마운트를 설정해야 합니다.

#### `docker-compose.yml` 예제

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx_server
    volumes:
      - /var/log/nginx:/var/log/nginx  # 로그 공유
    ports:
      - "80:80"

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.17.0
    container_name: filebeat
    user: root
    volumes:
      - /var/log/nginx:/var/log/nginx  # Nginx 로그 공유
      - /etc/filebeat.yml:/usr/share/filebeat/filebeat.yml  # Filebeat 설정 파일
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
    command: ["filebeat", "-e", "-strict.perms=false"]
```

### Filebeat 실행 및 확인

```plain text
docker-compose up -d
```

```plain text
docker logs filebeat -f
```

정상적으로 실행되면 Filebeat가 Elasticsearch로 로그를 전송하고 있는 것을 확인할 수 있습니다. 



