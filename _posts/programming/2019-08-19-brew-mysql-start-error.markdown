---
layout: post
title:  "brew로 설치한 MySQL의 mysql.server start오류 해결"
categories: ['기타']
---

```
brew remove mysql
brew cleanup
launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
rm ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
sudo rm -rf /usr/local/var/mysql

brew install mysql
```

출처: [https://stackoverflow.com/a/36156848](https://stackoverflow.com/a/36156848)
