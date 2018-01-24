---
layout: post
title:  "Django에서 비밀 값(secrets) 관리하기"
categories: ['Django']
---

장고에서 사용하는 AWS 시크릿 코드, 장고 시크릿 키 등의 비밀 값들은 프로젝트 코드에 포함되면 안됩니다. 이러한 비밀 값들을 별도의 JSON파일로 보관하고, 해당 값들을 장고에서 불러오는 방법에 대해 알아봅니다.

## 프로젝트 구조

제 장고 프로젝트는 아래와 같은 구조를 갖습니다. 장고 튜토리얼과는 달리, 장고 코드를 감싸고 있는 프로젝트 컨테이너 폴더가 하나 더 존재하며 장고 코드의 상위폴더에 장고와는 관계없으나 프로젝트를 구성하기 위한 여러 파일들이 위치해있습니다.

예제에서 프로젝트명은 `sample-project`, 장고의 설정 패키지는 `config`를 사용합니다. 프로젝트 구성법에 대해서는 **[PyCharm과 pyenv를 사용한 Django 개발 환경 설정](/pyenv-python-development-environment-setting)**을 참조해주세요

```
~/projects/
  sample-project/      <- 프로젝트 전체 컨테이너 폴더, sample-env라는 가상환경을 사용
    .git               <- 로컬 Git 폴더
    .gitignore
    secrets.json       <<- 비밀 값들을 모아놓은 JSON파일, 이 포스팅에서 새로 만들 파일
    django/            <- Django프로젝트 폴더
      config/          <- Django프로젝트의 설정 패키지. settings, urls, wsgi모듈을 포함
        __init__.py
        settings.py
        urls.py
        wsgi.py
      manage.py
    requirements.txt   <- 설치할 pip패키지 목록 파일
```

---

## secrets.json파일 구성

### 파일 생성

프로젝트 컨테이너 폴더에서 JSON파일을 만들어줍니다.

**`Terminal`**
```shell
cd ~/projects/sample-project
touch secrets.json
```

### 파일 내용 편집

vim을 사용하거나 PyCharm에서 편집합니다. 이 때, JSON파일의 키는 실제 장고 프로젝트의 설정모듈에서 사용할 이름을 그대로 적습니다.

**`~/projects/sample-project/secrets.json`**
```json
{
  "SECRET_KEY": "<Django secret key>"
}
```

## settings.py에서 해당 파일 불러와 동적으로 속성 할당

일반적으로 settings.py에 설정을 할당할때는 직접 `KEY = VALUE`로 속성값을 지정합니다. 여기서는 파이썬의 내장함수 `setattr()`을 이용해 JSON파일에 있는 key-value를 동적으로 settings모듈에 할당해봅니다.

```python
import sys
import json

BASE_DIR = ...
# 장고 BASE_DIR보다 상위의 프로젝트 컨테이너 폴더를 ROOT_DIR로 지정
ROOT_DIR = os.path.dirname(BASE_DIR)
# secrets.json의 경로
SECRETS_PATH = os.path.join(ROOT_DIR, 'secrets.json')
# json파일을 파이썬 객체로 변환
secrets = json.loads(open(SECRETS_PATH).read())

# json파일은 dict로 변환되므로, .items()를 호출해 나온 key와 value를 사용해
# settings모듈에 동적으로 할당
for key, value in secrets.items():
    setattr(sys.modules[__name__], key, value)
```

## .gitignore에 secrets.json을 관리되지 않도록 추가

.gitignore파일에 secrets.json을 추가해서 비밀값들이 커밋에 포함되지 않도록 설정합니다.

**`~/projects/sample-project/.gitignore`**
```
secrets.json
```

## 마치며

파이썬 모듈에서 사용할 key를 JSON파일의 key로 보관하면, `setattr()`을 이용해 복잡한 과정 없이 settings모듈의 속성을 지정할 수 있습니다. 이 방법을 사용하면 어떤 속성이 설정에 적용되었는지 일일이 확인 할 필요 없이, JSON파일만 보면 어떤 값들이 현재 설정에 들어가 있는지 알 수 있다는 장점이 있습니다.

만약 이미 기존 커밋에 비밀값들이 포함 된 상태라면, 커밋 이후에 비밀값들을 제외하도록 하여도 기존 커밋에는 내용이 남아있습니다. 이때는 git의 reset명령어로 비밀값이 포함된 커밋을 삭제해야 하며, 또 커밋이 공개 저장소에 push된 상태라면 서비스 프로바이더(페이스북, AWS등등...)에서 해당 비밀값들을 모두 재설정하셔야 합니다.