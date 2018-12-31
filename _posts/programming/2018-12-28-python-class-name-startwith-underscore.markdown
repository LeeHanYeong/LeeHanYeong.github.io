---
layout: post
title:  'Underscore(_)로 시작하는 파이썬 클래스명'
categories: ['Python']
---

언더스코어로 시작하는 클래스명이 파이썬 코드 컨벤션에 맞는지 여기저기 질문하면서 찾은 관련 내용을 기록 겸 포스팅합니다.

- [Python Korea에 올린 질문글](https://www.facebook.com/groups/pythonkorea/permalink/2030613773688440/)

## PEP8

> 페이스북 질문에서 [양영필](https://www.facebook.com/yypbd)님이 달아주신 댓글 내용입니다

PEP8에 명시적으로 언더스코어가 붙은 클래스가 정의되어 있지는 않지만, 문서의 일부에서 언더스코어로 시작하는 클래스에 대한 언급이 있습니다.

- [https://www.python.org/dev/peps/pep-0008/#public-and-internal-interfaces](https://www.python.org/dev/peps/pep-0008/#public-and-internal-interfaces)

> Even with \_\_all\_\_ set appropriately, internal interfaces (packages, modules, classes, functions, attributes or other names) should still be prefixed with a single leading underscore.

## 파이썬 표준라이브러리 코드

> [youknowone](https://github.com/youknowone)님이 알려주신 내용입니다

파이썬 표준라이브러의 코드에서 이 방식으로 클래스를 사용하고 있습니다.

**[https://github.com/python/cpython/blob/master/Lib/functools.py#L439](https://github.com/python/cpython/blob/master/Lib/functools.py#L439)**

```python
class _HashedSeq(list):
    ...
```

## PyCharm

파이참에서 언더스코어로 시작하는 클래스명을 작성 시 문법 Warning을 보여주지 않습니다(...)

## 마치며

찾으면서 새롭게 안 사실인데, 언더스코어로 시작하는 변수나 클래스는 `from <module> import *`과 같은 Asterisk를 사용하는 import문에 자동으로 불러와지지 않는다고 합니다.
