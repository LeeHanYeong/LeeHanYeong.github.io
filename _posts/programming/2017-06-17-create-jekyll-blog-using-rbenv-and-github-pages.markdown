---
layout: post
title:  "rbenv환경에서 Jekyll 블로그 생성하고 GitHub Pages에 배포하기"
categories: ['기타']
---

이 포스팅에서는 `macOS`환경에서 `rbenv`를 사용해 `Jekyll`블로그를 생성하고, 이를 `GitHub Pages`에 배포하는 방법을 다룬다.  

---
## Jekyll?
`Jekyll`은 일반적으로 블로그를 만들기 위해 사용하는 정적 사이트 생성기이다.  
데이터베이스를 사용하지 않고 `git`과 같은 버전관리 시스템을 사용해 포스트들을 관리하며, 정적 웹사이트이기 때문에 단순 파일 서빙만으로 블로그를 만들 수 있다.

정적 웹 사이트라는 특징덕에 `GitHub`에서는 `Jekyll`블로그를 서빙하기 위한 시스템을 제공하며 무료로 사용가능하다.

또한 포스트(소스)의 형태를 `HTML`이 아닌 개발자 친화적인 다른 마크업(대부분의 경우 `Markdown`)을 사용할 수 있으며, 데이터베이스를 쓰지 않는다는 점은 역으로 버전관리 시스템을 사용해서 작성한 글 들을 편리하게 관리할 수 있다는 장점을 준다.

데이터베이스를 쓰지 않기 때문에 댓글기능은 타사 서비스를 사용하거나(`disqus`) 직접 만들어야 한다.

## 설치
`Jekyll`을 로컬환경에서 실행하기 위해서는 특정 버전 이상의 `Ruby`가 필요하다. `macOS`에 기본설치된 `Ruby`는 버전이 낮기때문에, 시스템의 `Ruby`를 업그레이드하거나 `Ruby`의 버전관리도구를 사용하는 방법이 있다.  
개인적으로 시스템에 이미 설치된 파이썬이나 루비를 변경하는것을 선호하지 않기 때문에, 여기서는 `Ruby`의 버전관리도구중 하나인 `rbenv`를 사용한다.

#### Homebrew설치
`rbenv`를 설치하기 위해 macOS용 패키지 관리자인 `Homebrew`를 이용한다.  

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- `Homebrew`에 대한 자세한 내용은 [공식 페이지](https://brew.sh/index_ko.html)에서 확인 할 수 있다.
- 이미 `Homebrew`가 설치되어 있는경우 `brew update`를 이용해 `Homebrew`를 최신버전으로 업데이트해준다.


#### 이미 설치되어있던 ruby삭제
이미 `Homebrew`를 사용하던 경우, 앞으로 `rbenv`에서 `Ruby`의 실행을 관리할 것이므로 기존에 설치된 `Ruby`를 지워준다.

```
brew uninstall ruby
```

#### rbenv설치

```
brew install rbenv ruby-build
```

#### rbenv를 위한 설정을 셸 설정파일에 추가
`zsh`을 쓰는 경우 `~/.zshrc`, 기본 `bash`를 쓰는 경우 `~/.bash_profile`에 작성한다.

```
# rbenv
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
```

**설정파일을 작성한 후 터미널을 종료하고 새로 열어준다**

#### rbenv를 이용해 ruby설치, 전역에서 사용할 ruby버전 지정
```
➜ rbenv install 2.4.1
➜ rbenv global 2.4.1
➜ rbenv versions
  system
* 2.4.1 (set by /Users/lhy/.rbenv/version)
```

`*`표가 붙은 부분이 현재 사용하고 있는 `Ruby`버전을 나타낸다.

#### 새 ruby버전 설치 후 rehash작업
```
rbenv rehash
```

`rehash`는 `rbenv`가 관리하는 루비 명령어들을 `~/.rbenv/shims`디렉토리에 셸 스크립트 파일로 복사해주는 역할을 한다.  
새 `gem`을 설치하거나 새 `Ruby`버전을 설치한 경우 해당 명령을 실행해주어야 하며, 위의 셸 설정파일에 추가한 `rbenv init -`명령어에는 `rehash`기능이 포함되어있다. 터미널을 새로 열면 자동으로 `rehash`명령이 수행되므로 위 명령어를 실행하거나 새 터미널 창을 열면 된다.

#### RubyGems, Gem, bundler
[RubyGems](https://rubygems.org/)는 `Ruby`패키지 관리자이며, 각 패키지는 `Gem`이라고 불린다.
설치해야 할 `Gem`은 `jekyll`과 `bundler`, `github-pages`이다.

일반적으로 `Ruby`의 패키지들은 `RubyGems`를 이용해 관리되며, 시스템 어디에서나 설치된 `Gem`들을 사용할 수 있다.  
하지만 프로젝트별로 필요한 각 패키지(`Gem`)들의 버전이 다를 수 있으며, 이러한 의존성 문제를 해결하기 위해 [bundler](http://ruby-korea.github.io/bundler-site/)라는 패키지를 사용한다. (`bundler`역시 `RubyGem`의 패키지이다)  
`bundler`는 `Gemfile`과 `Gemfile.lock`파일을 이용해 현재 프로젝트 폴더에서 사용하는 패키지 버전들을 관리한다.

`github-pages`는 `GitHub`에서 사용하는 `Jekyll`과 관련된 의존성 패키지들을 지원하는 `Gem`이다. 로컬에서 `Jekyll`을 이용해 사이트를 생성할때와는 다르게, `GitHub`에 코드를 업로드 했을 때 자동으로 사이트가 생성될 때는 `GitHub`에서 사이트가 생성되기 위한 여러 요소들이 필요한데, 이러한 부분들을 처리해주기 위한 지원 패키지 역할을 한다.

```
gem install jekyll bundler github-pages
```

## Jekyll블로그 생성 및 로컬 실행
필요한 `Gem`들을 설치한 후, `Jekyll`블로그를 생성할 폴더의 상위 폴더에서 아래 명령을 실행하고 결과를 확인한다.

```
➜ jekyll new <블로그명>
➜ cd <블로그명>
➜ ls -al
drwxr-xr-x   9 lhy  staff   306B  6 17 17:13 .
drwxr-xr-x@ 61 lhy  staff   2.0K  6 17 17:12 ..
-rw-r--r--   1 lhy  staff    35B  6 17 17:13 .gitignore
-rw-r--r--   1 lhy  staff   953B  6 17 17:13 Gemfile
-rw-r--r--   1 lhy  staff   1.2K  6 17 17:13 Gemfile.lock
-rw-r--r--   1 lhy  staff   1.4K  6 17 17:13 _config.yml
drwxr-xr-x   3 lhy  staff   102B  6 17 17:13 _posts
-rw-r--r--   1 lhy  staff   525B  6 17 17:13 about.md
-rw-r--r--   1 lhy  staff   213B  6 17 17:13 index.md
```

`jekyll`패키지 대신 `github-pages`를 사용하도록 `Gemfile`의 내용을 수정해준다.  
`gem "jekyll"...`부분은 주석처리하고, `gem "github-pages"...`부분을 활성화시킨다.

```
➜ vi Gemfile
...
# gem "jekyll", "3.4.3"
# This is the default theme for new Jekyll sites. You may change this to anything you like.
gem "minima", "~> 2.0"

# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
gem "github-pages", group: :jekyll_plugins
...
```

실행 전 `bundle`로 관리되는 패키지들을 업데이트 시켜준다

```
bundle install
bundle update
```

실제 정적 사이트를 생성하고, 테스트를 위한 로컬 서버를 실행한다.

```
bundle exec jekyll serve
```

[127.0.0.1:4000](127.0.0.1:4000)으로 접속하면 아래와 같은 페이지를 확인할 수 있다.  
![Jekyll_Welcome]({{ site.url }}/images/jekyll-welcome.png)

## 글 작성
`Jekyll`의 글은 기본적으로 `Markdown`을 사용하며, `_posts`폴더내부에서 `YYYY-MM-DD-title.markdown`형식을 가진 파일 하나당 하나의 글이 된다.

기본예제에 글이 하나 있으니 해당 내용을 기반으로 새 글을 작성해본다. 이 포스팅에서는 `Jekyll`에서의 글 작성법이나 문법은 다루지 않는다. 자세한 내용은 아래의 링크를 참조한다.

[Jekyll 공식사이트 (영문)](https://jekyllrb.com/)  
[Jekyll 공식사이트 (한글)](https://jekyllrb-ko.github.io/)


## GitHub에 배포
로컬에서 `jekyll serve`명령어를 사용하면 `_sites`폴더가 생성되며, 해당 폴더의 내용들이 정적 사이트 생성 결과물이 된다.  
`GitHub Pages`에 배포할 때는 이 결과물이 아닌 `Jekyll`소스 자체를 사용한다.
`Git`및 `ssh`의 정의와 사용법에 대해서는 설명하지 않는다.

#### 저장소 생성
`GitHub Pages`를 이용해 `Jekyll`블로그를 만들 때는, `<자신의 유저명>.github.io`라는 이름의 저장소를 만들어야 한다.  
ex)`LeeHanYeong`사용자의 저장소 이름은 `LeeHanYeong.github.io`라는 이름을 가져야 한다.


![Pages_Repository]({{ site.url }}/images/github-new-repository.png){:width="253px"}
![Pages_Repository]({{ site.url }}/images/github-pages-repository.png){:width="595px"}


#### 로컬 git저장소 생성 및 .gitignore작성
`Jekyll`블로그 폴더에서 실행한다. 리모트 저장소 주소를 `git@`으로 시작하고 싶다면 `GitHub`계정에 `SSH-Key`가 등록되어 있는 상태여야 한다.

`Ruby`, `macOS`, `Jekyll`에서 `Git`으로 버전관리에서 제외할 파일 목록을 위해 `.gitignore`파일을 생성해준다

```
vi .gitignore
```
내용에는 [.gitignore(Ruby, Jekyll, macOS)](https://gist.github.com/LeeHanYeong/acb428567ba1b01d55ed9a078e46b32f){:target="_blank"}의 내용을 붙여넣는다.  


#### 리모트 저장소에 push
`.gitignore`파일을 작성한 후 아래와 같이 리모트 저장소에 로컬 저장소의 내용을 업로드한다.

```
➜ git init
➜ git add -A
➜ git commit -m 'First commit'
➜ git remote add git@github.com:LeeHanYeong/LeeHanYeong.github.io.git
➜ git push origin master
```

#### 작동확인
![Jekyll_GitHub_Pages]({{ site.url }}/images/jekyll-github-pages.png){:width="952px"}
