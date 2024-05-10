---
IDX: "NUM-178"
tags:
  - Pyenv
  - Homebrew
  - Python
description: "맥북에서 pyenv local 명령어가 먹히지 않는 경우를 해결해보자"
update: "2024-05-10T01:30:00.000Z"
date: "2024-05-09"
상태: "Ready"
title: "Pyenv local 실행을 했는데 파이썬 버전이 바뀌지 않는다"
---
## 문제

[이 링크](https://sharknia.github.io/Apple-Silicon과-pyenv)를 참조해 Pyenv를 설치했습니다. 이후, python 3.11버전을 추가로 설치하고 새로 시작하는 프로젝트에서 python 3.11 버전을 사용하기 위해 pyenv local 명령어를 사용했는데,

```shell
user@MacBook-Pro django % pyenv local 3.11.8
user@MacBook-Pro django % pyenv versions    
  system
  3.9.18
* 3.11.8 (set by /Users/user/Develop/django/.python-version)
user@MacBook-Pro django % python3 --version 
Python 3.9.18
```

python 버전이 바뀌지 않은 것을 확인했습니다. 

## 원인

이는 `pyenv` 셈(shim) 경로 설정이 제대로 되어 있지 않아서 발생하는 문제입니다.

## 해결

.zshrc에 다음의 내용을 추가합니다. 

```shell
# pyenv 설정
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
```

shims는 사용자가 설정한 특정 Python 버전 및 가상환경을 가리키는 링크를 포함한 디렉토리입니다.
이를 통해 python, pip, pytest 등의 명령어를 실행할 때 shims에 설정된 Python 버전을 자동으로 사용하게 됩니다.

다음 명령어로 변경 사항을 적용합니다. 

```shell
source ~/.zshrc
```

pyenv rehash 명령어를 사용해 재해시를 합니다. 

```shell
pyenv rehash
```

이제 다시 pyenv local을 사용하면 제대로 버전이 바뀐것을 확인할 수 있습니다. 

```shell
pyenv local 3.11.8
python3 --version
```



