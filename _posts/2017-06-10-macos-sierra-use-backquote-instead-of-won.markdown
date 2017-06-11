---
layout: post
title:  "macOS에서 backquote(`)를 입력할 때 원(₩)기호 입력 방지하기"
categories: ['macOS']
---

![한글키보드]({{ site.url }}/images/backquote.png)

`macOS`의 버전이 `Sierra`로 올라가며 한글키보드에서는 backquote(``` ` ```)가 기본적으로 `₩`으로 입력되도록 바뀌었다.  
이를 막으려면 아래와 같이 폴더와 파일을 추가해준다.



- `~/Library`에 `KeyBindings`폴더를 추가
- `~/Library/KeyBindings`에 `DefaultkeyBinding.dict`파일 생성
- 아래 코드를 추가

```text
{
  "₩" = ("insertText:", "`");
}
```

출처 : [A2 Devlog](https://ani2life.com/wp/?p=1753){:target="_blank"}
