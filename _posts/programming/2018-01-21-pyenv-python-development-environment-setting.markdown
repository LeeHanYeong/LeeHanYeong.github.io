---
layout: post
title:  "PyCharm과 pyenv를 사용한 Django 개발 환경 설정"
categories: ['Django', 'Python', 'PyCharm']
---

> 이 포스팅은 **[pyenv와 virtualenv를 사용한 파이썬 개발환경 구성](/configuring-the-python-development-environment-with-pyenv-and-virtualenv)** 포스팅 에서 이어집니다.
>
> 2018-01-21 | 이전 개발환경 구성 포스팅에서 연결되도록 내용 수정

PyCharm에서 pyenv의 가상환경을 사용한 Django 개발 환경을 설정하려면 많은 과정이 필요합니다. 이번 포스팅에서는 생성한 가상환경을 PyCharm에서 사용하도록 적용하고, 적절한 Django프로젝트의 폴더 구조를 구성하며, 로컬 Git저장소를 초기화하고 초기 설정이 완료된 프로젝트 커밋을 만듭니다.

## 사용하는 파이썬 환경 및 프로젝트 구조

이전 포스팅에서 아래와 같은 프로젝트 폴더를 생성했습니다.

```
~/projects/
  sample-project/      <- 프로젝트 전체 컨테이너 폴더, sample-env라는 가상환경을 사용
```

프로젝트 컨테이너 폴더는 `~/projects/sample-project`이며, Django프로젝트 폴더의 이름은 `app`, Django프로젝트의 설정 패키지명은 `config`를 사용한다. 여기서 설정하는 프로젝트 구조는 튜토리얼 등에서 사용하는 구조보다 한 단계를 더 가지며, Django코드와 프로젝트 설정파일들을 분리할 수 있도록 도와줍니다.

구성 완료 후에는 아래와 같은 구조를 가지게 됩니다.

```
~/projects/
  sample-project/      <- 프로젝트 전체 컨테이너 폴더, sample-env라는 가상환경을 사용
    .git               <- 로컬 Git 폴더
    .gitignore
    app/               <- Django프로젝트 폴더
      config/          <- Django프로젝트의 설정 패키지. settings, urls, wsgi모듈을 포함
        __init__.py
        settings.py
        urls.py
        wsgi.py
      manage.py
    requirements.txt   <- 설치할 pip패키지 목록 파일
```

---

## 프로젝트 폴더로 이동 및 Django프로젝트 생성, Django프로젝트 폴더의 이름 변경

```shell
# 프로젝트의 컨테이너 폴더는 이미 생성되어 있으며, 가상환경도 적용된 상태입니다.
cd ~/projects/sample-project

# Django 설치
pip install django

# Django project생성
# 프로젝트명으로 설정 패키지 명이 결정되므로, startproject에 전달되는 프로젝트명으로 config를 사용합니다.
django-admin startproject config

# Django project폴더의 이름 변경
# Django 프로젝트(웹 애플리케이션) 관련된 코드만이 모여있다는 의미로 "app"라는 이름을 붙입니다.
mv config app

# 설치된 pip패키지 목록을 requirements.txt에 남기기
pip freeze > requirements.txt
```

여기까지 진행하면 `.git`폴더와 `.gitignore`파일을 제외한 전체 프로젝트 구조가 완성됩니다. 위의 구성 완료 후 구조와 같은지 비교해봅니다.

## PyCharm 설정

### 폴더 열기

`PyCharm`에서 프로젝트 컨테이너 폴더 (위의 경우 `sample-project`폴더)를 열어줍니다.

### Python interpreter설정

이 프로젝트에서 사용하는 코드를 어떤 파이썬을 사용해서 해석할 지 정해주는 과정입니다.

1. PyCharm의 설정(Preferences)로 진입해 `Project: sample-project`탭 클릭
  - Ubuntu의 경우 `File -> Settings (단축키: Ctrl+Alt+s)`를 클릭
  - macOS의 경우 `PyCharm -> Preferences (단축키: Cmd+,)`를 클릭
2. `Project Interpreter`선택
3. 우측 창의 `Project Interpreter:` 우측의 드롭다운 메뉴를 클릭
4. 가장 아래로 스크롤하여 `Show All`선택
5. 초록색 `+`버튼 클릭 -> `Add local`선택
6. 새로 열린 창(Add Local Python Interpreter)에서 Virtualenv Environment탭이 선택되어있는지 확인
7. Existing environment 라디오버튼 클릭
8. **이미 현재 가상환경이 적용되어있다면 9, 10번은 진행하지 않아도 된다. 잘 모르겠으면 그냥 모두 진행**
9. Interpreter: 항목의 가장 우측에 있는 `...`버튼 클릭
10. 인터프리터로 사용될 파이썬 경로 선택
  - `macOS`는 `/usr/local/var/pyenv/versions/sample-env/bin/python`선택
  - `Ubuntu`는 `/Users/<자신의 홈폴더명>/.pyenv/versions/sample-env/bin/python`선택
11. 추가된 인터프리터 선택 후 `OK`클릭
12. 나온 창에서 `OK`클릭.

### `Sources root`설정

이 프로젝트에서 해석할 파이썬 애플리케이션의 최상단을 지정합니다. Django프로젝트는 프로젝트 루트 폴더의 한 단계 아래인 `sample-project/app`폴더에 있으므로 **반드시** 해당 폴더를 소스 루트로 지정해야 합니다.

좌측의 프로젝트 구조에서 Django프로젝트 폴더(`sample-project/app/`)에 우클릭 -> `Mark Directory as` -> `Sources Root`선택

## Git저장소 설정

로컬 Git저장소를 초기화하고, Git으로 관리되지 않을 파일 목록인 `.gitignore`파일을 가져옵니다. 제가 작성한 일반적인 Django프로젝트에서 사용하는 파일을 `wget`을 사용해 웹에서 복사해옵니다.

```shell
# 다른 경로에 있다면 프로젝트 폴더로 이동
cd ~/projects/sample-project

# 로컬 git저장소 초기화
git init

# 웹에서 .gitignore를 받아와 로컬에 저장
wget https://gist.githubusercontent.com/LeeHanYeong/8758517113be32dd2e885fef81c4a96e/raw/00727ac4af42834e6282df05c61606aa396d5b9c/.gitignore

# 현재까지의 작업내역을 commit
git add -A
git commit -m 'First commit'
```

위 과정을 진행하면 프로젝트에 `.git`폴더와 `.gitignore`파일이 추가됩니다. 리눅스 파일시스템에서 `.`으로 시작하는 파일은 숨김파일이므로, 터미널에서 `ls -al`명령어를 입력해야 파일을 확인할 수 있습니다.

## 마치며

프로젝트가 정상적으로 동작하는지 `manage.py`모듈이 있는 위치에서 `python manage.py runserver`를 실행하고, 웹 브라우저에서 `localhost:8000`에 접속해봅니다. 환영 메시지가 출력되어야 합니다.

Django를 처음 시작한다면, [Django Girls Tutorial](https://tutorial.djangogirls.org/ko/)을 따라해보는걸 추천드립니다. 여기서는 이미 가상환경과 코드 에디터 설정, Django설치, Django프로젝트 생성을 끝낸 상태이므로 [Django 모델](https://tutorial.djangogirls.org/ko/django_models/) 파트부터 진행하시면 됩니다.

로컬에서 초기화하고 하나의 커밋을 생성한 Git저장소는 [GitHub](https://github.com/)과 같은 서비스를 사용해 리모트 저장소를 만들고 `push`해 프로젝트를 잃어버리지 않도록 관리하는 것을 권장합니다.