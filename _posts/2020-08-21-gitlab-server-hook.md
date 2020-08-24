---
title: "GitLab Server-Side Hook 을 통해 Commit Message Convention 강제하기"
date: 2020-08-21 14:40:00
tags: CI/CD
---

# 1. 개요

팀 단위로 프로젝트를 시작할 때, 효율적인 협업을 위해 커밋 컨벤션을 지키자고 하지만 사실 `git log` 를 확인해보면 잘 지켜지지 않는 팀이 많습니다.

커밋 컨벤션을 지키는 것이 협업 시에는 중요하고 효율적인 이유를 모두가 한 번씩은 들어봤지만, 바빠서, 귀찮아서, 별로 중요한 것 같지 않아서 등등 나름의 이유들로 실제로는 등한시하는 개발자 분들이 많습니다.

그렇다면 팀장 이상의 관리자의 입장에서 모든 팀원이 커밋 컨벤션을 완벽하게 지키게 만들고 싶다면 어떻게 해야 할까요?

일정 주기로 `git log` 를 확인해가며 지키지 않은 팀원에게 불이익을 주면 해결할 수 있을까요?

다들 동의하시겠지만 이러한 **후처리** 방식은 구시대적인 방식이기도 하고, 앞으로 같은 실수를 반복하는 일이 생기지 않는다는 보장을 할 수 없습니다.
따라서 **선처리** 방식으로 커밋 컨벤션을 지키지 않은 커밋은 아예 `master branch` 에 반영될 수 없도록 시스템을 만드는 것이 중요합니다.

오늘은 git 의 내장기능 중 하나인 **git hook**, 그 중 **server-side hook**을 사용해서 이러한 시스템을 구축하는 방법을 알아보겠습니다.

-----

# 2. 커밋 컨벤션을 지켜야 하는 이유

먼저 커밋 컨벤션이 무엇이고, 왜 지켜야 하는지에 대해서는 좋은 글이 많기에 간략하게만 짚고 넘어가겠습니다.

본 글에서 말하는 커밋 컨벤션은 흔히 말하는 커밋 메시지 컨벤션을 말합니다. 커밋 컨벤션은 [좋은 커밋 메시지의 7가지 룰](https://chris.beams.io/posts/git-commit/)에 나와있듯이 '제목과 본문은 한 줄 띄어쓰세요', '제목은 명령조로 작성하세요' 등과 같은 약속이 일반적이지만, 구체적인 내용은 팀 상황마다 조금씩 다를 수 있습니다.

그렇다면 커밋 컨벤션은 왜 지켜야 하는 걸까요?

일반적으로는 보기에 깔끔하고, 협업 시 커뮤니케이션할 때도 좋고, 추후에 디버깅이나 유지보수할 때도 좋다라는 개발자 관점에서의 이유들이 많이 알려져 있지만, 사실 사용자 입장에서도 굉장히 중요합니다.

개발자들은 개떡같이 작성된 커밋 메시지를 찰떡같이 알아보기는 어려울지라도, 실제 코드를 보고 `git blame`, `git log` 등을 뒤져보거나 오프라인 커뮤니케이션 등을 통해 어찌저찌 파악할 수는 있습니다.

하지만 소프트웨어의 패치노트만을 읽고 사용하는 일반 사용자는 개떡같이 작성된 커밋 메시지들로 구성된 패치노트만으로 자세한 사항을 파악하기는 굉장히 어려울 뿐더러 실제 운영 중인 환경에 잘못 사용하게 된다면 금전적 피해로 이어질 수도 있습니다.

본인이 개발한 소프트웨어에 대한 책임감이 있다면 이런 상황이 발생하게 놔둬서는 안 되겠죠.

-----

# 3. 강제하는 세 가지 방법

하지만 좋은 커밋 메시지를 작성해야 한다는 것에 동의를 했더라도, 실제 개발을 하다보면 실수로, 까먹어서, 바빠서, 귀찮아서 등등의 여러 이유로 지켜지지 않는 상황이 자주 발생하게 됩니다.

그렇기에 선처리 방식으로 지켜지지 않은 커밋은 `master branch` 에 반영될 수 없도록 사전 방지하는 시스템을 구축해놓는 것이 좋습니다.

로컬(git-client) 에서 작업한 commit 을 git-server 의 `master branch` 에 반영하는 과정에서 이를 검사하고 막는 방법은 크게 3 가지가 있습니다.

1. **git-client 단에서 체크**
2. **git-server 단에서 체크**
3. **제 3 의 서버에서 체크**

이 중 1, 2 번은 **Git Hook** 이라는 내장 기능을 사용하는데, 간단히 말하면 git 의 특정 action (commit, push, receive, rebase 등)이 발생하면, 정해놓은 스크립트를 수행하게 설정하는 것을 말합니다.

**Git Hook** 스크립트를 잘 작성하면 다양한 작업을 자동화할 수 있는데, 좋은 예시 중 하나가 [다음 문서](https://woowabros.github.io/tools/2017/07/12/git_hook.html)에 작성되어 있습니다.

## 1) git-client 단에서 체크

![client](/assets/images/2020-08-21/client.png){: width="90%" height="90%"}

먼저 client 단에서 체크하는 방법은 위 문서의 내용과 같이 `pre-commit` 혹은 `pre-push` 훅을 사용해서, 체크하는 방식입니다.

자주 사용되는 방식이며, 이를 설정하는 [husky](https://github.com/typicode/husky) 라는 좋은 오픈소스도 존재합니다.

하지만 각각의 client 들이 로컬 머신에 설정해주어야 한다는 귀찮음이 존재하며, hook 을 설정하지 않은 client 의 push 를 막지 못한다는 문제가 있습니다.
더군다나 컨벤션이 바뀌거나 하는 경우마다 지속적으로 모든 client 가 설정파일을 변경해주어야 하는 문제도 존재합니다.

## 2) git-server 단에서 체크

![server](/assets/images/2020-08-21/server.png){: width="90%" height="90%"}

두 번째 방법은 server 단에서 client 의 요청을 거부하는 것인데, 간단하게 말씀드리면 client 가 `git push` 할 때, id & password 가 틀리면 server 에서 push 를 거부하며 password 가 틀렸다고 응답을 주는 것과 비슷한 로직입니다.

id & password 를 체크하는 인증/인가 프로세스 대신 커밋 컨벤션 체크 프로세스를 거치는 것이라고 생각하시면 됩니다.

이 방식은 client 에서는 아무런 작업을 하지 않아도 되고, server 에서만 hook 을 설정해놓으면 된다는 장점이 존재합니다.

## 3) ci-server 단에서 체크

![ci-server](/assets/images/2020-08-21/ciserver.png){: width="90%" height="90%"}

세 번째 방법은 git 외에 jenkins 등의 ci-server 를 두고 막는 방법인데, 2)와 비슷하지만 큰 차이점이 존재합니다.

2)의 경우에는 `pre-receive` 라는 hook 을 사용하면 server 로 `push` 자체를 막을 수 있는데, 3)의 경우는 `push` 를 막지는 못하고 `merge` 를 막는 방식으로 진행됩니다. `pull request` 또는 `merge request`를 날릴 때, 화면에서 `merge` 버튼이 비활성화되는 형태가 일반적입니다.

대부분의 github 을 쓰는 오픈소스 프로젝트들은 호스팅 서버를 따로 두지 않고 github 에서 제공하는 호스팅 서버를 사용하기에 3)의 방식을 쓰는 경우가 많습니다.

-----

# 4. Gitlab Server-side hook

세 가지 방법 모두 장단점이 존재하지만, 개인적으로 1)은 client 가 많아지면 각자 설정해주어야 한다는 게 불편하다는 이유로, 3)은 merge request 단계에서 막는 것보다 더 앞인 push 단계에서 막고 싶다는 이유로, 2)를 사용하는 것이 좋다는 결론을 내렸고, GitLab 을 사용하고 있기에 **GitLab Server Hook** 사용법을 알아보았습니다.

[공식 문서](https://docs.gitlab.com/ce/administration/server_hooks.html)에 따르면 GitLab CE 에서는 다음 3 가지 Server Hooks 을 제공합니다.

- pre-receive
- post-receive
- update

3 가지 Hook 의 작동 방식은 모두 동일합니다. hook 별로 정해진 event 가 발생하면 해당 hook 에 적힌 스크립트를 수행하는 것입니다.

차이점은 수행되는 시점이 다르다는 점인데, 이 중 **push 요청을 받은 즉시 모든 branch 에 대해서 수행**하는 Hook 인 `pre-receive` 를 설정하겠습니다.

다른 2 가지 Hook 역시 파일명이 다르다는 점을 제외하고는 모든 설정방법은 동일합니다.

예를 들어, push 는 허용하고, 그 이후에 스크립트가 수행되길 원하신다면, 아래 내용을 동일하게 따라하시되, 파일명만 `pre-receive` 대신 `post-receive` 로 변경하여 사용하시면 됩니다.

-----

# 5. Gitlab Server hooks 설정하기

GitLab Server Hook 은 말 그대로 Server 에 설정하는 작업이기 때문에, 해당 Server 의 파일시스템 접근 권한이 있어야 합니다. Server Hook 은 각 repository 별로 설정할 수도 있고, 모든 repository 에 일괄적으로 설정할 수도 있지만, 본 글에서는 특정 repository 하나에만 설정하는 경우를 다루겠습니다.

## 설정 순서

먼저 설정하길 원하는 repository 가 server 의 어느 디렉토리에 해당하는지 찾아야 합니다.

해당 경로는 GitLab Server 설정에 따라 다르기 때문에 자세한 내용은 다음 [문서](https://github.com/gitlabhq/gitlabhq/blob/master/doc/administration/repository_storage_types.md#translating-hashed-storage-paths)와 [해시 스토리지 경우 문서](https://github.com/gitlabhq/gitlabhq/blob/master/doc/administration/repository_storage_types.md#translating-hashed-storage-paths)를 참고 바랍니다.

- 1) 해당 Repository 의 파일 시스템 경로를 찾았다면, 해당 경로로 이동해주시기 바랍니다. 보통 다음과 같은 구조를 지니고 있습니다.

```shell
root@gitlab:/var/opt/gitlab/git-data/repositories/@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.git# ls 
HEAD  config  description  hooks  info  language-stats.cache  objects  refs
```

- 2) `custom_hooks` 라는 이름으로 폴더를 생성하고 이동합니다.

```shell
$ mkdir custom_hooks
$ cd custom_hooks
```

- 3) `pre-receive` 라는 이름의 빈 파일을 생성합니다.

```shell
$ touch pre-receive
```

- 4) 해당 파일을 실행 가능하게 만들고, git user 에게 권한을 줍니다.

```shell
$ chmod +x pre-receive
$ chown git:git pre-receive
```

- 5) `pre-receive` 파일을 열어 컨벤션을 체크하는 스크립트를 작성합니다.

```shell
$ vi pre-receive
```
- pre-receive 는 system 의 stdin 을 읽어서 들어온 push 이벤트의 커밋 정보, ref 정보 등을 parsing 한 뒤, 정해진 컨벤션을 만족하는지 regex 등을 사용해 체크하고, 조건을 만족한다면 0, 그렇지 않다면 0 이 아닌 값을 return 하는 형태로 작성하면 됩니다.
- 자세한 예시는 다음 장에서 확인하실 수 있습니다.

-----

# 6. pre-receive hook 예시

- 다음 스크립트는 python3 로 작성된 hook 입니다. 해당 hook 을 사용할 경우, gitlab-server 에 python3 설치가 필요합니다.
  - [github 링크](https://github.com/anencore94/gitlab-server-hook)
  - 해당 repository 의 `pre-receive.py` 파일 내용을 copy 하여 gitlab server 의 custom hook 경로에 `pre-receive` 라는 파일명으로 추가하면 작동됩니다.

현재 버전(v1.0)은 다음과 같은 컨벤션을 체크하고 있습니다.
- new tag 생성, new branch 생성, merge 커밋, revert 커밋을 제외한 모든 커밋에 대해서 컨벤션 체크를 수행하게 됩니다.
- Commit Title 이 다음 `[New Feature]`, `[BugFix]`, `[Refactor]`, `[Style]`, `[Documentation]`, `[TypoFix]` 중 하나로 시작하는지를 검사합니다. (띄어쓰기 구분 없으며, 대소문자는 구분)
  - 원하는 컨벤션에 맞게 코드 내의 regex 를 수정하여 사용하시면 됩니다.
  - 추후 커밋 타이틀 외에 커밋 메시지까지 체크하는 코드로 발전될 예정입니다.

-----

# 7. 참조

- https://github.com/gitlabhq/gitlabhq/blob/master/doc/administration/server_hooks.md
- https://docs.gitlab.com/ce/administration/server_hooks.html
- https://bryan.wiki/297
- https://github.com/DragOnMe/gitlab-server-hooks