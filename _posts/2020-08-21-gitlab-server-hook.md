---
title: "GitLab Server-Side Hook 을 통한 커밋 컨벤션 강제하기"
date: 2020-08-21 14:40:00
tags: CI/CD
---

>https://github.com/gitlabhq/gitlabhq/blob/master/doc/administration/server_hooks.md
>https://docs.gitlab.com/ce/administration/server_hooks.html
>https://bryan.wiki/297
>https://github.com/DragOnMe/gitlab-server-hooks

# 1. 개요

# 2. 커밋 컨벤션을 지켜야 하는 이유

# 3. 지킬 수 밖에 없게 만드는 세 가지 방법

# 4. Gitlab Server-side hook
- pre-receive
- update
- post-receive

# 5. Gitlab Server hooks 설정하기
## Respository 위치 찾기
- [링크](https://github.com/gitlabhq/gitlabhq/blob/master/doc/administration/repository_storage_types.md#translating-hashed-storage-paths)

- 구조
```shell
root@gitlab:/var/opt/gitlab/git-data/repositories/@hashed/6b/86# ls
6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.git
6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.wiki.git
```

```shell
root@gitlab:/var/opt/gitlab/git-data/repositories/@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.git# ls
HEAD  config  description  hooks  info  language-stats.cache  objects  refs
```

- mkdir `custom_hooks`

```shell
$ mkdir custom_hooks
$ cd custom_hooks
$ vi pre-receive
$ chmod +x pre-receive
$ chown git:git pre-receive
```

# 6. pre-receive hook 만들기
- `pre-receive` 스크립트를 python 으로 작성할 경우 gitlab server 에 python3 설치 필요

# 7. 예시
- 스크린샷들