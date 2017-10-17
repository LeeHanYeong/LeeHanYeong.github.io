---
layout: post
title:  "PyCharm과 pyenv를 사용한 Django 개발 환경 설정"
categories: ['Django', 'Python']
---

## 새 프로젝트 설정 시 작업 모음

`pyenv`, `pyenv-virtualenv`이 설치되어 있다는 가정 하에 실행한다. `pyenv`를 설치하는 방법은 아래에서 확인한다.

**홈 폴더의 `projects`폴더에서 `instagram`프로젝트를 생성하기 위한 과정이다**

---

### 1. 터미널 명령어 실행

```shell
# <home directory>/projects/ 로 이동
cd ~/projects

# 프로젝트를 감싸는 컨테이너 폴더 생성 및 이동
mkdir instagram-project
cd instagram-project

# pyenv 가상환경 생성, 적용
pyenv virtualenv 3.6.2 instagram-env
pyenv local instagram-env

# Django 및 서드파티 패키지 설치
pip install django ipython django_extensions

# Django project생성
django-admin startproject instagram

# .gitignore설정
wget https://gist.githubusercontent.com/LeeHanYeong/8758517113be32dd2e885fef81c4a96e/raw/00727ac4af42834e6282df05c61606aa396d5b9c/.gitignore
```

### 2. PyCharm에서의 설정

`PyCharm`에서 프로젝트 컨테이너 폴더 (위의 경우 `instagram-project`폴더)를 열어준다.

현재 프로젝트의 구조는 다음과 같다

```shell
# 프로젝트 컨테이너 폴더
instagram-project/
  # Django 프로젝트 폴더
  instagram/
    # 설정파일 모음 패키지
    instagram/
      __init__.py
      ...
```
#### Python interpreter설정

1. `Preferences`로 진입해 `Project: instagram_project`탭 클릭
2. `Project Interpreter`선택
3. 우측 창의 `Project Interpreter:` 우측의 드롭다운 메뉴를 클릭
4. 가장 아래로 스크롤하여 `Show All`선택
5. 좌측 아래의 `+`클릭 -> `Add local`선택
6. 인터프리터로 사용될 파이썬 선택
  - `macOS`는 `/usr/local/var/pyenv/versions/instagram-env/bin/python`선택
  - `Ubuntu`는 `/Users/<자신의 홈폴더명>/.pyenv/versions/instagram-env/bin/python`선택
7. 추가된 인터프리터 선택 후 `OK`클릭
8. 나온 창에서 `OK`클릭.

#### `Sources root`설정

Django프로젝트 폴더(`instagram-project/instagram/`)에 우클릭 -> `Mark Directory as` -> `Sources Root`선택

#### 기본 설정파일 모음 패키지 이름 `config`로 변경

설정파일 모음 패키지(`instagram-project/instagram/instagram/`)에 우클릭 -> `Refactor` -> `Rename`선택

`Search for references`와 `Search in comments and strings`두 항목을 모두 체크 한 후, 값을 `config`로 변경 후 `Refactor`버튼 클릭

아래쪽의 `Do Refactor`버튼 클릭

이제 프로젝트 구조는 다음과 같다

```shell
# 프로젝트 컨테이너 폴더
instagram-project/
  # Django 프로젝트 폴더 (Sources Root)
  instagram/
    # 설정파일 모음 패키지
    config/
      __init__.py
      ...
```

이상없이 진행했다면 `runserver`가 정상적으로 동작하는지 테스트한다.

```shell
cd ~/projects/instagram-project/instagram
./manage.py runserver
```

브라우저로 `localhost:8000`으로 접속해 이상없음을 확인한다.


### 3. git저장소 설정

```shell
cd ~/projects/instagram-project/
git init
git add -A
# 아래 명령어에서 추가되면 안 되는 파일이 있는지 확인 (db.sqlite3등)
git diff --staged
git commit -m 'First commit'

# 리모트 저장소 추가 후
git push origin master
```

---

## pyenv

프로젝트별로 파이썬 버전을 따로 관리할 수 있도록 도와주는 라이브러리.  

여러 프로젝트를 동시에 진행하다보면, 어떤 프로젝트에서는 2.7을, 어떤 프로젝트에서는 3.5를 사용하는 식으로 다양한 버전을 사용할 수 있기 때문에 `pyenv`를 사용해서 파이썬 버전을 관리한다.

## virtualenv, pyenv-virtualenv
  
프로젝트별로 설치된 패키지들 간 충돌을 막아주기 위해 분리된 파이썬 환경을 만들어주는 `virtualenv`라는 패키지를 사용하며, `pyenv`를 사용할 경우, 가상환경을 `pyenv`와 좀 더 밀접하게 사용 할 수 있도록 나온 `pyenv-virtualenv`패키지를 사용한다.

---

### pyenv install

#### 패키지 설치

`Ubuntu`는 `pyenv-installer`에 나와있는 방법대로, `macOS`는 `brew`를 사용해서 설치한다.

**macOS**  
 
```
brew install pyenv
brew install pyenv-virtualenv
```

**Ubuntu**

<https://github.com/yyuu/pyenv-installer>  

```
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```

#### 설치 후 pyenv관련 설정을 shell설정에 추가  

`zsh`을 쓴다면 `~/.zshrc`에 추가한다.

**macOS**  

```
vi ~/.bash_profile

# 가장 아래쪽에 아래 문장 추가
export PYENV_ROOT=/usr/local/var/pyenv
if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi
if which pyenv-virtualenv-init > /dev/null; then eval "$(pyenv virtualenv-init -)"; fi
```

**Ubuntu**  

```
vi ~/.bashrc

# 가장 아래쪽에 아래 문장 추가
export PATH="~/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

#### shell설정을 바꾼 이후, 해당 설정을 현재 shell에 적용

터미널을 재시작하거나 `source ~/.bashrc` 또는 `source ~/.bash_profile`을 실행

#### pyenv를 이용해 파이썬을 설치 하기 전, 필요 패키지 설치

<https://github.com/yyuu/pyenv/wiki/Common-build-problems>

#### 설치 가능한 파이썬 버전 보기

```
pyenv install --list
```

#### 파이썬 3.5.3설치

```
pyenv install 3.5.3
```

---

### pyenv-virtualenv를 사용한 가상환경 관리

#### 컴퓨터 전체에서 사용할 파이썬 환경 설정

```
pyenv global <version or env_name>
```

#### 가상환경 생성

```
pyenv virtualenv <version> <env_name>
```

#### 가상환경을 해당 폴더에서 사용하도록 지정

```
pyenv local <env_name>
```

#### 현재 가상환경이 적용되어있는지 확인

```
pyenv version
```

---

## pip (Pip Installs Packages)

파이썬 패키지 관리자. 파이썬 패키지를 쉽게 설치하고 관리해준다.

### 관련 명령어

```
pip list
pip uninstall
pip install
```

### requirements가 있을 경우 설치

```
pip install -r requirements.txt
```

### 에러 해결

#### ImportError: No module named 'packaging'

```
pip install --upgrade pip
```
실행 후 다시 `pip install`

#### Could not import setuptools which is required to install from a source distribution.

```
pip install -U setuptools
```
실행 후 다시 `pip install`