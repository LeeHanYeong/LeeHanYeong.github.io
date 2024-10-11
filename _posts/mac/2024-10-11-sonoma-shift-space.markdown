---
layout: post
title:  "macOS Sonoma에서 Shift+Space로 한/영 전환"
categories: ['macOS']
---

### vscode에서 Binary Plist설치

<img src="../../images/2024-10-11-sonoma-shift-space/vscode.png" style="max-width: 300px;">



### Terminal에서 키보드 단축키 plist파일 열기

```shell
❯ open ~/Library/Preferences/com.apple.symbolichotkeys.plist
```



### Restricted모드 해제

제한모드에서는 plist파일을 열어도 Binary Plist가 해석해주지 못함. 하단의 Restricted Mode 클릭 → In a Trusted Window의 Trust버튼 클릭

![vs-res1](../../images/2024-10-11-sonoma-shift-space/vs-res1.png)

![vs-res2](../../images/2024-10-11-sonoma-shift-space/vs-res2.png)



### 이후 다시 plist파일 열기

```shell
❯ open ~/Library/Preferences/com.apple.symbolichotkeys.plist
```

![vs-res3](../../images/2024-10-11-sonoma-shift-space/vs-res3.png)

Open Anyway → Text Editor클릭

![vs-res4](../../images/2024-10-11-sonoma-shift-space/vs-res4.png)



### plist파일의 key 60, 61번 수정

60 → 393216  
61 → 131072

```xml
		<key>60</key>
		<dict>
			<key>enabled</key>
			<true/>
			<key>value</key>
			<dict>
				<key>parameters</key>
				<array>
					<integer>32</integer>
					<integer>49</integer>
					<integer>393216</integer>
				</array>
				<key>type</key>
				<string>standard</string>
			</dict>
		</dict>
		<key>61</key>
		<dict>
			<key>enabled</key>
			<true/>
			<key>value</key>
			<dict>
				<key>parameters</key>
				<array>
					<integer>32</integer>
					<integer>49</integer>
					<integer>131072</integer>
				</array>
				<key>type</key>
				<string>standard</string>
			</dict>
		</dict>
```

저장 후 재부팅 (로그아웃으로 적용 안됨. 반드시 재부팅)



### 키보드 설정 수정

지구본 모양 누를때 → 안 함




![settings1](../../images/2024-10-11-sonoma-shift-space/settings1-8628461.png)

### 키보드 단축키 확인

![settings2](../../images/2024-10-11-sonoma-shift-space/settings2.png)

Ctrl + Shift + Space와. 
Shift + Space

적용 확인
