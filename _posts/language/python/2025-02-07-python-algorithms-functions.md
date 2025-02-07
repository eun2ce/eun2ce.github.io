---
title: "[python] 자주 사용되는 python 알고리즘 함수 정리"
date: 2025-02-07 12:00:00 +0900
categories: [ "language", "python" ]
tags: [ "algorithm", "function", "python", "사용법", "프로그래밍" ]
pin: false
math: false
mermaid: false
---

알고리즘 문제 풀이에 자주 사용되는 함수들을 정리하여 레파토리 코드를 활용하고, 이를 통해 개념을 익히고 추후 적용할 수 있도록 합니다.

# 순열(permutations)과 조합(combinations) - itertools

| 구분 | 순열(permutations) | 조합(combinations) |
|----|------------------|------------------|
| 순서 | 중요함              | 중요하지 않음          |
| 중복 | 없음               | 없음               |

## 순열 - itertools.permutations(iterable, r)

`iterable`에서 순서를 고려한 `r`개의 원소로 이루어진 순열을 반환합니다. 순서가 중요합니다.

```python
import itertools
data = [1, 2, 3]
result = list(itertools.permutations(data, 2))  
# 출력: [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]
```

## 조합 - itertools.combinations(iterable, r)

`iterable`에서 중복 없이 `r`개의 원소로 이루어진 조합을 반환합니다. 순서는 중요하지 않습니다.

```python
import itertools
data = [1, 2, 3]
result = list(itertools.combinations(data, 2))  
# 출력: [(1, 2), (1, 3), (2, 3)]
```
