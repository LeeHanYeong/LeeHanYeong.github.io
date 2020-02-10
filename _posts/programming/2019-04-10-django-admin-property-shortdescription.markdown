---
layout: post
title:  'poetry사용'
categories: ['Django', 'Python']
---

**Poetry 베타 설치**

```sh
cd /tmp
curl -o get-poetry.py https://raw.githubusercontent.com/sdispater/poetry/master/get-poetry.py
python get-poetry.py
```



**셸 설정 파일 (`.bash_profile`, `.zshrc`)에 Pyenv보다 아래쪽에 Poetry설정 적용**

```sh
# pyenv, pyenv-virtualenv
export PYENV_ROOT=/usr/local/var/pyenv
if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi
if which pyenv-virtualenv-init > /dev/null; then eval "$(pyenv virtualenv-init -)"; fi

# poetry
export PATH=$HOME/.poetry/bin:$PATH
```



**셸 껐다 켠 후 poetry의 위치 확인**

```sh
(global) ➜  / which poetry
/Users/<username>/.poetry/bin/poetry
```



Poetry를 사용할 폴더에서 init

```
> poetry init

This command will guide you through creating your pyproject.toml config.

Package name [app]:  Project name
Version [0.1.0]:  1.0.0
Description []:
Author [Lee HanYeong <dev@lhy.kr>, n to skip]:
License []:
Compatible Python versions [^3.7]:

Would you like to define your main dependencies interactively? (yes/no) [yes] no
Would you like to define your dev dependencies (require-dev) interactively (yes/no) [yes] no
Generated file
```



Poetry로 패키지 추가

```

```

