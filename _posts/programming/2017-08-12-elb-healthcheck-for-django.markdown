---
layout: post
title:  "Elastic Beanstalk의 Django 애플리케이션에서 발생하는 ELB Health check 4xx에러 해결"
categories: ['Django', 'AWS', 'Python']
---

최근 `Django`프로젝트를 배포할 때 항상 `Docker`와 `AWS Elastic Beanstalk`을 사용하고 있다.

이번 포스팅에서는 `Elastic Beanstalk`에 `Django`애플리케이션을 배포할 때, `Elastic Load Balancing`서비스에서 `Health check`가 실패하는 문제를 해결했던 사례를 기술한다.

## EB, ELB, EC2, AutoScaling

`EB(Elastic Beanstalk)`는 서버 인프라의 설정 없이 자동으로 `AWS`의 클라우드에서 애플리케이션을 배포해주는 서비스이다. (라고 기술되어 있지만, 당연히 전부 자동은 아니며 여러가지 `EB`만의 설정이 필요하다)

`EB`로 작동하는 서비스로 들어오는 트래픽은 `ELB(Elastic Load Balancing)`를 통해 연결된 여러 `EC2`인스턴스로 분산된다. 이 때 `EC2`인스턴스들의 사용량에 따라 자동으로 인스턴스의 수를 조절하는 `AutoScaling`기능도 제공된다.
<br><br>

[Best Practices in Evaluating Elastic Load Balancing](https://aws.amazon.com/articles/1636185810492479){:style="text-align: center; display: block;"}
![Best]({{ site.url }}/images/elb-health-check-for-django/best.png){:style=" border: 1px solid #ddd; border-radius: 1%;"}

<center>(위의 Best하다는 구조를 자동으로 구성해준다...)</center>


## 100.0 % of the requests are erroring with HTTP 4xx

![Severe]({{ site.url }}/images/elb-health-check-for-django/severe.png)

`Django`에서는 `HTTP Host header attack`을 방지하기 위해 설정에 `ALLOWED_HOSTS`화이트 리스트를 이용할 것을 권장하고 있다. 일반적으로는 서비스에서 사용하는 도메인 명을 사용한다.

> [Django - Host header validation](https://docs.djangoproject.com/en/1.11/topics/security/#host-header-validation)  
> [Django - ALLOWED_HOSTS](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-ALLOWED_HOSTS)

```python
# settings.py
ALLOWED_HOSTS = [
    'lhy.kr',
]
```

이렇게 구성할 경우, `ELB`에서 `EC2`로의 `Health check`기능에서 문제가 생긴다. `Health check`는 실행중인 애플리케이션의 특정 URL로 GET요청을 보내, 해당 애플리케이션이 정상적으로 동작중인지 확인하는 기능이다.

일반적인 트래픽은 `HTTP Header`의 `HOST_NAME`과 서비스가 사용중인 도메인명이 같지만, `ELB`는 외부와 관계없이 `VPC`내부의 `EC2`내부 인스턴스 ip (Private IP)로 요청을 보내기 때문에 `ALLOWED_HOSTS`에 추가되지 않은 `EC2`의 PrivateIP는 Block되어 모든 `Health check`가 실패했다는 결과로 나타난다.

(정확히는 `Django`에서 `Bad Request(400)` Response를 보내준다)

## Add EC2 Private IP to ALLOWED_HOSTS

따라서 `Django`애플리케이션이 동작하는 각 `EC2`인스턴스에서는 자신의 `PrivateIP`를 `ALLOWED_HOSTS`에 추가해야 한다.

해결법은 [이 글을 참조](https://hashedin.com/2017/01/06/5-gotchas-with-elastic-beanstalk-and-django/)했으며, `Python3`에 맞도록 약간의 코드를 수정했다.

```python
# settings.py
ALLOWED_HOSTS = []

def is_ec2_linux():
    """Detect if we are running on an EC2 Linux Instance
       See http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/identify_ec2_instances.html
    """
    if os.path.isfile("/sys/hypervisor/uuid"):
        with open("/sys/hypervisor/uuid") as f:
            uuid = f.read()
            return uuid.startswith("ec2")
    return False


def get_linux_ec2_private_ip():
    """Get the private IP Address of the machine if running on an EC2 linux server"""
    from urllib.request import urlopen
    if not is_ec2_linux():
        return None
    try:
        response = urlopen('http://169.254.169.254/latest/meta-data/local-ipv4')
        ec2_ip = response.read().decode('utf-8')
        if response:
            response.close()
        return ec2_ip
    except Exception as e:
        print(e)
        return None
        
private_ip = get_linux_ec2_private_ip()
if private_ip:
    ALLOWED_HOSTS.append(private_ip)
```

설정 변경 후 `eb deploy`로 새 버전을 배포하면 `Health check`가 정상적으로 동작하는 것을 확인할 수 있다.

![OK]({{ site.url }}/images/elb-health-check-for-django/ok.png)