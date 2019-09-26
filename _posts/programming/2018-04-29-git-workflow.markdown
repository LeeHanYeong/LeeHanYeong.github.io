---
layout: post
title:  "Git을 이용한 협업 워크플로우"
categories: ['Git']
---

Git을 사용한 협업 방법에 대해 가장 많이 참조하던 글의 이미지가 깨져있어, CC라이센스상 문제가 없기에 번역문에서 이미지 링크를 수정하고 Subversion과의 비교를 제외하여 작성했습니다.

- [원문(영어), Atlassian - Getting Git Right - Comparing Workflows](https://www.atlassian.com/git/tutorials/comparing-workflows)
- [번역문(한글), Git을 이용한 협업 워크플로우 배우기](http://blog.appkr.kr/learn-n-think/comparing-workflows/)

---

이 글에서는 여러 엔터프라이즈 개발팀을 조사하여 정리한 대표적인 Git 협업 워크플로우를 소개합니다. 여기서 제시하는 워크플로우들은 엄격한 규칙이라기보다는 상황에 적절한 워크플로우를 선택하기 위한 일종의 가이드입니다.
<!-- ![]({{ site.url }}/images/git-workflow/.svg) -->

---

# 1. Centralized Workflow

![Centralized Workflow]({{ site.url }}/images/git-workflow/centralized-workflow.svg)

Centralized Workflow는 프로젝트에 단일 중앙 저장소를 사용하며, `master`브랜치 하나만 사용합니다.

팀 구성원들은 중앙 저장소를 복제하여 로컬 저장소를 만들고, 로컬 저장소에서 변경 내용을 커밋하고 언제든 중앙 저장소와 동기화합니다.

![Centralized and local]({{ site.url }}/images/git-workflow/centralized-local.svg)

## 1.1 충돌 처리

![Centralized Conflicts]({{ site.url }}/images/git-workflow/centralized-conflict.svg)

중앙 저장소의 커밋이 기준이 되므로, 로컬 저장소의 변경 내용을 푸시하려 할 때, 저장소의 커밋 이력과 충돌한다면 Git은 저장소의 커밋을 보호하기 위해 푸시를 거부합니다.

이 때는 중앙 저장소의 변경 내용을 먼저 로컬 저장소로 가져와서(fetch) 자신의 변경 내용을 합치거나(Merge) 재배열(Rebase)해야 합니다. 

머지나 리베이스도중 중앙저장소의 변경 내역과 자신의 커밋 내역이 충돌한다면, 수작업으로 충돌을 해결해야 합니다. 충돌을 해결한 후에는 해당 사항을 새로운 커밋으로 만들어 머지나 리베이스를 완료하고 중앙 저장소에 푸시합니다.

## 1.2 적용 사례

철수와 영희, 두 명의 개발자로 구성된 작은 팀이 Centralized Workflow를 이용해 어떻게 협업하는 지 살펴봅니다. 그림에는 세 번째 인물이 있지만, 이번 사례에서는 등장하지 않습니다.

### 1.2.1 중앙 저장소 생성

철수나 영희 둘 중 한명이 중앙 저장소를 생성합니다. 일반적인 협업플로우에서는 `GitHub`이나 `BitBucket`과 같은 온라인 리모트 저장소를 활용합니다. 지금은 `GitHub`에 리모트 저장소를 만든다고 가정합니다.

![Centralized GitHub]({{ site.url }}/images/git-workflow/centralized-github.png){:.half}

![GitHub Quick setup]({{ site.url }}/images/git-workflow/github-quicksetup.png)

생성을 완료하면 이렇게 Git저장소 주소가 생성됩니다.
{:.img-caption}

### 1.2.2. 중앙 저장소 복제

모든 팀 구성원들은 `git clone`명령어로 중앙 저장소를 복제한 로컬 저장소를 만듭니다. 위에서 생성한 저장소의 주소를 사용합니다.

```
$ git clone git@github.com:LeeHanYeong/Centralized-Workflow.git centralized-workflow
```

### 1.2.3 철수의 작업

![철수 local]({{ site.url }}/images/git-workflow/cheolsoo-local.svg)

철수는 로컬 저장소에서 자신이 맡은 기능을 개발하고 변경 내용을 커밋합니다.

```
$ echo '철수의 작업' > work.txt
$ git add work.txt
$ git commit -m '철수의 작업 추가'
```

로컬 저장소에 커밋 할 때는 몇 번이고 커밋을 변경하며 내용을 수정해도 상관없습니다. 가능하다면 아주 작은 단위로 커밋하여 상세하게 프로젝트 이력을 유지하는 것이 좋습니다.

### 1.2.4 영희의 작업

![영희 local]({{ site.url }}/images/git-workflow/yeonghee-local.svg)

영희 역시 철수와 같이 로컬 저장소에서 자신이 맡은 기능을 개발하고 커밋합니다.

```
$ echo '영희의 작업' > work.txt
$ git add work.txt
$ git commit -m '영희의 작업 추가'
```

### 1.2.5 철수의 작업 내용 발행

![철수 publish]({{ site.url }}/images/git-workflow/cheolsoo-push.svg)

철수는 `git push`명령으로 자신의 로컬 커밋 이력을 중앙 저장소에 발행해 다른 팀 구성원과 공유합니다.

```
git push origin master
```

`origin`은 철수가 중앙 저장소를 복제할 때 자동 생성된 중앙 저장소의 별칭입니다. `master`는 현재 `push`명령을 실행하고 있는 브랜치의 내용을 중앙 저장소인 `origin`의 `master`브랜치에 동기화 하겠다는 뜻입니다. 

로컬 저장소에서 따로 브랜치를 생성하지 않았기 때문에, 지금은 로컬의 `master`브랜치 변경 내역을 `origin`의 `master`브랜치와 동기화 하게 됩니다. 

철수와 영희중 아무도 중앙 저장소를 변경하지 않았기 때문에, 철수의 푸시는 충돌없이 순조롭게 진행됩니다.

### 1.2.6. 영희의 작업 내용 발행

![영희 publish denied]({{ site.url }}/images/git-workflow/yeonghee-push-denied.svg)

철수가 푸시한 후 영희가 로컬 커밋을 푸시하면 어떻게 되는지 알아봅시다.

```
$ git push origin master
```

영희의 커밋 이력은 중앙 저장소의 최신 커밋 이력을 포함하고 있지 않기 때문에, 중앙 저장소는 영희의 푸시를 거부합니다. 이는 중앙 저장소의 커밋 이력을 보호하기 위한 장치입니다.

```
To github.com:LeeHanYeong/Centralized-Workflow.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:LeeHanYeong/Centralized-Workflow.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

영희는 중앙 저장소의 최신 커밋 이력을 로컬로 받아온 후, 자신의 로컬 커밋 이력과 통합하고 다시 푸시해야 합니다.

### 1.2.7. 영희의 fetch와 rebase

영희는 `git fetch`명령을 사용해 중앙 저장소의 변경 이력을 로컬 저장소에 내려받습니다. 

![영희 fetch]({{ site.url }}/images/git-workflow/yeonghee-fetch.svg)

```
$ git fetch origin
```

`fetch`명령어를 변경 이력을 로컬 저장소에 가져올 뿐, 현재 로컬 이력과 자동으로 합쳐주지 않습니다. 이를 `git log`명령어에 옵션을 사용해 알아봅니다.

```
$ git log --oneline --all --graph

* 1e36628 (HEAD -> master) 영희의 작업 추가
* 0fabb50 (origin/master) 철수의 작업 추가
(END)
```

`rebase`명령을 사용해 중앙 저장소의 커밋 이력을 영희의 커밋 이력 앞에 끼워넣습니다.

![영희 try rebase]({{ site.url }}/images/git-workflow/yeonghee-try-rebase.svg)

```
$ git rebase origin/master
```

철수와 영희 모두 `work.txt`라는 파일을 생성했기 때문에, Git은 해당 파일의 변경 내역을 자동으로 합쳐주지 못합니다. 이러한 파일의 충돌은 수동으로 해결해야 합니다.

```
First, rewinding head to replay your work on top of it...
Applying: 영희의 작업 추가
Using index info to reconstruct a base tree...
Falling back to patching base and 3-way merge...
Auto-merging work.txt
CONFLICT (add/add): Merge conflict in work.txt
error: Failed to merge in the changes.
Patch failed at 0001 영희의 작업 추가
Use 'git am --show-current-patch' to see the failed patch

Resolve all conflicts manually, mark them as resolved with
```


### 1.2.8. 영희의 충돌 해결

![영희 solve merge conflict]({{ site.url }}/images/git-workflow/yeonghee-solve-merge-conflict.svg)
충돌을 해결하고 `rebase`를 진행해서 위와 같은 그림을 만들어보겠습니다
{:.img-caption}

`git status`명령어를 사용해 어떤 파일에 충돌이 발생했는지 확인합니다.

```
$ git status
rebase in progress; onto 0fabb50
You are currently rebasing branch 'master' on '0fabb50'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git reset HEAD <file>..." to unstage)
  (use "git add <file>..." to mark resolution)

	both added:      work.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

`work.txt`파일을 양쪽에서 수정해서 Conflict가 발생했습니다. 영희는 해당 파일을 적절히 수정하고 해당 내역을 커밋으로 만들어야 합니다.

```
$ vi work.txt

<<<<<<< HEAD
철수의 작업
=======
영희의 작업
>>>>>>> 영희의 작업 추가
```

`<<<<<<< HEAD`에서 `========`까지의 내용은 저장소에 원래 존재하던 내용이며, `=========`이후 `>>>>>>>>> 영희의 작업 추가`사이의 내용은 `영희의 작업 추가`로컬 커밋의 내용입니다. 이 파일의 내용을 아래와 같이 수정합니다.

```
철수의 작업
영희의 작업
```

이제 변경사항을 Git에 추가하고, `rebase`를 다시 진행합니다.

```
$ git add work.txt
$ git rebase --continue
```

이제 `git log`를 사용해 작업 내역을 확인하면 `origin/master`의 뒤에 영희의 커밋 내역이 있는 것을 확인할 수 있습니다.

```
$ git log
commit 6fc9779c4b4c6a43e35ccb45f5d62502e0b3e878 (HEAD -> master)
Author: LeeHanYeong <dev@lhy.kr>
Date:   Sun Apr 29 19:29:31 2018 +0900

    영희의 작업 추가

commit 0fabb50ebbfef4c79567458d00de1509f3677c3b (origin/master)
Author: LeeHanYeong <dev@lhy.kr>
Date:   Sun Apr 29 19:28:51 2018 +0900

    철수의 작업 추가
```

중앙 저장소의 내역을 로컬에 전부 적용하고, 새 커밋을 만들었으니 중앙 저장소에 새 커밋 내역을 푸시합니다.

```
$ git push origin master
```

로컬 커밋이 중앙 저장소와 동기화 되었음을 최신 커밋의 상태로 확인할 수 있습니다.

```
$ git log
commit 6fc9779c4b4c6a43e35ccb45f5d62502e0b3e878 (HEAD -> master, origin/master)
Author: LeeHanYeong <dev@lhy.kr>
Date:   Sun Apr 29 19:29:31 2018 +0900

    영희의 작업 추가
```

## 1.3. 다음 단계

Centralized Workflow는 Git의 특장점인 분산 버전 관리의 이점을 사용할 수는 없지만, Git에 익숙하지 않을 때 최소한의 명령어로 협업을 진행해 볼 수 있습니다.

Centralized Workflow를 사용하되, 좀 더 유연하게 협업하려면 다음에 소개할 Feature Branch Workflow를 사용하는 것이 좋습니다. Feature Branch Workflow에서는 개발할 기능을 개별 브랜치로 분리함으로써 `master`브랜치에 새 기능을 병합하기 전 충분한 토론을 할 수 있는 장점이 있습니다.

# 2. Feature Branch Workflow

Feature Branch Workflow의 핵심은 기능별 브랜치를 만들어 작업하는 것입니다. `master`브랜치는 항상 버그 프리 상태로 유지하며, 병합시 권한을 가진 사용자가 풀 리퀘스트를 적용할 수 있습니다.

## 2.1. 작동 원리

이 워크플로우도 변경 이력을 기록하기 위해 하나의 중앙 저장소와 `master`브랜치를 사용합니다. 다만, `master`브랜치에 직접 커밋하지 않고 새로운 기능을 개발할 때마다 브랜치를 만들어 작업합니다. 보통은 `animated-menu-items`또는 `issue-#1061`처럼 의미를 담고 있는 브랜치명을 사용합니다.

Git은 `master`브랜치와 다른 브랜치들을 기술적으로 구분하지 않습니다. 따라서 Centralized Workflow에서 배운 스테이징, 커밋 등의 명령을 기능 개발 브랜치에 그대로 적용하면 됩니다.

새로 만든 기능 개발용 브랜치도 중앙 저장소에 올려 팀 구성원들과 개발 내용에 대한 의견(코드 리뷰 등)을 나눌 수 있습니다. `master`브랜치에 손대지 않기 때문에 다른 기능 브랜치를 얼마든지 올려도 됩니다. 이는 일종의 로컬 저장소 백업 역할도 겸합니다.

### 2.1.1. 풀 리퀘스트

브랜치를 이용하면 격리된 영역에서 안전하게 새 기능을 개발 할 수 있을 뿐만 아니라, 풀 리퀘스트를 이용해서 브랜치에 대한 팀 구성원들의 토론 참여를 이끌어 낼 수도 있습니다. 기능 개발을 끝내고 `master`에 직접 병합하는 것이 아니라, 브랜치를 중앙 저장소에 올리고 `master`에 병합해 달라고 요청하는 것이 풀 리퀘스트입니다.

풀 리퀘스트는 특정 브랜치에 대한 코드 리뷰의 시작점이라 볼 수 있습니다. 따라서 기능 개발 초기 단계에 미리 풀 리퀘스트를 보낸다고 문제될 것은 없습니다. 예를 들면, 기능 개발 중 막히는 부분에 대해 미리 풀 리퀘스트를 보내서 다른 팀 구성원들의 도움을 받을 수도 있습니다. 풀 리퀘스트에 포함된 커밋에 팀 구성원들이 의견을 제시할 수 있고, 새 의견이 등록되면 알림을 보낼 수도 있습니다. 이러한 기능은 Git자체의 기능이 아니라, Git을 호스팅하는 서비스에서 제공합니다.

## 2.2. 적용 사례

### 2.2.1. 영희의 작업

![영희 new branch]({{ site.url }}/images/git-workflow/yeonghee-new-branch.svg)

영희는 새 기능을 개발하기에 앞서 격리된 작업 브랜치를 만들고, 해당 브랜치로 체크아웃합니다.

```
$ git branch yh-feature
$ git checkout yh-feature
```

영희는 새 브랜치 `yh-feature`에서 작업을 하고 커밋을 남깁니다.

### 2.2.2. 영희의 점심시간

![영희 push branch]({{ site.url }}/images/git-workflow/yeonghee-push-branch.svg)

영희는 오전 동안 새로 만든 브랜치에 꽤 여러번의 커밋을 남겼습니다. 점심을 먹으로 가기 전, 작업을 중앙 저장소에 푸시해놓기로 합니다. 이는 로컬 저장소의 백업 역할을 할 뿐만 아니라, 다른 팀 구성원들이 미애의 작업 내용과 진도를 확인 할 수 있도록 합니다.

```
$ git push -u origin yh-feature
```

`yh-feature`브랜치를 중앙저장소에 푸시하는 명령입니다. `-u (--set-upstream)`옵션은 로컬 기능 개발 브랜치와 중앙 저장소의 같은 이름의 브랜치를 연결하는 역할을 합니다. 한 번 연결한 브랜치는 이후 `git push`명령만으로 푸시할 수 있습니다.

### 2.2.3. 영희의 기능 개발 완료

![영희 pull request]({{ site.url }}/images/git-workflow/yeonghee-pull-request.svg)

오후에 맡은 기능 개발을 모두 완료했습니다. `master`브랜치에 병합하기 전에 풀 리퀘스트를 올려 팀 구성원들에게 작업 완료 사실을 알립니다.

```
$ git push
```

### 2.2.4. 민수 팀장의 풀 리퀘스트 검토

![민수 review]({{ site.url }}/images/git-workflow/minsu-review.svg)

풀 리퀘스트를 확인한 민수 팀장이 `yh-feature`브랜치를 검토하다가, 병합하기 전에 몇 가지 수정이 필요하다고 판단하고 영희에게 해당 내역을 알려줍니다.

### 2.2.5. 영희의 수정 반영

![영희 modify]({{ site.url }}/images/git-workflow/yeonghee-modify.svg)

영희는 민수 팀장의 수정 요청 항목을 반영하고 커밋내역을 브랜치에 푸시합니다. 이 과정에서 수정 내용은 기존 풀 리퀘스트에 전부 표시되며, 민수 팀장도 수정 내용을 보고 언제든 새로운 의견을 제시합니다.

### 2.2.6. 영희가 개발한 기능의 병합 완료

![영희 merge]({{ site.url }}/images/git-workflow/yeonghee-merge.svg)

민수 팀장이 영희의 풀 리퀘스트를 수용하기로 결정합니다. 누군가 병합 작업을 진행합니다. (병합은 누구든 할 수 있습니다)

```
$ git checkout master
$ git fetch origin yh-feature
$ git merge origin/yh-feature
$ git push origin master
```

> `master`브랜치의 내용이 최신이라고 가정합니다. 만약 중앙 저장소의 `master`브랜치에 새로운 내용이 있다면, 먼저 `master`브랜치를 최신버전으로 동기화시켜줍니다.

먼저 `master`브랜치로 체크아웃하고, `yh-feature`브랜치의 내용을 로컬로 가져옵니다. 가져온 `yh-feature`브랜치의 내용을 로컬의 `master`브랜치와 병합하고, 로컬의 `master`브랜치에 새로 추가된 커밋 이력을 중앙 저장소에 푸시합니다.

이 과정에서 병합 커밋이 생길 수 있으며, 만약 추가 이력을 일직선으로 유지하고 싶다면 병합 전에 리베이스를 사용합니다.

### 2.2.7. 철수의 작업

철수 역시 다른 기능 브랜치를 만들어 자신이 맡을 기능을 개발합니다. 이 워크플로우는 격리된 브랜치로 안전하게 작업하며 중간 과정을 공유하기도 쉽습니다.

## 2.3. 다음 단계

더 큰 팀에서 규모있는 프로젝트를 관리할 때는 Gitflow Workflow를 사용해 기능 개발, 릴리스, 유지보수를 위해 좀 더 엄격한 워크플로우를 유지할 수 있습니다.

---

# 3. Gitflow Workflow

이번에 소개하는 Gitflow Workflow는 nvie.com의 빈센트 드리센이 제안한 것입니다.

Gitflow Workflow는 코드 릴리스를 중심으로 좀 더 엄격한 브랜칭 모델을 제시합니다. Feature Branch Workflow보다 복잡하지만, 대형 프로젝트에도 적용할 수 있는 강건한 작업 절차입니다.

## 3.1. 작동 원리

Gitflow Workflow도 팀 구성원간의 협업을 위한 창구로 중앙 저장소를 사용합니다. 또 다른 워크플로우와 마찬가지로, 로컬 브랜치에서 작업하고 중앙 저장소에 푸시하는 과정을 거칩니다. 단지 브랜치의 구조만이 다를 뿐입니다.

## 3.2. Master와 Develop브랜치

![Master and Develop branches]({{ site.url }}/images/git-workflow/master-develop.svg)

이 워크플로우에서는 두 개의 브랜치를 변경 이력을 유지하기 위해 사용합니다. `master`브랜치는 릴리스 이력을 관리하기 위해 사용하고, `develop`브랜치는 기능 개발을 위한 브랜치들을 병합하기 위해 사용합니다.

## 3.3. 기능(Feature) 브랜치

![Feature branches]({{ site.url }}/images/git-workflow/feature-branches.svg)

새로운 기능은 각각의 브랜치에서 개발하고 백업 및 협업을 위해 중앙 저장소에 푸시합니다. 이 때, `master`브랜치에서 새 브랜치를 만드는 것이 아니라 `develop`브랜치에서 새 브랜치를 시작합니다. 기능개발이 끝나면 다시 `develop`브랜치에 작업 내용을 병합합니다. 즉, 기능 개발을 위한 브랜치는 `master`브랜치와는 어떠한 상호작용도 하지 않습니다.

Feature Branch Workflow에서는 여기서 `develop`브랜치에 기능을 병합하는 것으로 모든 과정이 끝나지만, Gitflow Workflow에서는 아직 할 일이 더 남아 있습니다.

## 3.4. 릴리스 브랜치

![Release branches]({{ site.url }}/images/git-workflow/release.svg)

`develop`브랜치에 릴리스를 하기 위한 기능이 모이면, `develop`브랜치의 해당 시점을 기준으로 릴리스를 위한 브랜치를 만듭니다. 새로 만든 브랜치로부터 릴리스 사이클이 시작되며, 버그 수정 및 문서 추가 등 릴리스를 위한 준비가 완료되면 `master`브랜치에 병합하고 버전 태그를 부여합니다. 릴리스를 준비하는 동안 `develop`브랜치가 변경되었을 수 있으므로 `develop`브랜치에도 역시 함께 병합해줍니다.

릴리스를 위한 전용 브랜치를 사용함으로써 한 팀이 릴리스를 준비하는 동안 다른 팀은 다음 릴리스를 위한 기능 개발을 계속할 수 있습니다.

릴리스 브랜치는 `release-*`또는 `release/*`과 같은 이름을 가지는 것이 일반적인 관례입니다.

## 3.5. 핫픽스 브랜치

![Hotfix branches]({{ site.url }}/images/git-workflow/hotfix-branches.svg)

운영 환경에 릴리스한 후 발견된 긴급패치는 `hotfix`브랜치를 사용합니다. `hotfix`브랜치는 `master`브랜치에서 시작하며, 완성된 패치는 `develop`브랜치와 함께 운영 환경 코드인 `master`브랜치에 곧바로 병합합니다.

버그 수정만을 위한 브랜치를 따로 만들었기 때문에, 다음 릴리스를 위해 개발하던 작업 내용에는 전혀 영향을 주지 않습니다.


# 4. Forking Workflow

Forking Workflow는 다른 워크플로우와 근본적으로 다릅니다. 하나의 중앙 저장소를 이용하는 것이 아니라, 개개인마다 서로 다른 원격 저장소를 운영합니다. 모든 프로젝트 참여자가 개인적인 로컬 저장소와 공개된 원격 저장소, 두 개 씩의 Git 저장소를 가지는 방식입니다.

모든 코드 기여자가 하나의 중앙 저장소에 푸시하는 것이 아니라, 기여자 각각은 자신의 원격 저장소에 푸시하고 그 내용을 공식 저장소의 프로젝트 관리자에게 풀 리퀘스트를 보냅니다. 즉, 프로젝트 관리자는 다른 개발자들에게 공식 저장소에 푸시 권한을 주지 않고도 다른 개발자의 커밋을 수용할 수 있습니다.

프로젝트와 직접 관련이 없는 제 3자뿐만 아니라, 아주 큰 규모의 분산된 팀에서도 안전하게 협업하기에 좋은 방법입니다. 특히, 오픈소스 프로젝트에서 많이 사용하는 방식입니다.

## 4.1. 작동 원리

서버에 만든 공식 저장소로 시작한다는 점은 다른 워크플로우오 같습니다. 하지만, 다른 개발자가 이 프로젝트에 참여할 때는 이 공식 저장소를 직접 복제하지 않습니다.

대신 공식 저장소를 포크(fork)해서 자신만의 원격 저장소를 만듭니다. 이제 이 복제본 저장소는 개인의 공개 저장소 역할을 하고, 다른 개발자들은 이 원격 저장소에 푸시할 수 없습니다. 프로젝트의 참여자는 포크한 디렉토리로부터 `git clone`명령어를 사용해 로컬 저장소를 만듭니다. 다른 워크플로우들처럼, 이 로컬 저장소에서 작업을 수행합니다.

로컬 저장소의 커밋 이력을 원격 저장소에 푸시할 때는 프로젝트의 공식 저장소가 아니라, 자신의 원격 복제본에 푸시합니다. 그리고 프로젝트 관리자에게 자신의 기여분을 반영해달라는 풀 리퀘스트를 보냅니다.

## 4.2. 프로젝트 공식 저장소

Git은 기술적으로 공식과 기여자의 복제본을 구분하지 않기 때문에, 이 워크플로우에서 '공식'이란 상징적인 의미에 불과합니다. 프로젝트 관리자의 공개 저장소이기 때문에 공식이라는 단어가 붙은 것 뿐입니다.

## 4.3. Forking Workflow에서의 브랜치

각 기여자의 원격 저장소는 다른 기여자와 변경 내용을 공유하기 위한 편의 장치일 뿐입니다. 따라서, Feature Branch Workflow나 Gitflow Workflow처럼 새로운 기능 개발을 위해 격리된 브랜치를 만드는 것은 각자의 자유입니다. 대신, 브랜치를 다른 참여자와 공유하는 방법은 다릅니다. 다른 워크플로우에서는 공식 저장소에 브랜치를 푸시해서 팀 구성원들이 공유했다면, Forking Workflow에서는 나의 브랜치를 다른 참여자들이 자신의 로컬 저장소로 내려 받아 참고하고 병합할 수도 있습니다.

## 4.4. 다음 단계

이 글을 통해 어떤 개발자의 기여 활동이 어떻게 프로젝트의 공식 저장소에 반영될 수 있는지를 설명했습니다. 꼭 공식 저장소가 아니더라도 어떤 저장소에서든 이 방법을 적용할 수 있습니다. 예를 들어, 서브 팀이 특정 기능을 개발할 때, 메인 저장소를 건드리지 않고 팀 구성원들이 작업 내용을 공유할 수도 있습니다.

다른 개발자와 자신의 변경 내용을 쉽게 공유할 수 있고, 어떤 브랜치든 공식 코드 베이스에 병합할 수 있기 때문에, Forking Workflow는 느슨한 팀 구조에서도 강력한 협업 환경을 제공합니다.

---

**원문의 라이선스**

원문의 라이선스
Except where otherwise noted, all content is licensed under a [Creative Commons Attribution 2.5 Australia License](http://creativecommons.org/licenses/by/2.5/au/).

참고한 번역문과 이 글의 라이선스

[Cretarive Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)를 따릅니다.
