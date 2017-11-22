---
layout: post
title:  "클라우드 서비스 모델의 종류 (IaaS, PaaS, SaaS)"
categories: ['Network']
---

### 클라우드 서비스 모델

![Cloud]({{ site.url }}/images/django-eb/cloud.png)

#### On-premise (Traditional IT)

사용자가 직접 인프라, 플랫폼, 애플리케이션을 관리하는 모델.  
규모있는 업체라면 직접 IDC를 구축하고, 일반적인 경우 IDC에 공간을 할당받아 물리서버를 설치하고 하드웨어, 운영체제, 서버 애플리케이션을 모두 관리한다.

#### IaaS (Infrastructure as a Service)

> AWS EC2

하드웨어를 서비스로 제공하는 클라우드 모델. OS와 애플리케이션을 관리한다.

#### PaaS (Platform as a Service)

> AWS Elastic Beanstalk

하드웨어에 더해 애플리케이션을 운영하기 위한 OS와 관련 기능들을 서비스로 제공한다. 개발자는 애플리케이션과 서비스로 제공되는 기능을 연결하는 로직을 작성해야 한다.

#### Saas (Software as a Service)

> Google Apps, Office365

애플리케이션 레벨까지 서비스로 제공된다. 개발자보다는 실 사용자에게 바로 제공.
