---
layout: post
title:  'Underscore(_)로 시작하는 파이썬 클래스명'
categories: ['Python']
---

언더스코어로 시작하는 클래스명이 파이썬 코드 컨벤션에 맞는지 회사 동료분들과 상의하며 찾은 관련 내용을 기록 겸 포스팅합니다.

## PEP8

- [https://www.python.org/dev/peps/pep-0008/#public-and-internal-interfaces](https://www.python.org/dev/peps/pep-0008/#public-and-internal-interfaces)

> Even with __all__ set appropriately, internal interfaces (packages, modules, classes, functions, attributes or other names) should still be prefixed with a single leading underscore.

## 파이썬 표준라이브러리 코드

**https://github.com/python/cpython/blob/master/Lib/functools.py#L439**

```python
class _HashedSeq(list):
    ...
```

PEP8에 명시적으로 언더스코어가 붙은 클래스가 정의되어 있지는 않지만, 문서의 일부에서 언더스코어로 시작하는 클래스에 대한 언급이 있고 표준 라이브러리에서 사용하며 파이참에서 오류를 내지 않는 것으로 보아 공식 스타일이 맞는것으로 보입니다.

찾으면서 새롭게 안 사실인데, 언더스코어로 시작하는 변수나 클래스는 `from <module> import *`과 같은 Asterisk를 사용하는 import문에 자동으로 불러와지지 않는다고 합니다.
