---
layout: post
title:  "AWS EC2 (Ubuntu)에 Nginx와 uWSGI를 사용하여 Django 배포"
categories: ['Django', 'AWS']
---

AWS의 EC2 인스턴스를 생성해 직접 웹 서버와 게이트웨이를 설정하는 가장 기본적인 배포방법을 설명한다.

pyenv로 사용하는 파이썬 버전은 `3.6.3`
가상환경 이름은 `fc-ec2-deploy`  
프로젝트 경로는 `~/projects/deploy/ec2_deploy_project`  
설정이 담긴 패키지명은 `config`
를 사용한다.

---

**Ubuntu Linux**  
서버의 OS

**Nginx**  
웹 서버. 클라이언트로부터의 HTTP요청을 받아 정적인 페이지/파일을 돌려준다.

**Django**  
웹 애플리케이션. 웹 요청에 대해 동적데이터를 돌려준다.

**uWSGI**  
웹 서버(Nginx)와 웹 애플리케이션(Django)간의 연결을 중계해준다.  
(Nginx에서 받은 요청을 Django에서 처리하기 위한 중계인 역할을 해준다)  

**WSGI**  
Web Server Gateway Interface  
파이썬에서 웹 서버와 웹 애플리케이션간의 동작을 중계해주는 인터페이스  
[Wikipedia](https://ko.wikipedia.org/wiki/%EC%9B%B9_%EC%84%9C%EB%B2%84_%EA%B2%8C%EC%9D%B4%ED%8A%B8%EC%9B%A8%EC%9D%B4_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4)

## IAM User생성

### Attach existing policies directly

AmazonEC2FullAccess 선택

### 생성 완료 후

`Access key ID`와 `Secret access key`의 값을 저장. 여기서 `Secret access key`는 절대로 외부에 유출되어서는 안 된다.

## 키 페어 생성

`서비스` -> `EC2` -> `네트워크 및 보안` -> `키 페어`

### 생성 완료 후 

다운로드된 `<name>.pem`파일을 `~/.ssh`폴더로 이동




## Instance생성

`서비스` -> `EC2` -> `인스턴스` -> `인스턴스 시작` -> `Ubuntu Server 16.04 LTS` -> `단계 6: 보안 그룹 구성`

보안 그룹 이름: `EC2-Deploy`  
설명: `EC2-Deploy Security Group`

-> `검토 및 시작` -> `시작`

이전에 생성한 키 페어 선택

### 인스턴스에 연결하기

[SSH를 사용하여 Linux 인스턴스에 연결](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

```
ssh -i <Private key> <username>@<EC2 public domain>

# 현재 프로젝트의 경우
ssh -i ~/.ssh/<Keypair name>.pem ubuntu@<EC2 Public domain>
```

#### UNPROTECTED PRIVATE KEY FILE 에러 발생 시

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/Users/Arcanelux/.ssh/fastcampus.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/Users/Arcanelux/.ssh/fastcampus.pem": bad permissions
Permission denied (publickey).
```

위와같은 경우, `chmod 400 <pem file>`로 소유주만 읽을 수 있도록 권한설정을 해준다.

### locale 설정

서버에서 편집 후 재접속한다

**`sudo vi /etc/default/locale`**

```
LC_CTYPE="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
LANG="en_US.UTF-8"
```


### 연결 후 기본 설정

```shell
# 패키지 정보를 업데이트
sudo apt-get update

# 설치되어 있는 패키지들을 의존성 검사하며 업그레이드
sudo apt-get dist-upgrade

# pip 설치
sudo apt-get install python-pip

# zsh 설치
sudo apt-get install zsh

# oh-my-zsh 설치
sudo curl -L http://install.ohmyz.sh | sh

# Default shell 변경
# shell 변경 후엔 exit로 연결을 종료한 뒤 재연결
sudo chsh ubuntu -s /usr/bin/zsh
```

### pyenv

#### pyenv requirements설치

[pyenv - Common-build-problems](https://github.com/yyuu/pyenv/wiki/Common-build-problems)

```
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils
```

#### pyenv 설치

```
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```

#### pyenv 설정을 .zshrc에 기록

**`vi ~/.zshrc`**
```
...
...
export PATH="/home/ubuntu/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

```
source ~/.zshrc
```


### Pillow 동작을 위한 Ubuntu python 라이브러리 설치

[Installation - Building on Linux](https://pillow.readthedocs.io/en/4.3.x/installation.html#building-on-linux)

```
sudo apt-get install python-dev python-setuptools
```

## Django 관련 설정

### Django 애플리케이션은 /srv Directory 사용

```
sudo chown -R ubuntu:ubuntu /srv/
```

[Linux file system structure]  
<http://www.thegeekstuff.com/2010/09/linux-file-system-structure/?utm_source=tuicool>

### 프로젝트 파일 전송

```
scp -i <Private key> -r <origin> <remote>

# 현재 프로젝트의 경우
scp -i <Keypair name>.pem -r ~/projects/django/ec2_deploy_project ubuntu@<EC2 Public domain>:/srv
```

### pyenv를 사용해서 virtualenv 생성

```
pyenv install 3.6.3
pyenv virtualenv 3.6.3 fc-ec2-deploy
```

### requirements설치

```
pip install -r requirements.txt
```

### runserver 테스트

```
cd mystie
./manage.py runserver 0:8080
```

### EC2 보안그룹에 포트 추가

`EC2` -> `네트워크 및 보안` -> `보안 그룹` -> `EC2-Deploy` -> `인바운드` -> `편집` -> `규칙 추가` -> `사용자 지정 TCP규칙` -> `8080`

## 배포를 위한 설정

### Django ALLOWED_HOSTS 설정

```
ALLOWED_HOSTS = [
  'localhost',
	'.amazonaws.com',
]
```

## uWSGI 관련 설정

### 웹 서버 관리용 유저 생성

```
sudo adduser deploy
```

### uWSGI설치

```
pyenv virtualenv 3.6.3 uwsgi-env
pyenv shell uwsgi-env
(uwsgi) pip install uwsgi
```

#### uWSGI 정상 동작 확인

```
uwsgi \
--http :(port) \
--home (virtualenv경로) \
--chdir <django프로젝트 경로> \
-w <설정 패키지명>.wsgi
```

ex) uWSGI가 설치된 virtualenv이름이 `uwsgi-env`, Django가 설치된 virtualenv이름이 `fc-ec2-deploy`, 장고 애플리케이션의 위치가 `/srv/ec2_deploy_project/mysite`, 설정 패키지 명이 `config`일 경우

```
/home/ubuntu/.pyenv/versions/uwsgi-env/bin/uwsgi \
--http :8080 \
--home /home/ubuntu/.pyenv/versions/fc-ec2-deploy \
--chdir /srv/ec2_deploy_project/mysite \
-w config.wsgi
```

실행 후 <ec2도메인>:8080으로 접속하여 요청을 잘 받는지 확인

#### uWSGI 사이트 파일 작성

**`(Local) ec2_deploy_project/.config/uwsgi/mysite.ini`**

```ini
[uwsgi]
chdir = /srv/ec2_deploy_project/mysite
module = config.wsgi:application
home = /home/ubuntu/.pyenv/versions/fc-ec2-deploy

uid = deploy
gid = deploy

socket = /tmp/mysite.sock
chmod-socket = 666
chown-socket = deploy:deploy

enable-threads = true
master = true
vacuum = true
pidfile = /tmp/mysite.pid
logto = /var/log/uwsgi/mysite/@(exec://date +%%Y-%%m-%%d).log
log-reopen = true
```

#### uWSGI site파일로 정상 동작 확인

```
# sudo로 root권한으로 실행
sudo /home/ubuntu/.pyenv/versions/uwsgi-env/bin/uwsgi -i /srv/ec2_deploy_project/.config/uwsgi/mysite.ini
```

#### uWSGI 서비스 설정파일 작성

**`(Local) ec2_deploy_project/.config/uwsgi/uwsgi.service`**

```ini
[Unit]
Description=uWSGI Emperor service
After=syslog.target

[Service]
ExecStart=/home/ubuntu/.pyenv/versions/uwsgi-env/bin/uwsgi -i /srv/ec2_deploy_project/.config/uwsgi/mysite.ini

Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

#### 서비스 파일 복사 및 시스템 재시작 시 자동으로 백그라운드에서 실행되도록 함

```
# scp명령어로 파일을 서버에 전송한 후 실행
sudo cp -f /srv/ec2_deploy_project/.config/uwsgi/uwsgi.service /etc/systemd/system/uwsgi.service
sudo systemctl daemon-reload
sudo systemctl enable uwsgi
```

## Nginx 관련 설정

#### Nginx 안정화 최신버전 사전세팅 및 설치

```
sudo apt-get install software-properties-common python-software-properties
sudo add-apt-repository ppa:nginx/stable
sudo apt-get update
sudo apt-get install nginx
nginx -v
```

#### Nginx 동작 User 변경

```
sudo vi /etc/nginx/nginx.conf

user deploy;
```

#### Nginx 가상서버 설정 파일 작성

**`(Local)ec2_deploy_project/.config/nginx/mysite.conf`**

```nginx
server {
    listen 80;
    server_name *.compute.amazonaws.com;
    charset utf-8;
    client_max_body_size 128M;

    location / {
        uwsgi_pass  unix:///tmp/mysite.sock;
        include     uwsgi_params;
    }
}
```

#### 설정파일 심볼릭 링크 생성

```
# scp명령어로 파일을 서버에 전송한 후 실행
sudo cp -f /srv/ec2_deploy_project/.config/nginx/mysite.conf /etc/nginx/sites-available/mysite.conf
sudo ln -sf /etc/nginx/sites-available/mysite.conf /etc/nginx/sites-enabled/mysite.conf
```

#### sites-enabled의 default파일 삭제

```
sudo rm /etc/nginx/sites-enabled/default
```

## uWSGI, Nginx재시작

```
sudo systemctl restart uwsgi nginx
```

### EC2 보안그룹에 포트 추가

`EC2` -> `네트워크 및 보안` -> `보안 그룹` -> `EC2-Deploy` -> `인바운드` -> `편집` -> `규칙 추가` -> `HTTP` -> `(자동으로 80번 포트 선택됨)`

---

### 오류 발생 시

#### systemctl restart시 오류 발생 시

```
(오류 발생한 서비스에 따라 아래 명령어 실행)
sudo systemctl status uwsgi.service
sudo systemctl status nginx.service
```

#### 502 Bad Gateway

**Nginx log파일 확인**  
```
cat /var/log/nginx/error.log
```

#### Nginx log파일에서 sock파일 접근 불가시

**socket파일 권한 소유자 확인**

```
cd /tmp
ls -al
srw-rw-rw-  1 nginx  nginx     0 Nov  8 06:58 mysite.sock

nginx가 소유자가 아닐 경우, 
sudo rm mysite.*로 삭제 후 서비스 재시작
```

## 정적 파일 설정

**`(Local) ec2_deploy_project/.config/nginx/mysite.conf`**

```
server {
    listen 80;
    server_name *.compute.amazonaws.com *.pikachu.kr;
    charset utf-8;
    client_max_body_size 128M;

    location / {
        uwsgi_pass  unix:///tmp/mysite.sock;
        include     uwsgi_params;
    }

    location /static/ {
        alias /srv/ec2_deploy_project/.static_root/;
    }

    location /media/ {
        alias /srv/ec2_deploy_project/mysite/media/;
    }
}
```

**이미지 업로드시 Permission Denied발생 할 경우**  
`sudo chmod 755 media`로 `media`폴더의 권한을 변경, `media`폴더 하위 폴더 모두 삭제 후 다시 실행


## RDS

### 생성 및 보안 그룹 설정

RDS가 속하는 보안그룹의 인바운드에서 `5432`번 포트에 자신의 IP와 연결을 허용하고자 하는 `EC2`가 속하는 보안 그룹을 추가해준다

### psql로 연결하고 싶을 때

```
psql --host=<RDS Endpoint> --user=<username> --port=5432 <db name>
```

## S3 연결

### boto3와 django-storages의 역할

**`django-storages`를 사용하지 않을 경우**

```python
# S3의 API를 사용해서 실제 파일 스토리지의 역할을 할 수 있는 클래스를 직접 정의해야 함
from django.core.files.storage import Storage


class S3Storage(Storage):
    def delete(file_name):
        url = f'https://s3.amazonaws.com/{file_name}'
        response = requests.delete(url, credentials=...)
        return response == 204:
    
    def exists(file_name):
        ....
        ....
```

**`boto3`의 역할**

```python
# django-storages라이브러리에서 S3 API와 통신할 때 사용
# 직접 구현한다면 requests라이브러리를 사용해야 함 
#  (작업할 부분이 많아지며, AWS의 API가 업데이트 될 경우 항상 수정해야 함)

def save(file):
    boto3.upload(bucket_name, file)
    
    
def save(file):
    response = requests.post(url, file)
    if response.status == 200:
      return File(respose.content)
    raise ValueError(..)
```

### S3 관련 권한을 IAM유저에 추가

`서비스` -> `IAM` -> `Users` -> `EC2-User` -> `Permissions` -> `Add permissions` -> `Attach existing policies directly` -> `AmazonS3FullAccess` -> `Next` -> `Add permissions`

### boto3를 이용해서 S3 Bucket생성

**`Terminal`**

```
pip install boto3
python
>>> import boto3
>>> session = boto3.Session()
>>> client = session.client('s3')
>>> client.create_bucket(Bucket='fc-6th-test', CreateBucketConfiguration={'LocationConstraint': 'ap-northeast-2'})
{'Location': 'http://fc-6th-test.s3.amazonaws.com/', {'ResponseMetadata': {'RequestId': '23A44CAD8A1360ED', 'HostId': 'VRGWEULfb3AkF+t71d00DT6Hug8ssDJvO1/R6QDdESXEQMMbgsrQARaswA1ybwZr8VPOr4wfX6Y=', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amz-id-2': 'VRGWEULfb3AkF+t71d00DT6Hug8ssDJvO1/R6QDdESXEQMMbgsrQARaswA1ybwZr8VPOr4wfX6Y=', 'x-amz-request-id': '23A44CAD8A1360ED', 'date': 'Sun, 29 Oct 2017 17:04:39 GMT', 'location': 'http://fc-6th-test.s3.amazonaws.com/', 'content-length': '0', 'server': 'AmazonS3'}, 'RetryAttempts': 0}, 'Location': 'http://fc-6th-test.s3.amazonaws.com/'}
>>> 
```

### AWS관련 설정 추가

**`ec2_deploy_project/.config_secret/settings_common.json`**

```json
{
  "aws": {
    "access_key_id": "<AWS Access key id>",
    "secret_access_key": "<AWS Secret access key>",
    "s3_bucket_name": "<S3 Bucket name>",
    "s3_region_name": "ap-northeast-2"
  }
}
```

### [django-storages](https://django-storages.readthedocs.io/en/latest/) 패키지 설치

**`Terminal`**

```
pip install django-storages
```

### Custom StorageClass정의

**`config/storages.py`**

```python
from django.conf import settings
from storages.backends.s3boto3 import S3Boto3Storage


class StaticStorage(S3Boto3Storage):
    location = settings.STATICFILES_LOCATION


class MediaStorage(S3Boto3Storage):
    location = settings.MEDIAFILES_LOCATION
```

### settings.py에 설정 추가

**`config/settings.py`**

```python
# AWS
AWS_ACCESS_KEY_ID = config_secret['aws']['access_key_id']
AWS_SECRET_ACCESS_KEY = config_secret['aws']['secret_access_key']
AWS_STORAGE_BUCKET_NAME = config_secret['aws']['s3_bucket_name']
AWS_S3_SIGNATURE_VERSION = 's3v4'
AWS_S3_REGION_NAME = 'ap-northeast-2'

# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.11/howto/static-files/

STATIC_DIR = os.path.join(BASE_DIR, 'static')
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    STATIC_DIR,
]
STATIC_ROOT = os.path.join(ROOT_DIR, '.static_root')
STATICFILES_LOCATION = 'static'
STATICFILES_STORAGE = 'config.storages.StaticStorage'

MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'
MEDIAFILES_LOCATION = 'media'
DEFAULT_FILE_STORAGE = 'config.storages.MediaStorage'
```

### static태그를 사용해 정적 파일 불러오기

```django
{% raw %}<img src="{% static 'pby59.jpg' %}" alt="" width="50%">{% endraw %}
```

### member application추가 및 AUTH_USER_MODEL재정의, img_profile필드 추가

**`Terminal`**

```
./manage.py startapp member
```

**`member/models.py`**

```python
from django.contrib.auth.models import AbstractUser
from django.db import models


class User(AbstractUser):
    img_profile = models.ImageField(upload_to='user', blank=True)
```

**`config/settings.py`**

```python
AUTH_USER_MODEL = 'member.User'

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'storages',

    'member',
]
```

### 기존 데이터베이스 삭제 후 새로 마이그레이션 생성 및 적용

`AUTH_USER_MODEL`을 재정의 했으므로 데이터베이스를 전체 삭제 후 다시 마이그레이션을 생성 및 적용시켜준다

**`Terminal`**

```
psql psql --host=<RDS endpoint> --user=<RDS username> --port=5432 postgres

postgres=> DROP DATABASE deploy;
DROP DATABASE
postgres=> CREATE DATABASE deploy OWNER <username>;
CREATE DATABASE
postgres=> \q

./manage.py makemigrations
./manage.py migrate
```