---
title: "Git 조금 더 자세히 알아보기 (1)"
date: 2021-04-17 12:00:00
tags: Linux
---

> Git 의 내부 동작을 조금 더 자세히 이해해보는 시리즈의 첫 번째 글입니다.

---

## Git 조금 더 자세히 알아보기 (1)

### 0) 대세 형상관리 툴 : Git

- 소프트웨어 혹은 관련 문서 등 소프트웨어 생명 주기에서 만들어지고 사라지는 모든 결과물에 대해 그 형상을 기록하고 변경 내역을 관리하는 작업을 쉽게 해주는 툴을 형상관리 툴이라 부르며, CVS, SVN 등을 거쳐 현재는 Git 이 대세가 되었습니다.

<div style="width:70%; margin:0 auto;" align="center" markdown="1">
![git-vs-svn](/assets/images/2021-04-18/git-vs-svn.PNG)
*(2005 년 탄생한 git 의 검색 트렌드 변화)*
</div>

- SVN 에 비해 Git 의 가장 큰 장점은 remote repo 와 local repo 의 분리를 통해 분산/독립적인 작업이 가능하다는 점과 branch checkout, merge 등의 작업이 굉장히 가볍고 빠르게 이루어진다는 점입니다.
- 하지만 모든 소프트웨어가 그렇듯 Git 또한 몇 십년 동안 대세로 남을 수는 없을 것이기에, 그 동안 Git 의 내부 구조에 대한 자세한 이해 없이 High-level Interface 만 사용해왔던 제 자신을 반성하며 Git 에 대해 조금 더 자세히 알아보는 시간을 가졌습니다.
  - `git push -f`, `git rebase -i`, `git stash`, `git reflog` 정도를 익히며 Git 을 잘 아는듯이 행동했던 과거의 나를 반성합니다..

- 도대체 Git 은 어떻게 동작하길래 `git reset --hard yesterday_commit`, `git merge my_friend's_branch`, `git checkout another_branch_1` 와 같이 매우 복잡해보이는 처리를 그렇게 빠르게 처리할 수 있는걸까요?
- 그리고 과연 앞으로 몇 년 동안 대세로 유지될 수 있을까요?

---

### 1) Git 의 2 가지 User Interface

**Porcelain & Plumbing**
- Git 을 사용해봤다 혹은 사용하고 있다하는 사람들이 실제 작업하며 흔히 사용하는 커맨드들은 대부분 **porcelain** 명령어입니다.

<div style="width:70%; margin:0 auto;" align="center" markdown="1">
![porcelain & plumbing](/assets/images/2021-04-18/plu-por.PNG)
*(porcelain & plumbing)*
</div>

- 하지만 이 둘을 strictly exclusive 하게 구분하기에는 살짝 애매한 부분이 있습니다. : [stackoverflow Q&A](https://stackoverflow.com/questions/39847781/which-are-the-plumbing-and-porcelain-commands/39848056#39848056))

- 그 이유는 Git 설계 시 의도적으로 이 두 가지 Interface 를 구분한 것이 아니라, Git 은 수많은/다양한 low-level commands (Plumbing commands) 를 재료로 제공해놓고, 유저가 원하는대로 plumbing commands 를 조합해서 사용하도록 설계했기 때문이라고 합니다.
- 이러한 조합 중에서 **사용성이 뛰어나 end-user 가 많이 사용하는 commands** 들을 porcelain commands 라고 부르는 것입니다.
- Git 을 활용해 소프트웨어 개발을 하는 대부분의 경우에는 porcelain commands 만 사용하는 것으로 충분하지만, Git 의 내부구조의 접근하기 위해서, 그리고 그 구조의 자세한 동작 방식을 이해하기 위해서는 plumbing commands 가 필요합니다.
    - 예를 들어, 작업 후 file 을 staging area 에 올리고(`git add`) local repo 에 커밋하는(`git commit`) 으로 이루어진 porcelain commands 작업은 다음과 같은 순서의 plumbing commands 로 이루어집니다.
        - git hash-object
        - git update-index
        - git write-tree
        - git commit-tree
        - git update-ref
- 이러한 각각의 plumbing commands 를 이해하기 위해서는 우선 Git 이 Version Control 을 하기 위해 **어떤 정보를, 어디에, 어떻게 기록**하고 있는지에 대한 내용부터 알아봐야 합니다.
- 사실상 Git 의 핵심은 VCS interface 자체보다는 여기에 있다고 볼 수 있습니다.

#### **<center><span style="color:red">'Git is fundamentally a content-addressable filesystem with a VCS user interface written on top of it'</span></center>**

(https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)

- 그럼 다음 글에서는 git 의 핵심인 여러 objects 들에 대해 본격적으로 알아보겠습니다.


---

## References

- [Git scm book - Git Internals](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)
- [깃의 속사정, 4대 원소를 파헤치기](https://storycompiler.tistory.com/7)
- [Git plumbing commands](https://storycompiler.tistory.com/7)
- [Stackoverflow - Which are the plumbing and porcelain commands?](https://stackoverflow.com/questions/39847781/which-are-the-plumbing-and-porcelain-commands)