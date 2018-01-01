---
layout: post
title:  "Windows환경에서의 파이썬 환경설정"
categories: ['강의']
---

윈도 환경에서 파이썬을 설치하고, `Visual Studio Code`를 사용해 파이썬 개발 환경을 구축해봅니다.

## 파이썬 설치

### [파이썬 공식사이트](https://www.python.org/)에 접속

![python.org]({{ site.url }}/images/env-basic/windows/01.python.org.png)

### 파이썬 인스톨러 다운로드

![download]({{ site.url }}/images/env-basic/windows/02.python-download.png)

### 파이썬 설치

![install-01]({{ site.url }}/images/env-basic/windows/03.python-install-01.png)

- 다운로드 받은 파일을 실행

![install-02]({{ site.url }}/images/env-basic/windows/03.python-install-02.png)

- **Add Python 3.6 to PATH**를 체크
- Customize installation을 클릭

![install-03]({{ site.url }}/images/env-basic/windows/03.python-install-03.png)

- 위와 같이 체크되어 있는지 확인 후 Next 클릭

![install-04]({{ site.url }}/images/env-basic/windows/03.python-install-04.png)

- Advanced Options를 위 화면과 같이 설정
- **Customize install location**의 값에 주의

### 파이썬 설치 확인

![powershell]({{ site.url }}/images/env-basic/windows/04.powershell.png)

- 시작 -> 실행 (시작키 + R)후 `powershell`입력

![check]({{ site.url }}/images/env-basic/windows/05.python-check.png)

- 열린 셸에서 `python` 입력
- 위와 같은 화면이 나오는지 확인

## Visual Studio Code 설치

### [Visual Studio Code 공식사이트](https://code.visualstudio.com/)에 접속
![visual-studio-code-install-01]({{ site.url }}/images/env-basic/windows/06.visual-studio-code-install-01.png)

### 다운로드
![visual-studio-code-install-02]({{ site.url }}/images/env-basic/windows/06.visual-studio-code-install-02.png)

### 설치
![visual-studio-code-install-03]({{ site.url }}/images/env-basic/windows/06.visual-studio-code-install-03.png)

![visual-studio-code-install-04]({{ site.url }}/images/env-basic/windows/06.visual-studio-code-install-04.png)

- 추가 작업 선택화면에서 위와 같이 체크

## Visual Studio Code에 파이썬 경로 사용자 설정

### Visual Studio Code실행 후 파이썬 확장모듈 설치
![install-python]({{ site.url }}/images/env-basic/windows/07.install-python.png)

좌측 메뉴의 맨 아래 **확장**메뉴에서 `python`검색 후 설치, 완료 후 **다시로드**클릭

### Visual Studio Code에서 사용할 파이썬의 경로 확인
![powershell]({{ site.url }}/images/env-basic/windows/08.powershell.png)

`powershell`을 실행.

![which-python]({{ site.url }}/images/env-basic/windows/09.which-python.png)

파이썬의 경로를 확인. 위의 경우 `C:\Python36\python.exe`이며, 위의 설치 과정을 따라왔다면 같은 결과가 나와야 한다.

### 파이썬 경로 사용자 설정
![python-path-01]({{ site.url }}/images/env-basic/windows/10.python-path-01.png)

`파일 -> 기본 설정 -> 설정`으로 이동

![python-path-02]({{ site.url }}/images/env-basic/windows/10.python-path-02.png)

- 위의 검색창에 `python.pythonpath`를 입력
- 왼쪽 창에 나타나는 값의 왼쪽에 마우스 커서를 갖다 대면 나오는 연필모양 아이콘을 클릭
- 우측 '사용자설정'에 값을 입력할 수 있게 되며, 기존에 `python`으로 입력되어 있던 값을 `C:\\Python36\\python.exe`로 변경
- `Ctrl + s`를 눌러 설정파일을 저장
- Visual Studio Code를 완전히 종료했다가 재실행

## 프로젝트 폴더 생성 후 Visual Studio Code에서 열기

### 폴더 생성

- `C:\`로 이동해서 `projects`폴더 생성
- 방금 생성한 `C:\projects\`폴더로 이동 후 `python-basic`폴더 생성

### 생성한 폴더를 Visual Studio Code에서 열기

![open-folder]({{ site.url }}/images/env-basic/windows/11.open-folder.png)

- 파일 -> 폴더열기 버튼 클릭
- `python-basic`폴더가 보이는 곳 까지 이동해서 '폴더 선택'클릭

## 첫 번째 파이썬 코드 작성 후 실행

### `first.py`파일 작성

![first-python-file]({{ site.url }}/images/env-basic/windows/12.first-python-file.png)

**탐색기**아래의 `python-basic`폴더 우측의 `새 파일`아이콘을 클릭 후, 생성할 파일 이름을 `first.py`로 지정

![execute-first-python]({{ site.url }}/images/env-basic/windows/13.execute-first-python.png)

<br>
```python
print('Hello, world!')
```

위 코드를 편집기에 입력 후 파일을 저장

### `first.py`파일 실행

- 아래의 '터미널'에서 `python first.py`명령어 입력
- 터미널이 보이지 않는다면 `보기 -> 디버그 콘솔`버튼을 클릭 후 나오는 아래 메뉴에서 `터미널`을 클릭
- 결과에 `Hello, world!`가 나오는지 확인