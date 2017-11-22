---
layout: post
title:  "ElasticBeanstalk을 사용하는 Django프로젝트에 TravisCI세팅"
categories: ['Django', 'AWS', 'CI']
---

## travis-ci.org 회원가입 및 관리할 저장소 지정

**[https://travis-ci.org](https://travis-ci.org)**

![Repositories]({{ site.url }}/images/travis/repository.png)


## Travis Client 설치

**[https://github.com/travis-ci/travis.rb](https://github.com/travis-ci/travis.rb)**


## 터미널에서 TravisCI Login

> GitHub계정명과 비밀번호를 입력한다

```
✗ travis login
We need your GitHub login to identify you.
This information will not be sent to Travis CI, only to api.github.com.
The password will not be displayed.

Try running with --github-token or --auto if you don't want to enter your password anyway.

Username: leehanyeong
Password for leehanyeong: 
Successfully logged in as LeeHanYeong!
```

## 프로젝트 폴더에 `.travis.yml`파일 생성

```yaml
language: python

services:
  - postgresql

python:
  - 3.6

install:
  - pip install -r requirements.txt

before_script:
  - psql -c "CREATE DATABASE travisci;" -U postgres
  - cd greenwrap

script:
  - python manage.py migrate --noinput
  - python manage.py test
```

## 암호화 된 secrets생성

> Git으로 관리되지 않는 비밀파일들은 Travis Client에서 제공하는 encrypt기능으로 암호화해 압축한 후, Travis서버에서 압축을 해제해서 사용한다
> 
> 여기서는 프로젝트의 `.config_secret/` 폴더를 압축해 `secrets.tar`파일을 만들며, 이를 암호화해 `secrets.tar.enc`파일로 만든다

**`secrets.tar 생성`**
```
✗ tar cvf secrets.tar .config_secret   
a .config_secret
a .config_secret/dockercfg.json
a .config_secret/settings_common.json
a .config_secret/settings_debug.json
a .config_secret/settings_deploy.json
a .config_secret/settings_local.json
a .config_secret/settings_travis.json
```

**`secrets.tar.enc 생성`**
```
✗ travis encrypt-file secrets.tar -a -f
encrypting secrets.tar for LeeHanYeong/GreenWrap
storing result as secrets.tar.enc
storing secure env variables for decryption

Make sure to add secrets.tar.enc to the git repository.
Make sure not to add secrets.tar to the git repository.
Commit all changes to your .travis.yml.
```

`-a`는 `.travis.yml`의 `before_script`부분에 해당 암호화 파일을 CI과정에서 사용할 수 있도록 암호화를 해제해주는 스크립트를 `before_install`항목에 추가해주며, `-f`는 이미 암호화 하려는 파일의 결과물이 이미 존재할 경우 덮어쓴다.

## .gitignore설정 및 .travis.yml, secrets.tar.enc를 git에 추가

**`Terminal`**
```
echo secrets.tar >> .gitignore
git add .travis.yml
git add secrets.tar.enc
git commit -m 'travis secrets'
```


## Travis AWS Elastic Beanstalk Deployment 설정

[https://docs.travis-ci.com/user/deployment/elasticbeanstalk](https://docs.travis-ci.com/user/deployment/elasticbeanstalk)

