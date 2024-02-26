---
IDX: "NUM-144"
tags:
  - Python
  - Homebrew
  - Pyenv
description: "pyenv를 통한 python 개발환경 설정"
update: "2024-02-26T05:51:00.000Z"
date: "2024-02-26"
상태: "Ready"
title: "Apple Silicon과 pyenv"
---
## 서론

새로운 프로젝트의 백엔드 언어가 파이썬으로 확정되었습니다. 파이썬 버전은 추후 머신러닝 사용을 감안하여 현재 라이브러리와 가장 호환성이 좋을 것으로 생각되는 3.9버전을 사용하려고 합니다. 다만, 파이썬 3.9는 그다지 최신 버전은 아닙니다. 최근에 타입스크립트를 만지면서 nvm을 좋게 사용한 경험도 있고, 이번에 내친김에 파이썬에서도 버전관리를 해주는 툴인 pyenv를 사용해서 환경을 꾸며보기로 했습니다. 

## 설치

설치는 아주 간단합니다. homebrew는 미리 설치되어 있어야 합니다. 

```bash
brew install pyenv
```

pyenv를 설치해주고,

```bash
pyenv install --list | grep 3.9
```

이 명령어를 사용하면 현재 설치 가능한 파이썬 3.9버전의 리스트를 확인할 수 있습니다. 현재 3.9의 최신 버전은 3.9.18로 보입니다. 해당 버전을 설치해줍니다. 

```bash
pyenv install python 3.9.18
```

간단하게 파이썬이 설치.. 되지 않았습니다! 오류가 납니다. 

![](image1.png)
## 해결

챗지피티한테 물어보거나 검색을 해보는 등 여러가지 방법을 써보면 많은 방법이 나옵니다. zshrc에 여러 명령어를 export 해주거나 특정 라이브러리를 설치해주거나.. 또는 파이썬 버전을 바꾸던가 

여러가지 방법을 시도했지만 잘 되지 않았습니다. 여전히 같은 오류가 발생합니다. 

문제는 홈브루에 있었습니다. 

### Homebrew 버전 확인 및 최신 버전 설치

#### Homebrew 버전 확인

아래 코드로 내 homebrew를 확인할 수 있습니다. 

```bash
which brew
```

이 코드를 실행했을 때에,

```bash
/usr/local/bin/brew
```

가 출력된다면 구버전의 Homebrew가 설치되어 있는 것입니다. 

#### Homebrew 구버전 삭제

다음의 명령어로 Homebrew를 삭제할 수 있습니다. 

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"
```

해당 코드를 실행한 뒤에도 `which brew` 를 실행해보면 여전히 위치가 나옵니다. 이 경우에는 삭제 코드를 다시 실행해주면 됩니다. 정상적으로 삭제 된다면 `command not found` 가 발생합니다. 

#### Homebrew 최신 버전 설치

아래의 명령어로 최신 버전을 설치할 수 있습니다. 

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

설치가 완료되면 다음과 같은 문구가 나타납니다. 버전에 따라 다를 수 있습니다. 

```bash
==> Installation successful!

==> Homebrew has enabled anonymous aggregate formulae and cask analytics.
Read the analytics documentation (and how to opt-out) here:
  https://docs.brew.sh/Analytics
No analytics data has been sent yet (nor will any be during this install run).

==> Homebrew is run entirely by unpaid volunteers. Please consider donating:
  https://github.com/Homebrew/brew#donations

==> Next steps:
- Run these two commands in your terminal to add Homebrew to your PATH:
    (echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> /Users/UserName/.zprofile
    eval "$(/opt/homebrew/bin/brew shellenv)"
- Run brew help to get started
- Further documentation:
    https://docs.brew.sh

```

Next steps를 보니, 다음의 명령어를 실행하라고 합니다. 두 줄의 명령어를 실행하고 `which brew` 를 다시 실행해보면

```bash
/opt/homebrew/bin/brew
```

이렇게 표시되면 최신 버전의 brew가 설치된 것입니다. 

### 다시 pyenv, python 설치

다시 위로 돌아가서, pyenv과 python을 설치해주면 이번에는 오류가 발생하지 않습니다. 해~결~

## pyenv 설치 후

설치된 파이썬 버전 리스트를 확인할 수 있습니다. 

```bash
pyenv versions
```

현재 사용중인 파이썬 버전을 확인할 수 있습니다. 

```bash
pyenv version
```

사용중인 파이썬을 설치한 버전으로 바꿔줍니다. 

```bash
pyenv global 3.9.18
```

현재 디렉토리에서만 사용할 파이썬 버전을 지정할 수 있습니다.

```bash
pyenv local 3.9.18
```

쉘에만 적용할수도 있습니다. 

```bash
pyenv shell 3.9.9
```

## 참고

[https://www.lainyzine.com/ko/article/how-to-install-homebrew-for-m1-apple-silicon/](https://www.lainyzine.com/ko/article/how-to-install-homebrew-for-m1-apple-silicon/)

