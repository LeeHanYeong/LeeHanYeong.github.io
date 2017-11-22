---
layout: post
title:  "Elastic Beanstalk을 사용한 Django프로젝트 배포"
categories: ['Django', 'AWS']
---

Elastic Beanstalk은 AWS에서 제공하는 PaaS서비스이다. 대부분의 서버 플랫폼 세팅은 AWS가 제공하며, 개발자는 애플리케이션 코드만 업로드하면 손쉽게 배포를 할 수 있다.

클라우드 서비스 모델(IaaS, PaaS, SaaS)에 대해서는 [이 포스팅](/cloud-model)을 참조한다.

물론 배포와 관련된 세부사항을 설정하는것은 쉽지않다. 하지만 로드밸런싱이나 서버 스케일링, 모니터링 등을 직접 구축하는 것 보다는 AWS에서 제공해주는 서비스를 익히는 것이 더 간편하다.

이번 포스팅에서는 Django프로젝트에 Elastic Beanstalk서비스를 사용해 서버에 배포하는 설정을 추가하고, 터미널에서 배포하는 방법에 대해 기술한다. 따라할 때는 Git과 pyenv, vim의 사용법에 대한 간단한 이해를 필요로 한다.

- [pyenv와 virtualenv를 사용한 파이썬 개발환경 구성](/configuring-the-python-development-environment-with-pyenv-and-virtualenv)

작성에는 AWS Elastic Beanstalk의 공식문서인 [Django 애플리케이션을 Elastic Beanstalk에 배포](http://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/create-deploy-python-django.html)의 일부분을 참조하였다.

## Django프로젝트 세팅 및 프로젝트 구조

프로젝트는 `~/projects/eb-django-project/`폴더에 위치하며,  
Django코드는 `~/projects/eb-django-project/eb-django/`내에 있도록 구성한다.

pyenv를 사용하며, 이 때 파이썬 버전은 3.6.3을 사용한다.

### Django 프로젝트 세팅

**`Terminal`**
```shell
# 홈 폴더에 projects폴더가 없다면 생성 후 진행한다
mkdir ~/projects

# ~/projects/ 폴더로 이동 후 eb-django-project폴더를 생성한다
cd ~/projects
mkdir eb-django-project

# ~/projects/eb-django-project/ 폴더로 이동 후 pyenv-virtualenv를 사용해서 가상환경을 생성 및 적용시킨다
cd eb-django-project
pyenv virtualenv 3.6.3 eb-django-env
pyenv local eb-django-env

# 가상환경에 Django패키지를 설치한다
pip install django

# 설정 관련 폴더명을 config로 할 것이기 때문에, startproject명은 config로 지정한다
django-admin startproject config

# 이후 Django코드의 컨테이너 폴더명을 eb-django로 바꾼다
mv config eb-django
```

### .gitignore및 requirements작성, 첫 커밋

[.gitignore](https://raw.githubusercontent.com/BlogSampleProjects/Settings/master/.gitignore)

**`Terminal`**
```shell
# 위의 .gitignore파일의 내용을 복사해 작성한다
vi .gitignore

# git저장소를 초기화한다
git init

# requirements.txt 작성
pip freeze > requirements.txt

# 첫 번째 커밋 작성
git add -A
git commit -m 'First commit'
```

### 프로젝트 구조

위 과정을 완료한 후 프로젝트 폴더의 위치와 구조는 홈 폴더 기준으로 다음과 같다

```
~/projects/
  eb-django-project/
    .git/
    .gitignore
    requirements.txt
    eb-django/
      config/
        __init__.py
        settings.py
        urls.py
        wsgi.py
      manage.py
```

### 기본 뷰와 ALLOWED_HOSTS설정

배포가 완료되었을 때 확인할 수 있는 기본 뷰와, Elastic Beanstalk에서 제공하는 도메인을 허용하는 설정을 추가한다.

**`~/projects/eb-django-project/config/views.py`**
```python
from django.http import HttpResponse
from django.views import View


class IndexView(View):
    def get(self, request):
        return HttpResponse('<h1>EB Django Project</h1>')
```

**`~/projects/eb-django-project/config/urls.py`**
```python
from django.conf.urls import url
from django.contrib import admin

from . import views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^$', views.IndexView.as_view(), name='index'),
]
```

**`~/projects/eb-django-project/config/settings.py`**
```python
...
ALLOWED_HOSTS = [
    'localhost',
    '127.0.0.1',
    '.elasticbeanstalk.com',
]
...
```

세 파일을 수정한 변경사항을 커밋해준다

**`Terminal`**
```shell
git add -A
git commit -m 'IndexView추가, ALLOWED_HOSTS설정'
```

---

## AWS 설정

AWS 회원가입 과정은 생략한다.

### IAM User생성

> Elastic Beanstalk을 사용할 수 있는 권한을 가진 사용자 계정을 생성한다.

1. [AWS 웹 콘솔](https://console.aws.amazon.com/console/home)에 접근해서 우측 위의 선택된 리전이 서울인지 확인 후, 좌측 위의 **서비스** 버튼 클릭 -> **IAM**서비스로 이동한다.

2. [서울리전 IAM서비스](https://console.aws.amazon.com/iam/home?region=ap-northeast-2#/home)로 이동한 후, 좌측의 **Users**탭을 클릭한다.

3. 상단의 **Add user**버튼을 클릭한다.

4-1. Details
  - User name은 **EB-User**를 사용.
  - Access type은 **Programmatic access*만 체크한다.

4-2. Permissions
  - Attach existing policies directly 탭을 클릭한다.
  - 아래의 검색창에 'ElasticBeanstalkFullAccess'를 검색하면 남는 하나의 정책을 체크한다.

4-3. Review
  - User name: EB-User
  - AWS access type: Programmatic access - with an access key
  - Permissions summary: AWSElasticBeanstalkFullAccess

4-4. Complete
  - Download .csv버튼을 눌러 계정 Access key와 Secret access key가 기록된 파일을 다운받는다.

### AWS CLI 설치 및 Credentials설정

[AWS 명령줄 인터페이스](https://aws.amazon.com/ko/cli/)(AWS CLI)는 AWS 서비스를 명령줄(Command line)에서 제어할 수 있는 도구이다.

위에서 유저를 생성하고 다운로드받은 **credentials.csv**파일을 열어 AWS Access ID와 Secret access key를 확인하고 아래 과정을 진행한다.

**`Terminal`**
```shell
# 프로젝트 폴더로 이동해 가상환경에 awscli패키지를 설치한다
cd ~/projects/eb-django/project/
pip install awscli

# 로그인 정보를 저장하고 기본 지역과 출력 포맷을 설정한다
aws configure
> AWS Access Key ID [None]: AKIAJ***************
> AWS Secret Access Key [None]: 9CNXeyB***********************
> Default region name [None]:  ap-northeast-2
> Default output format [None]: json
```

### AWS Elastic Beanstalk CLI설치

[Elastic Beanstalk 명령줄 인터페이스](http://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/eb-cli3.html)(EB CLI)는 AWS의 Elastic Beanstalk서비스를 명령줄에서 제어하는 도구이다.

**`Terminal`**
```shell
# 프로젝트 폴더로 이동해 가상환경에 awsebcli패키지를 설치한다
cd ~/projects/eb-django/project/
pip install awsebcli
```

## Elastic Beanstalk을 사용하기 위한 배포 설정

Elastic Beanstalk을 사용한 배포에서, 하나의 프로젝트는 EB에서 **Application**으로 취급된다. 실제로 작동하는 서버는 EB에서 **Application**에 종속된 **Environment**로 동작하며, 하나의 **Application**은 여러개의 **Envrionment**를 가질 수 있다.

또한 하나의 **Environment**는 물리적 컴퓨터로 취급되는 가상 컴퓨팅 파워인 **EC2**를 여러개 가질 수 있다.

하나의 **Application**이 여러개의 **Environment**를 가질 수 있는 이유는, 실제 개발환경에서는 하나의 애플리케이션이 배포/개발/테스트 등 여러 환경에서 동작하도록 할 필요가 있기 때문이다.

반면에 여러개의 **EC2**가 하나의 **Environment**에 존재할 수 있는 이유는 서비스로의 많은 요청에 응답하기 위해 동작하는 컴퓨팅 엔진(EC2)의 수를 조절할 수 도록 하기 위해서이다.

이 내용은 지금 당장은 이해하기 어려울 수 있으나, 서버 배포를 공부하다보면 자연스럽게 알게되는 내용이니 바로 실습으로 넘어가도 무관하다.

## .ebextensions/django.config작성

.ebextensions는 Elastic Beanstalk의 동작에 대한 설정들을 담는 폴더이며, 여기서는 Elastic Beanstalk에서 Django가 동작하기 위한 설정을 추가해준다.

**`Terminal`**
```shell
cd ~/projects/eb-django-project/
mkdir .ebextensions
cd .ebextensions
vi django.config
```

**`~/projects/eb-django-project/.ebextensions/django.config`**
```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    DJANGO_SETTINGS_MODULE: config.settings
    PYTHONPATH: /opt/python/current/app/eb-django:$PYTHONPATH
  aws:elasticbeanstalk:container:python:
    WSGIPath: eb-django/config/wsgi.py
```
  
모든 과정을 완료했으면 변경사항을 저장하고 커밋한다
  
**`Terminal`**
```
git add -A
git commit -m 'ElasticBeanstalk및 ebextensions설정 추가'
```

### Application 생성

**`Terminal`**
```shell
# Application 생성
eb init

# 서울을 선택한다.
Select a default region
1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-south-1 : Asia Pacific (Mumbai)
7) ap-southeast-1 : Asia Pacific (Singapore)
8) ap-southeast-2 : Asia Pacific (Sydney)
9) ap-northeast-1 : Asia Pacific (Tokyo)
10) ap-northeast-2 : Asia Pacific (Seoul)
11) sa-east-1 : South America (Sao Paulo)
12) cn-north-1 : China (Beijing)
13) us-east-2 : US East (Ohio)
14) ca-central-1 : Canada (Central)
15) eu-west-2 : EU (London)
(default is 3): 10

# Application이름을 입력해준다. (EB Django Project)
Enter Application Name
(default is "eb-django-project"): EB Django Project
Application elastic-beanstalk has been created.

# 파이썬이 맞으므로 엔터를 입력한다.
It appears you are using Python. Is this correct?
(Y/n):

# 최신버전의 파이썬을 선택한다.
Select a platform version.
1) Python 3.6
2) Python 3.4
3) Python
4) Python 2.7
5) Python 3.4 (Preconfigured - Docker)
(default is 1):

# CodeCommit은 사용하지 않는다. 엔터를 입력한다.
Note: Elastic Beanstalk now supports AWS CodeCommit; a fully-managed source control service. To learn more, see Docs: https://aws.amazon.com/codecommit/
Do you wish to continue with CodeCommit? (y/N) (default is n):

# SSH 접속을 허용할 지 물어본다. 엔터를 치거나 y를 입력한다.
Do you want to set up SSH for your instances?
(Y/n):

# EC2에 접속할 수 있는 키페어를 생성한다. 기본값을 사용하기 위해 엔터를 입력.
# 이후 과정도 전부 엔터를 입력한다.
Type a keypair name.
(Default is aws-eb):
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/lhy/.ssh/aws-eb.
Your public key has been saved in /Users/lhy/.ssh/aws-eb.pub.
The key fingerprint is:
SHA256:dJ3WqkCA7Xf1C0T1hxjtJR29jqZEChqwM6UX4tJuAvY aws-eb
The key's randomart image is:
+---[RSA 2048]----+
|     o.    .oo.oo|
|   o.o..   .o=oo+|
|  o *.. o .o=ooo+|
|.o B o.+...o..o..|
|o + + o.S.o ..o. |
| . E .   o o o.. |
|  o       o o    |
|           .     |
|                 |
+----[SHA256]-----+
WARNING: Uploaded SSH public key for "aws-eb" into EC2 for region ap-northeast-2.
```

## Environment 생성

애플리케이션에서 실제로 동작할 환경을 생성해준다. 이 과정에서는 로드밸런서와 오토스케일링 설정, 모니터링을 위한 클라우드워치 설정 및 각각에 대한 보안그룹이 생성된다.

설정이 완료되면 환경에 URL이 부여되며, 소유한 도메인이 있다면 추후 DNS 설정에서 이 URL을 CNAME으로 원하는 도메인으로 연결할 수 있다.

**`Terminal`**
```shell
eb create
Enter Environment Name
(default is EBDjangoProject-dev):

# DNS CNAME이 겹칠 경우, 다른 값을 입력해준다. 이 값은 리전에서 유일해야 한다.
Enter DNS CNAME prefix
(default is EBDjangoProject-dev):

# application을 선택해준다.
Select a load balancer type
1) classic
2) application
3) network
(default is 1): 2

Creating application version archive "app-171123_040427".
Uploading eb-django-project/app-171123_040427.zip to S3. This may take a while.
Upload Complete.
Environment details for: eb-django-project-dev
  Application name: eb-django-project
  Region: ap-northeast-2
  Deployed Version: app-171123_040427
  Environment ID: e-jhqg32ppmh
  Platform: arn:aws:elasticbeanstalk:ap-northeast-2::platform/Python 3.6 running on 64bit Amazon Linux/2.6.0
  Tier: WebServer-Standard
  CNAME: eb-django-project-dev.ap-northeast-2.elasticbeanstalk.com
  Updated: 2017-11-22 19:04:29.435000+00:00
Printing Status:
INFO: createEnvironment is starting.
INFO: Using elasticbeanstalk-ap-northeast-2-344415754306 as Amazon S3 storage bucket for environment data.
 -- Events -- (safe to Ctrl+C)
```

배포가 완료된 후 `eb open`명령어를 입력하면 환경에 해당하는 URL로 브라우저가 열린다.

![Complete]({{ site.url }}/images/django-eb/eb-complete.png)

## 소스코드

이 프로젝트의 소스코드는 [https://github.com/LeeHanYeong/EB-Django-Deploy-Sample](https://github.com/LeeHanYeong/EB-Django-Deploy-Sample) 저장소에서 볼 수 있으며, 포스팅 진행 여부에 따라 최신 코드는 업데이트 될 수 있다.

현재 상황의 코드를 Git을 사용해 가져와서 보고 싶다면, 아래 명령어를 입력한다.  
여기서는 `~/projects/sample/eb-django-project/` 폴더에 코드를 가져온다.

**`Terminal`**
```shell
# ~/projects/sample/ 폴더를 생성한다.
cd ~/projects/
mkdir sample

# sample 폴더에서 eb-django-project폴더로 Git저장소의 내용을 가져온다.
cd sample
git clone https://github.com/LeeHanYeong/EB-Django-Deploy-Sample.git eb-django-project
cd eb-django-project

# Git의 checkout기능을 사용해 v1.0태그의 커밋 위치에 새 브랜치를 만들고 체크아웃한다.
git checkout -b v1 v1.0

# pyenv를 사용해 가상 환경을 생성하고 적용한 후, requirements.txt의 패키지들을 설치한다.
pyenv virtualenv 3.6.3 eb-django-env
pyenv local eb-django-env
pip install -r requirements.txt

# runserver명령어를 사용해 동작을 확인한다. (브라우저에 localhost:8000 입력)
cd eb-django
./manage.py runserver
```

## 마치며...

배포는 서버에 있던 모든 내용들을 덮어쓴다. Django의 기본 설정은 로컬 파일 데이터베이스인 Sqlite를 사용하며, 사용자들이 업로드한 파일은 로컬 파일시스템에 저장된다. 사용자의 파일과 데이터베이스 내용을 보존하고 싶다면, Elastic Beanstalk과는 별도로 구성된 스토리지와 데이터베이스 서버를 사용해야 한다.

다음 포스팅에서는 AWS의 스토리지 서비스인 S3와 데이터베이스 서비스인 RDS를 사용해 항상 파일과 데이터베이스를 유지할 수 있도록 배포하는 방법에 대해 알아보도록 하겠다.