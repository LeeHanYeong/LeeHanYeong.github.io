---
layout: post
title:  "AWS EC2에 직접 Django프로젝트 배포하기"
categories: ['Django', 'AWS', 'Python']
---



---

## 시작 전 필요사항

- AWS 회원가입


## AWS 설정

### 인스턴스 및 키페어 생성

![Region_Seoul]({{ site.url }}/images/abc/region_seoul.png){:width="291px"}

우측 위의 지역을 `Seoul`로 변경 후, AWS services에서 `EC2`검색

`EC2 Dashboard`에서 `Instances`항목을 클릭, `Launch Instance`클릭  
AMI선택에서 `Ubuntu Server 16.04 LTS`를 선택  
나머지 설정은 전부 기본으로, `Review and Launch`클릭

처음이라면 `Launch`클릭시 키페어를 생성해야한다. `Create a new key pair`항목이 선택된 상태에서 키페어 이름을 넣어주고 `Download Key Pair`를 클릭. 여기서의 키페어는 다시 확인 불가능하므로 `~/.ssh`폴더 내부에 저장해둔다. 

생성된 인스턴스는 기본적으로 `22`번 포트로 `SSH`요청만을 허용한다.

### 인스턴스에 Security Group설정

생성된 인스턴스는 자동으로 `launch-wizard-<n>`이라는 이름을 가진다. 별개의 `Security Group`을 만들어 적용시켜준다.

#### 새 Security Group생성

`EC2 Dashboard`에서 `NETWORK & SECURITY`탭의 `Security Groups`항목으로 이동  
`Create Security Group`을 실행, Inbound에 `SSH`항목을 추가시켜준다. Source항목은 기왕이면 `MyIP`를 선택한다.

다시 `Instances`항목으로 돌아와 생성한 인스턴스를 클릭, 위의 `Actions`를 클릭하고 `Networking` -> `Change Security Group`을 선택, 방금 생성한 `SecurityGroup`을 사용하도록 변경해준다.


## SSH 접속

[SSH를 사용하여 Linux 인스턴스에 연결](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)


## Ubuntu 기본 설정

### python-pip설치
```
sudo apt-get install python-pip
```

### zsh 설치
```
sudo apt-get install zsh
```

### oh-my-zsh 설치
```
sudo curl -L http://install.ohmyz.sh | sh
```

### Default shell 변경
```
sudo chsh ubuntu -s /usr/bin/zsh
```

### pyenv requirements설치
[공식문서](https://github.com/yyuu/pyenv/wiki/Common-build-problems)

```
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils
```

### pyenv 설치
```
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```

### pyenv 설정 .zshrc에 기록
```
vi ~/.zshrc
export PATH="/home/ubuntu/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```


### Pillow 라이브러리 설치
[공식문서](https://pillow.readthedocs.io/en/3.4.x/installation.html#basic-installation)

```
sudo apt-get install libtiff5-dev libjpeg8-dev zlib1g-dev \
    libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python-tk
```

## Django 관련 설정

#### 장고 애플리케이션은 /srv Directory 사용
```
sudo chown -R ubuntu:ubuntu /srv/
```

## uWSGI 설정

### uWSGI 설정 파일 작성

`django_app/config_secret/uwsgi/debug.ini`

```ini
[uwsgi]
home = <가상환경 경로>
chdir = <django_app>폴더의 절대경로
module = config.wsgi_modules.debug
http = :8000
```

`django_app/config_secret/uwsgi/deploy.ini`

```ini
[uwsgi]
home = /home/ubuntu/.pyenv/versions/deploy_ec2
chdir = /srv/deploy_ec2/django_app
module = config.wsgi_modules.deploy

uid = deploy
gid = deploy

socket = /tmp/ec2.sock
chmod-socket = 666
chown-socket = deploy:deploy

enable-threads = true
master = true

vacuum = true
logger = file:/tmp/uwsgi.log
```



### uWSGI Service 설정 파일

`sudo vi /etc/systemd/system/uwsgi.service`

```ini
[Unit]
Description=uWSGI service
After=syslog.target

[Service]
ExecStart=/home/ubuntu/.pyenv/versions/deploy_ec2/bin/uwsgi --ini /srv/deploy_ec2/.config_secret/uwsgi/deploy.ini

Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

## Nginx 설정

### Nginx설정

`sudo vi /etc/nginx/nginx.conf`

`user`항목을 `deploy`로 변경  
`http`항목 내부의 `server_names_hash_bucket_size`항목의 주석을 해제하고, 값을 250으로 변경. EC2의 public domain길이가 기본값보다 크기때문에 설정해준다.

### server설정

`sudo vi /etc/nginx/sites-available/ec2`

```
server {
        listen 80;
        server_name ec2-52-79-92-108.ap-northeast-2.compute.amazonaws.com;
        charset utf-8;
        client_max_body_size 128M;

        location / {
                uwsgi_pass      unix:///tmp/ec2.sock;
                include         uwsgi_params;
        }
}
```
