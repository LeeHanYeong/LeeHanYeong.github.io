---
layout: post
title:  "특정 Port의 프로세스 죽이기"
categories: ['Linux']
---

```
kill -9 $(lsof -t -i:8000)
```