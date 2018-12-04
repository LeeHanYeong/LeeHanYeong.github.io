---
layout: post
title:  'boto3를 사용한 AWS SecurityGroup의 Inbound Rule추가 (Django Command구현)'
categories: ['AWS', 'Django']
---

기본적으로 RDS의 보안그룹은 외부 접속을 허용하지 않도록 생성하나, 편의를 위해 VPC외부에서의 접속을 허용하도록 설정 할 때가 있습니다. 이 때, 최소한의 보안을 위해 개발/사용중인 IP만을 Inbound규칙에 추가하도록 하는 파이썬 코드를 Django Command로 동작하도록 작성해보았습니다.

```python
from urllib import request

import boto3
from django.conf import settings
from django.core.management import BaseCommand


class Command(BaseCommand):
    def handle(self, *args, **options):
        ip = request.urlopen('https://ident.me').read().decode('utf-8')
        session = boto3.session.Session(
            profile_name=settings.AWS_EB_SESSION_PROFILE,
            region_name='ap-northeast-2',
        )
        ec2 = session.resource('ec2')
        sg_rds = ec2.SecurityGroup(settings.AWS_RDS_SECURITY_GROUP_ID)
        exist_permissions = sg_rds.ip_permissions
        if exist_permissions:
            sg_rds.revoke_ingress(IpPermissions=exist_permissions)
        sg_rds.authorize_ingress(
            IpPermissions=[{
                'FromPort': settings.DATABASES['default']['PORT'],
                'ToPort': settings.DATABASES['default']['PORT'],
                'IpProtocol': 'TCP',
                'IpRanges': [{
                    'CidrIp': f'{ip}/32',
                }],
                'UserIdGroupPairs': [{
                    'GroupId': settings.AWS_EB_ENVIRONMENTS_SECURITY_GROUP_ID,
                    'UserId': settings.AWS_EB_USER_ID,
                }],
            }],
        )
```
> 위 코드는 `F-string`을 사용하였기 때문에 파이썬 3.6이상의 버전에서만 동작합니다. 이전의 파이썬에서 사용하고자 할 경우, `F-String` format을 이전 버전이 지원하는 String format으로 변경해야 합니다.

`settings`로부터 가져오는 값 들은 소스코드와 관계없는 AWS에서의 고유값들이며, Git에 포함되지 않도록 별도의 비밀값으로 관리하는것을 권장합니다. 관련 내용은 이전에 작성한 [Django에서 비밀 값 관리하기](https://lhy.kr/django-secrets)에서 확인할 수 있습니다.

위 예제에서는 사용중인 RDS의 SecurityGroup을 `sg_rds`변수에 할당하고, 기존에 존재하던 Inbound규칙이 있을 경우 `revoke_ingress`메서드를 사용해 모든 Inbound rule을 삭제합니다. 이후, `authorize_ingress`메서드의 `IpPermissions`인수에 `IpRanges`와 `UserIdGroupPairs`를 사용해 각각 **IP**와 **SecurityGroupID**를 추가합니다. 

커맨드를 실행하는 컴퓨터가 속한 네트워크의 PublicIP는 `https://ident.me`에 HTTP요청을 보낸 결과로 알아내며, **UserIdGroupPairs**에 들어가는 **GroupId**와 **UserId**는 각각 보안그룹과 IAM화면에서 확인 할 수 있습니다.
