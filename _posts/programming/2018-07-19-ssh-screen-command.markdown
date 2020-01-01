---
layout: post
title:  "ssh를 사용해서 screen세션 생성 후 명령어 실행"
categories: ['기타']
---

```
# 필요시 실행중이던 세션 종료
ssh <Host> -C 'screen -X -S <Session name> quit'

# detach모드로 세션 실행
ssh <Host> -C 'screen -S <Session name> -d -m'

# 실행중인 세션에 stuff로 명령어 전달. 마지막에 개행(\n)전달
ssh <Host> -C "screen -r <Session name> -X stuff $'<Shell Command>\n'"
```
