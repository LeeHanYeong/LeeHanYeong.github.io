---
layout: post
title:  "pyenv와 virtualenv를 사용한 파이썬 개발환경 구성"
categories: ['Python']
---

파이썬으로 프로젝트를 진행한다면 각 프로젝트별로 파이썬 버전과 가상환경을 분리하여 사용할 것을 추천합니다.  
이 포스팅은 파이썬 버전과 가상환경 분리에 `yyuu`의 `pyenv`, `pyenv-virtualenv`라이브러리를 사용하며, `macOS Catalina`또는 `Ubuntu linux(18.04)`환경을 기준으로 설명합니다.

> 2018-01-21 자세한 업데이트 및 Ubuntu설치과정 추가

---

## 설치 전 필요 과정

### macOS

#### `brew`설치

`Homebrew`는 `macOS`를 위한 패키지 관리자입니다.  
[https://brew.sh/index_ko.html](https://brew.sh/index_ko.html)에 접속, 설치 명령어를 터미널에 붙여넣기해서 실행합니다.

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Ubuntu

#### 패키지 관리자 업데이트 및 설치된 패키지 업데이트

```shell
sudo apt update
sudo apt dist-upgrade
```

---

## pyenv, pyenv-virtualenv

### pyenv

[GitHub Repository](https://github.com/pyenv/pyenv)  
여러 프로젝트를 동시에 진행하다보면, 예전의(레거시) 프로젝트에서는 2.7버전을, 새로 시작하는 프로젝트에서는 3.6버전을 사용하는 식으로 다양한 버전을 사용하게 되므로 파이썬 버전을 각각 분리하여 사용해야 하며, 이 때 `pyenv`라는 파이썬 버전 관리 시스템을 사용한다.  

### virtualenv, pyenv-virtualenv

가상환경은 프로젝트별로 설치된 패키지들간의 충돌을 막아주기 위해 필요하다. 파이썬에서는 이를 해결하기 위해 일반적으로 `virtualenv`라는 패키지를 사용하며, 이 포스팅에서는 `pyenv`와 같이 사용할 경우 좀 더 밀접하게 사용가능하도록 나온 `pyenv-virtualenv`를 사용한다.

---

## pyenv, pyenv-virtualenv 설치

### macOS

`brew`를 이용해 `pyenv`와 `pyenv-virtualenv`를 설치합니다

```
brew install pyenv
brew install pyenv-virtualenv
```

### Ubuntu

`pyenv`와 `pyenv-installer`를 쉽게 설치할 수 있도록 `pyenv`의 제작자가 설치 스크립트를 구현해 놓았습니다.  
[pyenv-installer][pyenv-installer]를 참조합니다.

**`Terminal`**  
```shell
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```

> 만약 Ubuntu를 처음 설치했다면 git과 curl이 필요합니다.  
> 설치가 진행되지 않는다면 `sudo apt install git curl`명령어로 `git`과 `curl`패키지를 설치해줍니다.

---

## 설치 후 pyenv관련 설정을 shell설정 파일에 추가

`pyenv`및 `pyenv-virtualenv`가 정상동작 할 수 있도록, 셸의 설정파일에 각 프로그램의 초기화 코드를 추가해야 합니다.

`macOS`와 `Ubuntu`에서는 기본 셸로 `bash`를 사용합니다.  
셸의 설정파일은 `~/.bash_profile`파일을 사용하며, `~/.bashrc`파일을 사용하는 경우도 있습니다. 어떤 설정파일을 사용하는지는 `pyenv`를 설치한 후 터미널에 나타나니, 해당 메시지를 주의하여 보고 아래 과정을 따라합니다.

만약 `zsh`을 쓴다면 `~/.zshrc`에 설정해주어야 합니다. 설치 후 터미널에 나타나는 메시지가 사용하는 셸의 종류에 따라 자동으로 변경됩니다.

아래 메시지는 `Ubuntu`에서 `lhy`사용자가 `pyenv`를 설치 완료 한 후 나타나는 메시지 예제입니다.

**`설치 후 메시지 예제`**
```shell
# Load pyenv automatically by adding
# the following to ~/.bash_profile:

export PATH="/home/lhy/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

### macOS

```shell
➜ vi ~/.bash_profile

# 가장 아래쪽에 아래 문장 추가
export PYENV_PATH=$HOME/.pyenv
if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi
if which pyenv-virtualenv-init > /dev/null; then eval "$(pyenv virtualenv-init -)"; fi
```

### Ubuntu

`macOS`에서와 비슷하나, 설치후 메시지의 `the following to <셸 설정파일 경로>`에 있는 설정파일 경로를 `vim <셸 설정파일 경로>`명령어로 열어준 후 export... ~ eval로 끝나는 3줄을 파일의 마지막에 추가해줍니다.

> 만약 `vim`이 없다면, `sudo apt install vim`명령어로 `vim`패키지를 설치해줍니다.

### 설치 후 pyenv설치 확인

맥과 우분투 모두, 셸의 설정파일에 내용을 추가한 뒤에는 설정 파일을 바꾸었던 해당 터미널 창을 종료하고, 새 터미널 창을 열어줍니다.

새 창에서 `pyenv`라고 명령어를 입력했을 때, 아래와 같은 명령어 목록이 출력되어야 합니다.

```shell
❯ pyenv
pyenv 1.2.13
Usage: pyenv <command> [<args>]

Some useful pyenv commands are:
   commands    List all available pyenv commands
   local       Set or show the local application-specific Python version
   global      Set or show the global Python version
   shell       Set or show the shell-specific Python version
   install     Install a Python version using python-build
   uninstall   Uninstall a specific Python version
   rehash      Rehash pyenv shims (run this after installing executables)
   version     Show the current Python version and its origin
   versions    List all Python versions available to pyenv
   which       Display the full path to an executable
   whence      List all Python versions that contain the given executable

See `pyenv help <command>' for information on a specific command.
For full documentation, see: https://github.com/pyenv/pyenv#readme
```

`Ubuntu`의 경우, 분명히 3줄의 설정내용을 `~/.bash_profile`에 입력했는데도 `pyenv`커맨드를 찾지 못한다는 오류가 나온다면, `~/.bashrc`파일의 맨 마지막에 해당 내용을 적고 터미널을 재시작해봅니다.

---

## pyenv를 이용해 파이썬을 설치 하기 전, 필요 패키지 설치

`pyenv`를 사용해서 파이썬을 설치하기전에, 운영체제별로 반드시 필요한 패키지들이 있습니다. 이 과정을 거치지 않으면 설치한 파이썬이 정상적으로 동작하지 않습니다.  
아래에 필요 패키지 설치 명령어를 정리해 놓았지만, `pyenv`의 버전에 따라 설치해야 하는 패키지 목록이 바뀔 수 있습니다. 최신 명령어는 `pyenv`저장소의 [Common build problems][pyenv-common-build-problems]를 참조합니다.

[pyenv-installer]: https://github.com/yyuu/pyenv-installer
[pyenv-common-build-problems]: https://github.com/yyuu/pyenv/wiki/Common-build-problems

### macOS

```shell
brew install readline xz
```

### Ubuntu

```shell
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev libffi-dev liblzma-dev python-openssl git
```

---

## pyenv로 관리되는 파이썬 설치 후, 시스템 전역에서 사용할 파이썬 설정

### pyenv로 파이썬 설치

pyenv를 이용해 설치 가능한 파이썬 버전을 확인합니다. 연습용 신규 프로젝트라면 파이썬3의 가장 최신 버전을 사용합니다. (2018-01-21시점에서 3.7.4)

```shell
➜ pyenv install --list
# 이후 원하는 버전을 설치한다.
➜ pyenv install 3.7.4
```

### pyenv로 관리되는 파이썬 목록 확인

pyenv로 관리되는 파이썬 버전들을 보려면 아래 명령어를 입력합니다.

```shell
➜ pyenv versions
* system
  3.7.4
```

목록 중 `*`이 붙어있는 항목이 현재 셸에서 사용되고 있는 파이썬 버전입니다.

### 전역에서 사용할 파이썬 설정

설치된 파이썬 목록 중 방금 설치한 `3.7.4`를 전역에서 사용하기 위해 아래 명령어를 입력합니다.

```shell
➜ pyenv global 3.7.4
➜ pyenv versions
  system
* 3.7.4
```

### 현재 사용되는 파이썬 버전 확인

버전 확인 명령어를 입력 시 출력 결과가 아래와 같아야 합니다.

```shell
➜ pyenv version
3.7.4 (set by <파이썬 설치 경로>)

➜ python --version
Python 3.7.4
```

---

## pyenv-virtualenv를 사용한 가상환경 관리

이제 컴퓨터 전역에서 `pyenv`로 관리되는 `3.7.4`버전의 파이썬을 사용합니다. 이번에는 해당 버전의 파이썬을 사용하는 가상환경을 만들어봅니다.

### 가상환경 `sample-env`생성

가상환경을 생성하는 문법은 `pyenv virtualenv <version> <env_name>`입니다.

```
# 3.7.4버전을 기준으로 sample-env라는 환경을 만듭니다.
➜ pyenv virtualenv 3.7.4 sample-env
(생성하면서 Requirement already....라는 메시지가 출력됩니다)
```

> 가상환경의 생성과 삭제는 시스템의 어느 위치에서 실행해도 상관없습니다.

### 가상환경을 프로젝트 폴더에 지정

생성한 가상환경은 프로젝트 폴더 단위로 구분해서 사용합니다. 지금은 아래와 같은 폴더 구조에서 `sample-project`라는 폴더를 프로젝트 폴더로 사용합니다.

**`Structure`**
```
# ~ 는 자신의 홈 폴더를 의미합니다. "cd ~/" 명령어로 이동할 수 있습니다.
~/projects/
  sample-project/   <- 이 폴더를 사용합니다.
```

만약 위 구조가 없다면, 아래 명령어로 해당 폴더구조를 생성하고 이동합니다.

**`Terminal`**
```shell
cd ~/
mkdir projects
cd projects
mkdir sample-project
cd sample-project
```

현재 폴더에 특정 가상환경을 적용할 때는 `pyenv local`명령어를 사용합니다.

```shell
~/projects/sample-project ➜ pyenv local sample-env
# 프롬프트 좌측에 적용된 가상환경명이 표시됩니다
(sample-env) ~/projects/sample-project ➜
```

가상환경이 적용되었는지 `pyenv versions`명령어로 확인해봅니다.

```shell
(sample-env) ~/projects/sample-project ➜ pyenv versions
  system
  3.7.4
  3.7.4/envs/sample-env
* sample-env (set by <.python-version>파일의 위치)
```

위와 같이 `sample-env`에 `*`표시가 되어있으면 적용이 완료된 상태입니다.


### 가상환경의 패키지 목록 확인

처음 생성한 가상환경은 `pip`와 `setuptools`, 2개의 패키지만 설치되어 있어야 합니다. 이를 확인해봅니다.

```
(sample-env) ~/projects/sample-project ➜ pip list
Package    Version
---------- -------
pip        9.0.1
setuptools 28.8.0
```

## 마치며

지금까지 `pyenv`와 `pyenv-virtualenv`를 사용해 파이썬 버전과 가상환경을 분리해 보았습니다.

가상환경은 아주 작은 프로젝트 단위라도 구분해서 사용하시는 것을 추천드리며, 설치 방법 외의 자세한 내용은 [pyenv 공식 사이트](https://github.com/pyenv/pyenv)를 참조하세요.

생성한 가상환경을 이용해 Pycharm에서 Django프로젝트를 설정하는 방법은 [PyCharm과 pyenv를 사용한 Django 개발 환경 설정](/pyenv-python-development-environment-setting)에서 이어서 포스팅합니다.