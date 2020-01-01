---
layout: post
title:  "Ubutnu 외장 하드 자동 마운트"
categories: ['기타']
---

외장 하드를 연결하고 UUID 확인

```shell
sudo blkid
```

**`sudo vi /etc/fstab`**

```shell
UUID=<확인한 UUID> /<마운트될 경로> <포맷> defaults 0 0
ex) UUID=sdfesdf23-2d /mount-hdd ex4 defaults 0 0
```

뒤쪽의 defaults, 0, 0은 어떤 값인지 아직 미확인

- 출처: [https://medium.com/@jjeaby/외장-하드-자동-마운트-2bfdc58961c](https://medium.com/@jjeaby/외장-하드-자동-마운트-2bfdc58961c)