---
layout: post
title:  "runserver 실행시 마다 DB내용 백업"
categories: ['Django', 'Python']
---

로컬에서 개발 시 사용하던 DB의 테스트 데이터를 잃고 싶지 않아서 `db.sqlite3`파일을 클라우드 폴더에서 링크하는 방법을 사용했는데, 이번에 `PostgreSQL`전용 필드를 사용하다보니 로컬 DB파일 자체를 백업하는 방식을 사용할 수 없어 자주 사용하는 `runserver`커맨드 실행시마다 자동으로 DB내용을 백업하도록 만들어 보았습니다.

## Command를 등록할 app생성, management및 commands패키지 생성

Command만을 등록할 app을 생성합니다. app의 이름은 `scripts`를 사용합니다.

**`Terminal`**
```
python manage.py startapp scripts
```

그리고 생성된 `scripts`패키지에 아래와 같은 구조를 만듭니다.

**`Structure`**
```
<django project directory>/
  scripts/
    management/
      commands/
        __init__.py
        dump.py
        runserver.py
    __init__.py
  __init__.py    
```

## INSTALLED_APPS에 scripts app추가

`scripts` app에 새로 정의한 커맨드가 기존 커맨드를 덮어씌우게 하기 위해 `INSTALLED_APPS`의 윗 부분에 앱을 추가합니다.

**`settings.py`**
```python
INSTALLED_APPS = [
    'scripts',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    ...
]
```

## 파일을 백업하는 dump.py모듈 작성

**`scripts/management/commands/dump.py`**
```python
import subprocess

from django.conf import settings
from django.core.management import BaseCommand


class Command(BaseCommand):
    CMD = 'python manage.py dumpdata ' \
          '<공백으로 구분된 app이름> ' \
          '--indent=4 > ' \
          '<db파일을 저장할 경로>/db.json'

    def handle(self, *args, **options):
        process = subprocess.Popen(
            self.CMD,
            cwd=settings.BASE_DIR,
            shell=True,
            stdout=subprocess.PIPE
        )
        process.communicate()
```

`CMD`에 할당된 문자열을 셸에서 실행시킵니다.

## runserver Command 재정의

**`scripts/management/commands/runserver.py`**
```python
import subprocess

from django.conf import settings
from django.core.management.commands import runserver


class Command(runserver.Command):
    CMD = 'python manage.py dump'

    def handle(self, *args, **options):
        process = subprocess.Popen(
            self.CMD,
            cwd=settings.BASE_DIR,
            shell=True,
            stdout=subprocess.PIPE
        )
        process.communicate()
        super().handle(*args, **options)
```

원래 `runserver`커맨드가 하던 작업 앞에 이전에 정의한 `dump`명령어를 실행하도록 합니다.

## 마치며

`python manage.py runserver`또는 `python manage.py dump`를 실행할 때 마다 지정한 경로로 현재 DB파일의 내용을 백업합니다.

여러 토이 프로젝트를 진행하다보면 로컬 DB의 내용을 보관하는 작업이 조금 귀찮은데, 이번에는 기존 Command를 재정의하는 방법을 사용해봤습니다.