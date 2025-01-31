---
IDX: "NUM-240"
tags:
  - Hobby
description: "Deepseek R1 로컬에 설치해보기"
update: "2025-01-31T07:33:00.000Z"
date: "2025-01-29"
상태: "Ready"
title: "Deepseek R1 로컬 설치"
---
![](image1.png)
## 서론

오늘은 요 며칠 화제의 챗지피티 대안 모델 Deepseek R1을 로컬에 간단하게 설치해보겠습니다. 중국에 정보가 넘어간다, 중국 공산당 관련 내용은 검열된다, 접속이 안된다, 이런 저런 말이 많지만 로컬에 설치하면 문제 없습니다. 

## 권장 사양

이 모델의 특징은 기존 모델 대비 엄청난 저사양에서도 돌릴 수 있다는 것이 특징입니다. 저는 안해봤지만 아이폰에서도 돌리는게 가능하다고 합니다. 저는 오늘 Deepseek 모델 중에서도 14b 모델을 실행해볼 예정인데, 이 모델을 위해선 적어도 32기가의 메모리에, 모델 용량만 10gb 언저리이니 넉넉하게 2-30gb의 여유 공간이 필요할 것 같습니다. 

저는 M3 Pro macbook을 사용중이며, 램 용량은 32기가를 사용중으로, 결과적으로는 속도는 조금 답답했지만 넉넉하게 잘 실행됐습니다. 

## Ollama

오늘 모델은 Ollama를 사용해 서빙하려고 합니다. Ollama는 로컬에서 AI 모델을 쉽게 관리하고 실행할 수 있게 해주는 도구입니다. 간단한 명령어로 모델을 설치하고, 서버를 실행하며, 다양한 설정을 조정할 수 있어 Deepseek R1과 같은 모델을 로컬에서 운영하는 데 매우 유용합니다.

## Open-WebUI

또 Ollama로 서빙된 모델을 사용하기 위한 UI로 Open-WebUI를 사용하려고 합니다. Open-WebUI는 사용자 친화적인 웹 인터페이스를 제공하여 AI 모델과 상호작용할 수 있게 해줍니다. 별도의 코딩 없이도 모델을 테스트하고, 설정을 변경하며, 결과를 확인할 수 있어 비개발자도 쉽게 사용할 수 있습니다.

ChatGPT와 유사한 인터페이스를 제공해 사용이 아주 편리합니다. 

## 설치

### Ollama 설치

Ollama를 설치하기 위해 Homebrew를 사용합니다. 물론 Ollama도 Docker-compose를 사용해 실행할 수 있지만, 기본적으로 docker의 가상환경에는 많은 램이 배정되지 않으므로 추가적인 설정이 필요합니다. 저는 그 과정이 일반적이지 않으며, 귀찮기도 해 Ollama는 로컬에 그냥 brew로 설치하기로 했습니다. 

터미널을 열고 다음 명령어를 입력하세요.

```bash
brew install --cask ollama
```

설치가 완료되면 Ollama 서버를 백그라운드에서 실행합니다.

```bash
nohup ollama serve > ollama.log 2>&1 &
```

그 다음, Deepseek R1:14B 모델을 다운로드합니다.

```bash
ollama pull deepseek-r1:14b
```

### Docker-Compose 설정

Docker와 Docker-Compose가 설치되어 있는지 확인합니다. 설치되지 않았다면 [Docker 공식 사이트](https://www.docker.com/)에서 설치할 수 있습니다.

다음과 같은 docker-compose.yml 파일을 작성합니다. 

```python
services:
    open-webui:
        image: ghcr.io/open-webui/open-webui:main
        container_name: open-webui
        restart: unless-stopped
        environment:
            - OLLAMA_BASE_URL=http://host.docker.internal:11434
        ports:
            - '33000:8080'
        volumes:
            - openwebui_data:/app/backend/data

volumes:
    openwebui_data:
```

포트는 자유롭게 지정해도 상관없습니다. 

## 실행

yaml 파일이 있는 터미널에서 다음 명령어를 실행하여 Docker 컨테이너를 시작합니다.

```bash
docker compose up -d
```

이후 [Open-WebUI](http://localhost:33000/) 페이지로 접속하면 실행됩니다. 혹시 다른 포트를 사용했다면, 포트를 바꿔주면 됩니다. 

## 기타 모델 다운로드

[Ollama 사이트](https://ollama.com/search)에서 다운로드 가능한 다른 모델을 살펴볼 수 있습니다. 원하는 모델의 모델명:태그를 Open-WebUI 관리 페이지에 입력해주면 간단하게 다른 모델을 실행할 수 있습니다. 

또는 [huggingface 사이트](https://huggingface.co/models)에서 원하는 모델을 선택한 후, Quantizations된 모델을 제공한다면 해당 모델을 Ollama에서 바로 사용할 수도 있습니다. 

## 결론

실행은 잘 되고, 성능도 꽤 좋아보입니다. 하지만 아무래도 맥북에서는 속도가 좀 답답한 감이 있으며, 한국어는 아무래도 좀 능력이 떨어지는지 자주 외국어와 섞여나오는 증상이 있습니다. 

아무래도 저는 아직은 당분간은 유료 챗지피티를 사용할 것 같습니다. 

