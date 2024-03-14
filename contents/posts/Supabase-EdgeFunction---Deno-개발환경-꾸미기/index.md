---
IDX: "NUM-159"
tags:
  - Supabase
  - Deno
description: "로컬에서 Deno 개발하기"
update: "2024-03-14T05:54:00.000Z"
date: "2024-03-14"
상태: "Ready"
title: "Supabase EdgeFunction  - Deno 개발환경 꾸미기"
---
![](image1.png)
## 개요

Edge function을 열심히 개발하고 있었는데, 코드 자동완성이나 줄바꿈이나 이런것들이 적용되지 않았습니다. 

Edge function은 Deno를 사용하는데, 이것때문인 것 같긴 했는데 짬이 안나 살펴보지 못하다가 이제서야 살펴보니, 뭔가를 하지 않아서였습니다. 

## Deno 설치

정말 당연하게도 Deno를 먼저 설치해야 합니다. 이런것도 하지 않고 개발을 하고 있었습니다.. 맥 기준으로 설치 방법을 정리하겠습니다. 

### Deno 설치

다음의 명령어를 통해 설치 여부를 먼저 확인해줍니다. 

```bash
deno --version
```

설치가 되어있지 않다면 설치를 진행합니다. [공식사이트](https://docs.deno.com/runtime/manual)에서 자세한 설치법을 확인할 수 있습니다. 

```bash
curl -fsSL https://deno.land/install.sh | sh
```

를 이용해 설치를 해줍니다. 

그 다음, ~/.zshrc 를 수정해 환경변수에 deno 실행 명령어를 등록합니다. 

### Deno VS Code 확장 설치

vscode를 사용하고 있으므로 deno 확장을 설치해줍니다. 

### Deno 확장 활성화 및 설정

확장 설치 후, Deno를 사용할 프로젝트 폴더에 대해 Deno 확장을 활성화해야 합니다. 프로젝트의 루트 디렉토리에 `.vscode` 폴더를 생성하고, 그 안에 `settings.json` 파일을 만들어 다음 설정을 추가합니다.

```json
{
    "deno.enable": true,
    "deno.lint": true,
    "deno.unstable": true
}
```



여기까지 해주고 vs code를 재실행하면 이제 개발환경이 잘 꾸며진 것을 확인할 수 있습니다. 

## 오류

`Uncached or missing remote URL` 오류가 발생할때가 있습니다. 이 오류는 지정된 URL에서 모듈을 캐시할 수 없거나 URL이 잘못되었을 때 발생합니다. 이 오류는 Deno가 모듈을 다운로드하고 캐싱하는 방식과 관련이 있습니다. 오류 메시지는 `https://esm.sh/@supabase/supabase-js@2`로 지정된 모듈을 Deno가 찾지 못했거나 캐시하지 못했다는 것을 나타냅니다. 

### 해결법

Deno는 모듈을 한 번 다운로드하면 로컬에 캐시합니다. 캐시된 모듈이 문제를 일으킬 수 있으므로, 모듈 캐시를 강제로 업데이트해 볼 수 있습니다. 다음 명령어를 사용하여 모듈을 강제로 캐시에서 업데이트하고 다시 로드할 수 있습니다.

```bash
deno cache --reload https://esm.sh/@supabase/supabase-js@2
```

### 캐시 초기화

외부 모듈이 자동으로 캐시되지 않거나 오래된 버전의 캐시가 문제를 일으킬 수 있습니다. Deno 캐시를 갱신하려면, 터미널에서 `deno cache --reload` 명령어를 사용할 수 있습니다.



