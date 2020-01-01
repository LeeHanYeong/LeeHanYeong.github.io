---
layout: post
title:  "brew에서 postgresql 업그레이드 후 DB접속 문제 해결"
categories: ['macOS']
---

`postgresql`의 버전이 업그레이드 되면, 분명히 `brew`를 이용해 `postgreql`이 서비스로 동작하고 있음에도 `psql`명령어 등을 사용했을 때 DB에 접속할 수 없다는 에러가 발생한다.

**`접속 에러 메시지`**

```
➜  ~ psql instagram
psql: could not connect to server: No such file or directory
	Is the server running locally and accepting
	connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

이 경우 아래와 같이 `brew switch`명령어를 사용해서 사용하는 `postgresql`의 버전을 선택해준다.

**`설치된 버전들 확인`**

```
➜  postgresql cd ~/
➜  ~ cd /usr/local/Cellar/postgresql
➜  postgresql ls -al
total 24
drwxr-xr-x   7 lhy  admin   238 11 13 10:56 .
drwxrwxr-x  45 lhy  admin  1530 10 26 10:48 ..
drwxr-xr-x  12 lhy  admin   408 10 26 10:48 10.0
drwxr-xr-x  12 lhy  admin   408 11 13 10:52 10.1
-rw-------   1 lhy  admin   324 11 13 10:56 pg_upgrade_internal.log
-rw-------   1 lhy  admin   179 11 13 10:56 pg_upgrade_server.log
-rw-------   1 lhy  admin   179 11 13 10:56 pg_upgrade_utility.log
```

**`버전 교체`**

```
➜  postgresql brew switch postgresql 10.1
```