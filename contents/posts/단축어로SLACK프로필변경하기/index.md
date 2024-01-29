---
tags:
  - ETC
update: "2024-01-28"
date: "2023-11-17"
상태: "POST"
title: "단축어로 SLACK 프로필 변경하기"
---
## 소개

애플 단축어를 활용해 간단하게 슬랙 상태를 원클릭으로 변경할 수 있습니다. 슬랙의 API를 이용합니다. 

저는 개인적으로 raycast의 뽀모도로 익스텐션과 함께 다음과 같이 사용하고 있습니다.

![](image3.png)
집중을 시작할 때에 

- 집중 모드 킴
- 음량 조절
- 노래 재생
- 슬랙 방해금지 상태 변경
![](image4.png)
집중을 끝낼 때에

- 집중 모드 끔
- 슬랙 방해금지 상태 해제


집중 모드에서는 다른 알림을 모두 꺼두고 슬랙 알림만 켜서 업무 흐름은 놓치지 않게 해두었습니다. 

![](image5.png)
메뉴바에서 원클릭으로 설정할 수 있어 편리합니다. 

## 설정법

### 슬랙의 API App 생성

먼저 슬랙의 API App을 만들어야 합니다. 

[https://api.slack.com/](https://api.slack.com/)

해당 페이지로 이동 후, 우측 상단의 Your apps으로 이동합니다. 

`Create New App` 을 클릭합니다. 

`From an app manifest` 를 선택 후, workspace를 고르고 Next 합니다. 

`YAML`을 고르고, 다음의 내용을 입력합니다. 

```yaml
display_information:
  name: Apple 상태변경 단축어
oauth_config:
  scopes:
    user:
      - users.profile:write
settings:
  org_deploy_enabled: false
  socket_mode_enabled: false
  token_rotation_enabled: false
```

권한을 더 원한다면 더 많이 입력해주셔도 됩니다. 

`Create`를 눌러 앱을 생성합니다. 



그 후 바로 그 화면에서 `Install to Workspace`를 클릭해 앱을 인스톨합니다. 



그리고, 좌측 메뉴에서 `OAuth & Permissions` 메뉴로 이동합니다. 

**OAuth Token** 값을 복사해줍니다. 슬랙 페이지에서 할 일은 여기까지입니다. 

### 단축어 설정

단축어 앱을 열고 새로운 단축어를 추가합니다. 

`URL 콘텐츠 가져오기` 를 검색해서 다음과 같이 설정합니다. 

![](image6.png)
입력해야 하는 값은 다음과 같습니다. 

- 헤더
키 : Authorization

값 : Bearer xoxp-xxxxx-xxxx…



키 : Content-Type

값 : application/json; charset=UTF-8

- 바디
키 : profile

값 : 

```json
{
		"status_text": "방해금지",
		"status_emoji": ":no_entry_sign:"
}
```

원하는 이모지와 텍스트를 넣어주시면 됩니다. 



이렇게 하면 단축어가 완성됩니다. 

## 참고

[https://api.slack.com/apps?deleted=1](https://api.slack.com/apps?deleted=1)

