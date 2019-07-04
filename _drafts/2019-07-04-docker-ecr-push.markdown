---
layout: post
title:  "AWS ECR에 Docker image push"
categories: ['AWS', 'Docker']
---

로컬에 로그인 되어있는 DockerHub계정과 별개로 ECR에 이미지를 Push하기 위한 명령어

```shell
$(aws ecr get-login --no-include-email --region ap-northeast-2) && docker push <image>
```



