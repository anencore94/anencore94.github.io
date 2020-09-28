---
title: "python process 'OOM killed error' 의 미봉책"
date: 2020-09-28 17:33:00
tags: Python Linux
---

# 1. 개요

heavy 한 deep learning 모델을 python 으로 구현하고 학습하다보면, GPU memory 혹은 RAM memory 의 부족으로 해당 python 프로세스가 강제로 종료되는 현상을 종종 보게 됩니다. 이러한 문제를 해결하는 가장 좋은 방법은 memory 를 많이 소모하는 code 를 최적화하거나, 더 좋은 GPU 혹은 RAM 으로 업그레이드하는 것이겠지만, 본 문서에서는 그러지 못한 경우 미봉책으로 사용할 수 있는 방법을 다룹니다.

-----

# 2. 증상
- `python xxx.py` 를 terminal 혹은 Pycharm 등의 IDE 에서 실행 시, 중간에 종료되며 OOM killer 에 의해 종료되었다는 에러 메시지가 출력된 경우
- `python xxx.py` 를 terminal 혹은 Pycharm 등의 IDE 에서 실행 시, 자세한 에러 메시지 없이 `sig killed (-9)` 라는 메시지만 출력되었으며, `/var/log/syslog` 를 확인해본 결과 OOM killer 에 의해 python process 가 종료된 것을 확인한 경우

-----

# 3. 임시 해결책

> 해당 해결책은 다음과 같은 문제가 있으므로 정말 필요한 경우에만 사용하고 함부로 사용하지 않는 것을 추천합니다.

> 본 방법은 oom killer 를 비활성화시키는 것이 아니라, oom killer 가 python 프로세스를 가장 마지막에 죽이도록 순서를 변경하는 것에 불과합니다. **따라서 memory 가 부족할 경우, python process 대신에 chrome, bash 등의 다른 process 들을 하나하나 죽이게 되며, 그래도 부족한 경우에는 결국 python process 까지 죽이게 됩니다.** 즉, 웬만하면 리소스를 업그레이드하는 것을 추천하며, **본 방법을 사용하기 전에 백업이 필요한 다른 프로세스는 모두 백업한 뒤 사용하시기 바랍니다.**

```shell
# memory 를 많이 사용하는 python 파일을 실행합니다.
$ python xxx.py

# 해당 python process 의 pid 를 확인합니다.
$ pidof python

# 해당 python process 의 oom killer 발동 순서를 가장 마지막으로 변경합니다.
$ sudo echo -17 > /proc/{$해당 파이썬 pid}/oom_adj
$ sudo echo -1000 > /proc/{$해당 파이썬 pid}/oom_score_adj
# 이제 해당 프로세스는 memory 부족 현상이 나타나더라도 제일 마지막에 kill 하게 됩니다.
```

-----

# 4. 참고
- [OOM killer 란?](https://mozi.tistory.com/28)
- [Linux 의 OOM killer](https://m.blog.naver.com/PostView.nhn?blogId=hanajava&logNo=220656756884&proxyReferer=https:%2F%2Fwww.google.com%2F)