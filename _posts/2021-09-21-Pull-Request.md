---
layout: article
title: 오픈소스 기여를 위한 git Pull Request(풀리퀘스트) 방법 - Commit, Push, Rebase 등
tags: [git, 오픈소스아카데미]
aside:
    toc: true
---

2021년 하반기 오픈소스 컨트리뷰션 아카데미에 참여하게 되었습니다😎👏. <br/>
[Microsoft Azure SDK](https://github.com/Azure) 프로젝트의 조원이 되어 우선 번역 작업에 기여하고 있는데요, <br/>
아카데미 초반에 진행한 git 교육 내용 + 멘토님이 알려주신 내용 + 제가 처음으로 PR 날리면서 시행한 과정을 정리해서 올려 봅니다.

## Fork & Clone
### Fork
공식 오픈소스 프로젝트 저장소의 Fork 버튼을 눌러 내 개인 저장소로 복사합니다. (Github 상에서만 일어나는 작업입니다.)

![Github Fork](/assets/images/posts/2021-09-21_pr_1_fork.png)

### Clone
공식 프로젝트를 Fork한 나의 개인 저장소의 프로젝트를 Clone(다운로드)합니다.

우선, Clone URL 복사합니다. 왼쪽 위 저장소 이름에서 나의 개인 저장소인지 확인하고, Code 버튼을 누른 후 URL을 복사해줍니다.
![Github Clone](/assets/images/posts/2021-09-21_pr_2_clone.png) <br/>

git 명령어로 저장소를 Clone 합니다.
``` shell
# 디렉토리 이동
$ cd [프로젝트 복사할 디렉토리]

# 프로젝트 복사
$ git clone [Github 저장소 URL]
```

## Branch 생성
작업의 시작을 위해 Branch(브랜치)를 생성합니다. 작업 내용을 대표하는 키워드로 생성하기를 추천합니다. (예시는 Java implementation 문서 번역에 참여했을 때 작성한 이름입니다.)
```shell
# Branch 생성
$ git checkout -b java/impl

# 현재 Branch 내역 확인
# 앞에 *이 붙은 Branch가 현재 Branch
$ git branch
* java/impl
  main
```

## 소스 파일 수정하기
각자 생성한 Branch에서 소스 파일을 수정합니다.

### Add
수정한 파일을 추가합니다.
```shell
# status(현재 소스 파일의 상태를 확인하는 명령어)로 수정한 파일을 확인
# 자세한 변경 내역은 git diff 명령어로 확인 가능 (종료하려면 q)
$ git status
On branch java/impl
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   docs/java/implementation.md

# 위의 명령어로 출력한 modified 파일 중 Commit에 반영할 파일을 추가
$ git add [수정한 파일 경로]
```

### Commit
`add` 명령어로 수정한 파일을 모두 추가한 후, 수정 내역(Commit)을 만듭니다. <br/>
`commit` 할 때 메시지는 다른 개발자와 소통하기 쉽도록 명확하게 적어 줍니다.
```shell
# 옵션 없이 commit 명령어를 실행하면 vi 에디터로 메시지 작성
$ git commit

# -m 옵션으로 commit 메시지를 바로 작성
$ git commit -m "Translate API Implementation of java implementation into Korean"


# 내가 작성한 commit을 확인 (종료하려면 q)
$ git show
```

> 💡 `commit` 메시지 작성 시, Github에서 제공하는 기능으로 Issue - PR 사이를 연동할 수 있습니다. 메시지에 `closes #이슈번호` 등으로 함께 적어주면 됩니다. 자세한 사항은 [Github Doc](https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue)을 참고하세요!

### 참고: Stash
`stash`는 수정한 내용을 임시 저장해두는 명령어입니다. Commit stage에 올리기는 애매하지만 전후 상황을 비교해보고 싶을 때 사용합니다. (저의 경우 가이드라인 프로젝트 빌드를 위해 Gemfile을 수정해야 했는데, 이 내용을 Commit 내역에 올릴 수 없어 사용했습니다.)

```shell
# 수정한 내용을 임시 저장
# status 명령어 실행 시 modified 내역이 없다고 뜹니다.
$ git stash

# 수정한 내용 복구
$ git stash pop
```

## Rebase
내가 소스 파일을 수정하는 도중, 다른 사람이 소스 파일을 수정하여 공식 저장소에 반영될 가능성이 있습니다. 즉, 내가 Clone 받은 프로젝트가 더이상 최신 소스가 아닐 수 있습니다. 이 경우, 충돌이 일어나거나 프로젝트 빌드가 안 될 수 있습니다. 이를 해결하기 위해 프로젝트를 미리 최신 소스 수정 내역으로 업데이트하는 작업이 `rebase`입니다. <br>
저는 Push 하고 Pull request를 하기 전에, 미리 Rebase 작업을 해주었습니다. (PR 후에 아래 과정을 진행해도 PR이 자동으로 갱신됩니다.)

우선, 공식 오픈소스 저장소를 등록합니다.
```shell
# upstream으로 공식 프로젝트 URL을 등록
$ git remote add upstream [공식 프로젝트 URL]

# 원격 저장소 설정 현황 확인
# origin: Fork로 복사한 나의 개인 저장소
# upstream: 오픈소스 공식 저장소
$ git remote -v
origin  https://github.com/i-hope9/azure-sdk-korean.git (fetch)
origin  https://github.com/i-hope9/azure-sdk-korean.git (push)
upstream        https://github.com/Azure/azure-sdk-korean (fetch)
upstream        https://github.com/Azure/azure-sdk-korean (push)
```

Rebase 작업을 해줍니다.
```shell
# main branch로 이동
$ git checkout main

# 공식 upstream 저장소에서 최신 commit 내역 가져오기 (main에서 main으로)
$ git pull upstream main:main

# 내가 생성한 branch로 이동
$ git checkout java/impl

# 최신 commit history 기준으로 갱신
$ git rebase main
```

## Push
이제 내가 수정한 내역을 나의 개인 저장소에 Push합니다. 내가 수정한 Branch 이름을 명시해야 합니다.
```shell
$ git push --force origin java/impl:java/impl
```

## Pull Request
Push 작업을 마치면, 개인 저장소 Github 페이지에 가면 Compare & pull request 버튼이 생깁니다. <br/>
![Github Fork](/assets/images/posts/2021-09-21_pr_3_pr.jpg) <br/>

버튼을 눌러 수정 내역을 작성한 후 Create pull request 버튼을 누릅니다. 내용 작성 시에는 어떤 부분을 왜 변경했는지 자세하게 작성합니다. <br/>
![Github Fork](/assets/images/posts/2021-09-21_pr_4_pr.jpg) <br/>

> 💡 참고로 스크린샷의 'WIP'는 'Work in Progress', 즉 진행 중의 약자입니다. 아직 해당 브랜치의 작업이 완료되지는 않은 상태를 의미합니다. 컨트리뷰션 진행 내역을 확인하기 위해 다음과 같은 약어를 사용할 수도 있습니다. (만약 한 Commit에 ㅊㅊ한 PR의 규칙이 있는 오픈소스 프로젝트는 이와 같은 약어를 사용하지 않을 수도 있습니다.)

## Merge 후 Branch 삭제
Pull request를 완료하면 공식 오픈소스 저장소의 Owner(혹은 다른 개발자)가 Review를 합니다. <br/>
Review 이후 Merge가 되면? Pull request의 한 사이클이 끝납니다👏! <br/>
Merge가 완료되면 아래의 명령어로 작업하던 브랜치를 삭제해줍니다.

```shell
$ git branch -d [브랜치이름]
```

***
_Pull request 사진, Review 반영 등 추가 내용 작성 예정입니다._

<!--more-->
## 참고자료
+ [초보몽키 개발공부로그](https://wayhome25.github.io/git/2017/07/08/git-first-pull-request-story/)

