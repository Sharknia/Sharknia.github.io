---
IDX: "NUM-246"
tags:
  - Python
description: "Ruff에 대해 알아보고 사용해보자"
update: "2025-02-20T05:34:00.000Z"
date: "2025-02-20"
상태: "Ready"
title: "Ruff"
---
![](image1.png)
## 서론

[uv](https://sharknia.github.io/uv-간단-소개-및-적용)를 조사하고 알고 하다가 Ruff의 존재도 알게 되었습니다.

현재 회사의 팀에서는 나머지 백엔드 개발자는 파이참을 주력으로, 저는 Cursor를 사용하고 있고 포맷팅이나 린팅에 대한 팀의 컨벤션 통일이 안되어있어 Ruff를 제안하면 좋겠다고 생각이 들어 알아보게 됐습니다. 

## Ruff란?

Ruff는 Python 코드를 빠르고 효율적으로 분석 및 포맷팅해주는 도구입니다. Rust로 개발되어 기존의 Python 기반 린터보다 훨씬 빠른 속도를 자랑합니다. Ruff는 Black보다 최대 100배 빠르게 실행됩니다. 또한 여러 린터와 포매터의 기능을 하나로 통합하여 코드 품질을 관리할 수 있습니다.

- Black → 코드 스타일 포매팅

- isort → import 정리

- Flake8 → 린팅 (PEP 8 준수 검사 등)

- mypy 일부 기능 → 타입 검사 (제한적)

이런 기능들을 Rust로 빠르게 동작하도록 구현해서, Python 환경에서 가장 가벼우면서도 강력한 린터 & 포매터 역할을 합니다.

덕분에 Black + isort + Flake8 조합을 쓰던 사람들이 Ruff 하나로 통합해서 쓰는 경우가 많아졌습니다.

## 설치

Ruff는 pip로 간편하게 설치할 수 있습니다.

```bash
pip install ruff
```

물론 uvx(uv의 전역 설치), pipx, brew 등 여러가지 설치 옵션을 추가로 제공합니다. 

## 사용법

터미널에서 Ruff를 실행하는 기본 명령어는 다음과 같습니다.

```bash
ruff check .
```

해당 명령어는 현재 디렉터리 내의 모든 파일을 검사합니다. 자동 수정 기능은 --fix 옵션을 사용하여 적용할 수 있습니다.

```bash
ruff check . --fix
```

또 다음의 명령어를 통해 프로젝트 전체의 포맷을 조정할 수 있습니다. 

```bash
ruff format .
```

속도가 정말 빠릅니다. 

## VSCode에서 사용하기

VSCode에서는 PIP 설치 없이도 Ruff 확장을 설치하여 사용할 수 있습니다. 확장을 설치한 후, settings.json에 다음과 같이 설정하면 편리합니다.

```json
  "[python]": {
      "editor.defaultFormatter": "charliermarsh.ruff",
      "editor.formatOnSave": true,
      "editor.codeActionsOnSave": {
          "source.fixAll": "explicit",
          "source.organizeImports": "explicit"
      }
  },
```

이렇게 설정하면 코드 편집 시 자동으로 Ruff 검사가 실행되어 코드 품질을 유지할 수 있습니다.

- source.organizeImports: 자동으로 import를 정리합니다. 

- source.fixAll: 저장시 자동 포맷팅을 활성화 합니다. 

## Pycharm에서 사용하기

### 외부 도구(External Tools) 등록

PyCharm에서는 외부 도구(External Tools) 설정을 통해 Ruff를 연동할 수 있습니다.

1. 설정 → Tools → External Tools 로 이동합니다.

1. + 버튼을 클릭하여 새로운 도구를 추가합니다.

1. 다음과 같이 설정합니다.

    - Name: Ruff

    - Program: ruff (또는 ruff 명령어의 전체 경로)

    - Arguments: check $ProjectFileDir$

    - Working directory: $ProjectFileDir$

이후 PyCharm 메뉴에서 Tools > External Tools > Ruff를 실행하면 프로젝트 전반에 대해 린팅이 수행됩니다.

### 감시자(Watcher)를 활용한 Pycharm에서의 Ruff 활용

PyCharm의 File Watcher 기능을 활용하면, 파일이 저장될 때마다 자동으로 Ruff가 실행되어 실시간으로 코드 품질을 유지할 수 있습니다.

1. 설정 → Tools → File Watchers 에서 새 Watcher를 추가합니다.

1. File Type: Python

1. Scope: 현재 프로젝트 또는 특정 디렉터리

1. Program: ruff

1. Arguments: check $FileName$ –fix (또는 필요한 옵션)

1. Output Paths: 빈 칸으로 두거나, 필요에 따라 지정합니다.

이렇게 설정하면, 파일 저장 시 자동으로 Ruff가 실행되어 코드 컨벤션 위반 사항을 즉시 수정할 수 있습니다.

## Ruff의 기본 설정 파일 위치 

macOS와 Linux에서 Ruff는 설정 파일을 `~/.config/ruff/ruff.toml` 경로에서 찾으며, 이는 XDG\_CONFIG\_HOME 사양을 따릅니다. XDG\_CONFIG\_HOME 환경 변수가 설정되어 있지 않더라도, ruff는 기본적으로 ~/.config/ruff/ruff.toml 경로에서 전역 설정 파일을 찾습니다. 이는 XDG Base Directory 사양에 따라, XDG\_CONFIG\_HOME이 설정되지 않은 경우 기본값으로 $HOME/.config를 사용하기 때문입니다. 

따라서, XDG\_CONFIG\_HOME을 별도로 설정하지 않아도 ruff는 ~/.config/ruff/ruff.toml 파일을 자동으로 인식합니다.

Windows에서는 ~\AppData\Roaming\ruff\ruff.toml 경로를 사용합니다. 

프로젝트별 설정 파일이 없을 경우, Ruff는 최후의 수단으로 이 사용자별 설정 파일을 참조합니다.

## **팀 단위 코드 컨벤션 통일**

### **pyproject.toml 또는 ruff.toml을 사용**

모든 팀원이 동일한 린터 설정을 공유할 수 있도록 프로젝트 루트에 설정 파일을 배치합니다. 이 파일에 린팅 및 포매팅 규칙, 예외 사항, 플러그인 설정 등을 정의하면 IDE에 관계없이 동일한 룰을 적용할 수 있습니다.

```toml
[lint]
select = ["E", "F", "W", "I"]  # E: 스타일, F: 오류, W: 경고, I: import 정리

[format]
line-length = 88  # 줄 길이 설정 (Black과 동일)
indent-style = "space"  # 들여쓰기 스타일 (space / tab)
```

팀 전체가 동일한 기준으로 코드를 검사하고 포맷팅 할 수 있으며, CI/CD 파이프라인에 해당 설정 파일을 반영하여 머지 전 코드 품질 검증을 수행할 수도 있습니다. 

PyCharm 또는 VSCode에서 Ruff를 실행할 때, 설정 파일을 자동으로 참조하도록 명령어를 구성할 수 있습니다.

## 참조

[https://docs.astral.sh/ruff/](https://docs.astral.sh/ruff/)



