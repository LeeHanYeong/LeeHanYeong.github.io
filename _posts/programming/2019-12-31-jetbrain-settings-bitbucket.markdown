---
layout: post
title:  "JetBrains IDE의 Settings Repository로 Bitbucket저장소 사용하기"
categories: ['IDE']
---

JetBrains IDE의 **Settings Repository**플러그인을 사용하면, 사용자 설정을 Git저장소를 사용해서 관리할 수 있다.

여기서는 Bitbucket의 App passwords를 사용해 저장소를 설정하는 방법에 대해 설명한다.



## App password 발급

아래와 같이 Settings - App passwords - Create app password로 이동해 새 패스워드를 만들어준다.

![app-password]({{ site.url }}/images/jetbrain-settings/app-password.png)





## 저장소로 사용할 Repository생성

새 Repository를 만든다.

![app-password]({{ site.url }}/images/jetbrain-settings/repo.png)





## IDE설정

![app-password]({{ site.url }}/images/jetbrain-settings/ide-menu.png)

메뉴 클릭 후

![app-password]({{ site.url }}/images/jetbrain-settings/ide-url.png)

**!!!! URL입력 시, `https://bitbucket.org/<계정명>/<Repo명>.git` 의 형태가 되도록 한다 !!!!**

> Bitbucket에서 Clone버튼을 누르면 나오는 URL은 `https://<계정명>@bitbucket.org/<계정명>/<Repo명>.git` 인데, 여기서 앞의 `<계정명>@` 부분을 삭제해주어야 한다.

![app-password]({{ site.url }}/images/jetbrain-settings/ide-credential.png)

username, password입력창에서 **Bitbucket 계정명**(이메일주소와는 다름)과 발급받은 **App password**를 사용한다.

![app-password]({{ site.url }}/images/jetbrain-settings/mac-keychain.png)

(macOS의 경우) 입력한 내용은 키체인에 저장된다.