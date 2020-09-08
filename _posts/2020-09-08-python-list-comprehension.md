---
title: "Python List Comprehension"
date: 2020-09-08 18:03:00
tags: Python
---

# 1. 개요

리스트 컴프리헨션(List Comprehension)은 python 의 list 를 다루는 가장 유용한 구문으로, python 의 내장 기능으로 가벼울 뿐만 아니라 가독성이 좋다는 이유로 많이 사용됩니다. 실제로 리스트 컴프리헨션을 **잘** 사용하면 code line 수가 줄어들기 때문에 대체로 더 깔끔해지고 가독성도 좋아지게 됩니다.

> **잘** 사용하지 못 한다면 오히려 가독성이 떨어지는 상황이 발생할 수도 있습니다.

-----

# 2. 기본

리스트 컴프리헨션은 기존 리스트로부터 특정 조건을 걸어서 새로운 리스트를 생성하는 경우 사용하며, 다음과 같은 Syntax 를 가집니다.

```python
[ expression for element in list if condition_1 if condition_2 ... ]
```

리스트 컴프리헨션을 사용하지 않고 같은 구문을 풀어서 쓰면 다음과 같이 `for`문과 `if`문을 통해 표현할 수 있습니다.

```python
result = []
for element in list:
    if condition_1:
        if condition_2:
            result.append(expression)
```

기본 Syntax 만 보더라도 리스트 컴프리헨션이 훨씬 가독성이 좋은 것을 알 수 있습니다. 게다가 `expression` 또는 `condition` 의 꼴이 복잡한 경우라면 더 깔끔해집니다.

-----

# 3. 예시

예시는 리스트 컴프리헨션 표현식과 그 결과만 간단하게만 살펴보겠습니다. 예시를 눈으로 보는 것보다는 직접 `expression` 과 `condition` 을 조작해보면서, 리스트 컴프리헨션에 익숙해지는 것을 추천드립니다.

- 기본 형태
```python
[i for i in range(1,11)]
# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

- if 문 활용
```python
[i for i in range(1,11) if i % 2 == 0]
# [2, 4, 6, 8, 10]
```

- if 문 2개 활용, 가독성을 위해 line 분리
```python
[i for i in range(1,11)
    if i % 2 == 0
    if i < 5]
# [2, 4]
```

- for 문 2개 활용, 가독성을 위해 line 분리
```python
[(i, j) for i in range(1,11)
        for j in range(1, 3)
        if i % 2 == 0]
# [(2, 1), (2, 2), (4, 1), (4, 2), (6, 1), (6, 2), (8, 1), (8, 2), (10, 1), (10, 2)]
```
