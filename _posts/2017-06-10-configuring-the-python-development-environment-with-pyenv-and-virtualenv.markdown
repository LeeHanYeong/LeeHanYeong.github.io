---
layout: post
title:  "pyenv와 virtualenv를 사용한 파이썬 개발환경 구성"
categories: ['Python', 'PyCharm']
---
파이썬으로 프로젝트를 진행한다면 각 프로젝트별로 파이썬 버전과 가상환경을 분리하여 사용할 것을 추천한다.  
이 포스팅은 `yyuu`님의 `pyenv`, `pyenv-virtualenv`라이브러리를 사용하며, `macOS Sierra`환경에서 진행하였다. 또한 우분투 리눅스에서의 설치방법도 대략적으로 기술한다.

---

### pyenv

[GitHub Repository](https://github.com/pyenv/pyenv)  
여러 프로젝트를 동시에 진행하다보면, 예전의(레거시) 프로젝트에서는 2.7버전을, 새로 시작하는 프로젝트에서는 3.6버전을 사용하는 식으로 다양한 버전을 사용하게 되므로 파이썬 버전을 각각 분리하여 사용해야 하며, 이 때 `pyenv`라는 파이썬 버전 관리 시스템을 사용한다.  

### virtualenv, pyenv-virtualenv

가상환경은 프로젝트별로 설치된 패키지들간의 충돌을 막아주기 위해 필요하다. 파이썬에서는 이를 해결하기 위해 일반적으로 `virtualenv`라는 패키지를 사용하며, 이 포스팅에서는 `pyenv`와 같이 사용할 경우 좀 더 밀접하게 사용가능하도록 나온 `pyenv-virtualenv`를 사용한다.

---

### pyenv, pyenv-virtualenv 설치

#### macOS

```
brew install pyenv
brew install pyenv-virtualenv
```

#### Ubuntu

```
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```

[pyenv-installer][pyenv-installer]를 참조한다.

---

### 설치 후 pyenv관련 설정을 shell설정 파일에 추가

> zsh을 쓴다면 ~/.zshrc에 설정해준다.

#### macOS

```
➜ vi ~/.bash_profile

# 가장 아래쪽에 아래 문장 추가
export PYENV_ROOT=/usr/local/var/pyenv
if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi
if which pyenv-virtualenv-init > /dev/null; then eval "$(pyenv virtualenv-init -)"; fi
```

#### Ubuntu

```
➜ vi ~/.bashrc

# 가장 아래쪽에 아래 문장 추가
export PATH="~/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

---

#### shell설정을 바꾸었으면, 해당 설정을 현재 shell에 적용

터미널을 재시작하거나 `source ~/.bashrc`또는 `source ~/.bash_profile`를 입력.  
`zsh`을 사용할 경우 `source ~/.zshrc` 실행

---

#### pyenv를 이용해 파이썬을 설치 하기 전, 필요 패키지 설치

필수 패키지를 설치하지 않을 경우 파이썬이 설치되지 않거나, 오작동을 일으킨다.  
[Common build problems][pyenv-common-build-problems]를 참조


[pyenv-installer]: https://github.com/yyuu/pyenv-installer
[pyenv-common-build-problems]: https://github.com/yyuu/pyenv/wiki/Common-build-problems


---

### 설치 가능한 파이썬 버전 보기

pyenv를 이용해 설치 가능한 파이썬 버전을 확인한다. 일반적인 신규 프로젝트에서는 파이썬3의 가장 마지막버전을 사용한다. (2010-06-11시점에서 3.6.1)

```
➜ pyenv install --list
# 이후 원하는 버전을 설치한다.
➜ pyenv install 3.6.1
```

---

### pyenv-virtualenv를 사용한 가상환경 관리

#### 컴퓨터 전체에서 사용할 파이썬 환경 설정

```
➜ pyenv global 3.6.1
➜ pyenv versions
  system
* 3.6.1 (set by /usr/local/var/pyenv/version)
```

#### 가상환경 생성

가상환경 생성/삭제는 어느위치에서 실행해도 관계없다.

```
# pyenv virtualenv <version> <env_name>
➜ pyenv virtualenv 3.6.1 inaina
```

#### 가상환경을 현재 폴더에서 사용하도록 지정

일반적으로 프로젝트 폴더 (git저장소 단위 폴더)에 지정한다.  
`pyenv local`명령어를 사용하며, 실행결과 현재 위치에 `.python-version`이라는 파일이 생기며, 내용은 입력한 env_name이다.

```
# pyenv local <env_name>
➜ cd ~/projects/inaina/
➜ pyenv local inaina
(inaina) ➜ # 프롬프트 좌측에 가상환경명 표시
```

#### 현재 가상환경이 적용되어있는지 확인

```
(inaina) ➜ pyenv versions
  system
  3.6.1/envs/inaina
* inaina (set by /Users/lhy/projects/inaina/.python-version)
```

`pip list`입력 시 최소사항 (`setuptools`, `pip`)만 설치되어있다면 정상적으로 가상환경이 생성되고 적용된 상태이다.  
`pyenv`는 디렉토리 이동시마다 해당 폴더에 `.python-version`파일의 존재여부에 따라 자동으로 가상환경을 적용/해제시켜준다.
