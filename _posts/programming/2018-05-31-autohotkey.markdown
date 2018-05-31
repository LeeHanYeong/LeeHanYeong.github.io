---
layout: post
title:  "AutoHotkey 간단 사용법"
categories: ['기타']
---

AutoHotkey는 윈도 응용프로그램에서의 반복작업을 위한 스크립트 언어이다. 최근 윈도 매크로를 만들 일이 있어 10일 정도 다뤄보고 삽질했던 부분들을 간단히 정리해본다.

---

## 기본사항

**변수에 값 대입**

```
변수명 := 데이터
변수명 = 데이터 와는 결과가 다름. 문서를 정확히 보진 않았으나 대부분의 경우 위의 기호를 사용해야 했음
ex) Var := 35
```

**랜덤값 만들기**

```
Random, <변수>, <최소값>, <최대값>
ex) random, Var, 1, 100
    -> Var변수에 1 ~ 100중 하나의 값이 할당
```

**함수 호출 시 변수의 값 전달은 `%`사이에 변수를 넣는다**

`MsgBox, ABCDE`와 같은 명령문은 `MsgBox`함수에 `ABCDE`라는 문자열을 전달한다. 만약 `ABCDE`라는 변수의 값을 전달하고 싶다면 아래와 같이 사용

```
MsgBox, %ABCDE%
```

**리스트, For loop등의 인덱스는 1부터 시작**

**if문에서 비교 시 `=`하나만 사용**

```
if (ABC = 0) {
  MsgBox, ABC is 0
}
```

**함수 호출 시, 기본 매개변수 값이 있더라도 반드시 쉼표로 해당 값을 비워야 호출가능**

> 개인적으로 가장 별로였던 부분

```
DoSomething(A, B, C:=5, D:=10, E:=100) {
C, D값을 생략하고 호출시)
  DoSomething(30, 50, , , 120)
```

## 마우스 조작

**마우스 움직이기**

```
MouseMove, <x좌표>, <y좌표>
```

**마우스 클릭**

```
ControlClick, x<x좌표값>, y<y좌표값>, , <누를 버튼>
; 누를버튼은 Left, Right와 같은 텍스트를 입력
```

## 그 외

Gui기반 언어라서인지 모르지만, 로그를 출력하는 기능을 자체적으로 제공하지 않는다. 직접 Gui를 구성하고 그 인터페이스 위에 요소를 출력하는게 가장 보기 좋았지만, `MsgBox`라는 내장 함수를 사용하면 간단히 메시지를 출력할 수 있다.

**메시지 출력(윈도우 팝업)**

```
MsgBox, <출력내용>
```

**Sleep기능**

```
Sleep, <milliseconds>
```

**이미지 찾기 기능**

```
ImageSearch, <결과x좌표변수>, <결과y좌표변수>, <검색시작x좌표>, <검색시작y좌표>, <검색끝x좌표>, <검색끝y좌표>, *<민감도> <찾을 이미지 경로>
```
검색에 성공했을 경우 해당 스코프에 ErrorLevel이라는 변수가 생기며, 0이 할당된다.

풀 예제는 아래와 같다

```
ImageSearch, x1, y1, 0, 0, A_ScreenWidth, A_ScreenHeight, *30 %A_ScriptDir%\images\02.png
if (ErrorLevel = 0) {
  MsgBox, Search succcess, Position: (%x1%, %y1%)
}
```

여기서 쉼표로 구분된 부분과 띄어쓰기로 구분된 부분은 전부 지켜주어야한다(..............)


**Gui만들기**

```
Gui, Add, GroupBox, x22 y19 w130 h50 , Run count
Gui, Add, DropDownList, x32 y39 w110 h300 vRunCount, %RunCountList%
Gui, Add, GroupBox, x22 y79 w130 h50 , Reinforcement
Gui, Add, DropDownList, x32 y99 w110 h300 vReinforcementCount, %ReinforcementCountList%
Gui, Add, Button, x22 y139 w130 h30 gStartFunction vStartButton, Start
Gui, Show, x919 y13 h423 w588, New GUI Window
```

간단한 명령들로 Gui를 구성할 수 있지만, 좌표를 직접 잡아주어야 한다.

`Smart GUI Creator`라는 프로그램으로 기초를 잡은 뒤, 상세사항은 스크립트에서 수정하는것이 훨씬 간단. 코드로 옮긴 Gui도 다시 불러와서 편집할 수 있지만 그 경우 파일이 덮어쓰기 되므로 Gui를 담은 파일은 별개로 작성하고 `Include`로 불러오는 것이 좋다.
