---
IDX: "NUM-182"
tags:
  - BackEnd
description: "ngrok을 활용한 로컬 서버 공개"
update: "2024-05-14T01:36:00.000Z"
date: "2024-05-14"
상태: "Ready"
title: "ngrok: 로컬 서버를 쉽게 공개하는 도구"
---
## 서론

로컬에서 개발할 때, 종종 외부에서 로컬 서버에 접근해야 하는 상황이 발생합니다. 예를 들어, 웹훅(Webhook)을 테스트하거나, 클라이언트에게 데모를 보여주기 위해 로컬 서버를 공개해야 할 때가 있습니다. 이러한 상황에서 매우 유용한 도구가 바로 ngrok입니다.

## ngrok 소개

ngrok은 로컬에서 실행 중인 서버를 공용 인터넷에 노출시켜주는 역방향 프록시 도구입니다. 이를 통해 로컬에서 개발 중인 애플리케이션을 외부에서도 접근할 수 있게 됩니다. ngrok은 다음과 같은 주요 기능을 제공합니다:

- HTTPS 터널링: 로컬 서버를 HTTPS를 통해 안전하게 노출합니다.

- 인스펙션 인터페이스: 요청과 응답을 실시간으로 확인할 수 있는 웹 인터페이스를 제공합니다.

- 포트 포워딩: 특정 포트를 지정하여 외부에서 접근할 수 있게 합니다.

- 자동 재연결: 연결이 끊어질 경우 자동으로 다시 연결합니다.

## ngrok 설치 및 기본 사용법

ngrok을 사용하기 위해서는 먼저 설치가 필요합니다. 다음은 ngrok 설치 및 기본 사용법에 대한 단계별 설명입니다.

#### 1. ngrok 다운로드 및 설치

[ngrok 공식 웹사이트](https://ngrok.com/)에서 운영체제에 맞는 ngrok을 설치합니다. 

맥에서는 간단하게 다음의 명령어를 통해 설치할 수 있습니다. 

```shell
brew install ngrok/ngrok/ngrok
```

정상적인 실행을 위해서는 토큰을 등록해야 합니다. 사이트에 회원가입을 합니다. 

로그인 까지 마친 후 [이 페이지](https://dashboard.ngrok.com/get-started/setup/macos)의 installation 섹션에서 auth-token을 얻을 수 있습니다. 

```shell
ngrok config add-authtoken [token]
```

위의 명령어를 실행하면 토큰이 저장됩니다. 

#### 2. ngrok 실행

ngrok 설치가 완료되면, 다음 명령어를 사용하여 로컬 서버를 공개할 수 있습니다. 여기서 `8080`은 로컬 서버가 실행 중인 포트 번호입니다.

```bash
ngrok http 8080
```

명령어를 실행하면 다음과 같은 출력이 나타납니다:

```bash
ngrok by @inconshreveable                                      (Ctrl+C to quit)

Session Status                online
Account                       YourAccount (Plan: Free)
Version                       2.3.40
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://abcdefgh.ngrok.io -> http://localhost:8080
Forwarding                    https://abcdefgh.ngrok.io -> http://localhost:8080

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

여기서 `https://abcdefgh.ngrok.io` URL을 통해 외부에서 로컬 서버에 접근할 수 있습니다.

## 마무리

이제 ngrok을 사용하여 로컬 서버의 내용을 https로 접근할 수 있습니다. 무료 플랜의 단점(껐다 킬 때마다 주소가 바뀌는 등)도 있지만, 간단한 테스트를 수행하기에는 더 없이 적절합니다. 



