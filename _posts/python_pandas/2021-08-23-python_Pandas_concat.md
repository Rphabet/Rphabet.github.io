---
title: python Pandas (9) DataFrame Concatenation
date: 2021-08-23 17:35:00 +0900
categories: [python, pandas]
tags: [python, numpy, pandas, merge, concatenateness] 
math: true
comments: true
image:
  src: /../assets/images/pandas/logo.png
  width: 500
  height: 200
  alt: numpy logo
typora-root-url: ../
---

---

## Series와 DataFrame을 연결(Concatenate)해보자

지난 포스팅은 하나의 '키 🔑 '값을 기반으로 두 개의 `DataFrame`을 병합해보았다.

이번에는 `Series`와 `DataFrame`을 단순히 연결하는 법을 배워보자.

### Series 연결해보기

 `pandas Series`를 사용해보자.

```python
import numpy as np
import pandas as pd

s1 = pd.Series([0,1], index=['a', 'c'])
s2 = pd.Series([4, 3, 2], index=['b', 'c', 'e'])
s3 = pd.Series([5, 6], index=['f', 'g'])
```

자 시리즈를 연결하려면 무엇을 생각해야하는가? 크게 두가지 측면에서 바라보면 좋다.

1. 시리즈를 위, 아래로 이어 붙인다. -> 그대로 `series`  임.
2. 시리즈를 옆으로 이어 붙인다. ->  `series`가 하나의 `column`으로 변함. -> `DataFrame`의 자료구조가 됨

그래서 `axis`를 생각하면서 작업해야한다.

```python
s4 = pd.concat([s1, s2, s3], axis=0)  # 행방향으로 시리즈를 붙여보자.
print(s4)
"""
a    0
c    1
b    4
c    3
e    2
f    5
g    6
dtype: int64
"""

result = pd.concat([s1, s2, s3], axis=1, sort=True)   # axis=1 열방향
display(result)
"""
   0   1   2
a 0.0 NaN NaN
b NaN 4.0 NaN
c 1.0 3.0 NaN
e NaN 2.0 NaN
f NaN NaN 5.0
g NaN NaN 6.0
"""
```

여기서 눈여겨 봐야할 점은 중복된 인덱스는 어떻게 처리가 되었느냐이다.
`c` 같은 경우엔 `s1`에도 있고, `s2`에도 있었다. 그래서 값이 두개로 떨어졌음.

첫번째 시리즈가 첫번째 컬럼이 됨

두번째 시리즈가 두번째 컬럼이 됨

세번째 시리즈는 당연히 세번째 컬럼이 되었다.

---

### DataFrame 연결해보기

`DataFrame`또한 `Series`와 연결하는 방식이 비슷하다.

```python
import numpy as np
import pandas as pd

np.random.seed(1)

df1 = pd.DataFrame(np.random.randint(0, 9, (3,2)),
                   index=['a', 'b', 'c'],
                   columns=['one', 'two']) # 3x2 matrix

display(df1)
"""
  one  two
a  5    8
b  5    0
c  0    1
"""


df2 = pd.DataFrame(np.random.randint(0, 9, (2,2)),
                   index=['a', 'b'],
                   columns=['three', 'four']) # 3x2 matrix

display(df2)
"""
  three  four
a   7     6
b   2     4
"""

# 이렇게 각기 다른 두개의 데이터프레임을 연결해보자.

result = pd.concat([df1, df2],   # 두개나 여러개를 줘야하니깐 리스트로 표현
                   axis=1)  # axis=1 열방향 옆으로 붙이는거

display(result)
"""
   one  two  three  four
a   5    8    7.0   6.0
b   5    0    2.0   4.0
c   0    1    NaN   NaN
"""

result = pd.concat([df1, df2], axis=0)  # 행방향으로 붙여보았음
display(result)
"""
	one	two	three	four
a	5.0	8.0	NaN	NaN
b	5.0	0.0	NaN	NaN
c	0.0	1.0	NaN	NaN
a	NaN	NaN	7.0	6.0
b	NaN	NaN	2.0	4.0
"""

# 밑으로 행으로 붙이는 것이기에 없는 데이터에 한해서는 결치로 값이 나타남
```

