---
title: python Pandas (12) DataFrame Duplicates 중복값 제거
date: 2021-08-24 17:27:00 +0900
categories: [python, pandas]
tags: [python, numpy, pandas, drop_duplicates, duplicates] 
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

## 중복 값 handling

데이터를 다루면서 평균을 구하다보면 집계함수를 칼럼에 넣기 위해 중복된 값을 지워야하는 순간이 있다.

그때를 위한 함수가 몇가지 있다. 

바로 예제를 보자.

```python
import numpy as np
import pandas as pd

df = pd.DataFrame({
    'k1': ['one'] * 3 + ['two'] * 4,  # 파이썬의 리스트에 더하기 의미는 리스트를 붙이란 소리임
    'k2': [1, 1, 2, 3, 3, 4, 4]
}) 

display(df)
"""
   k1   k2
0  one  1
1  one  1
2  one  2
3  two  3
4  two  3
5  two  4
6  two  4
"""
```

보다시피 중복된 행이 많이 등장한다.

이럴 경우 어떻게 처리할까?

또 언제 중복된 행을 배제해야하는가?

우선 중복여부를 확인하기 위해 `.duplicated()` 메서드를 호출하자.

```python
display(df.duplicated())
"""
0    False
1     True
2    False
3    False
4     True
5    False
6     True
dtype: bool
"""
```



***

`.duplicated()`의 로직

​       k1    k2
0    one    1        중복됏나요?  ==>  아니요 처음이에요
1    one    1        중복됐나요?  ==> 네 **중복됐어요**
2    one    2        중복됐나요?  ==> 아니오
3    two    3        중복됐나요?  ==> 아니오
4    two    3        중복됐나요?  ==> 네 **중복됐어요**
5    two    4        중복됐나요?  ==> 아니오
6    two    4        중복됐나요?  ==> 네 **중복됐어요**

***

보다시피 `k1`과 `k2`의 값이 모두 같은 값으로 두번 이상 나오는 경우 중복된 부분에 있어 `bool`값으로 반환해서 알려준다

어떻게 활용할까? `boolean mask`로 활용하면 좋을거 같다.

```python
# 중복된 행만 추출
df.loc[df.duplicated(), :]   # 중복된 친구만 뽑았음.

"""
  k1   k2
1 one  1
4 two  3
6 two  4
"""
```



### 중복행 삭제

그렇다면 중복된 행은 어떻게 삭제할까? 

물론 `boolean mask`를 통해 중복되지 않은 행만 추출하여 새롭게 `df`을 만들어줘도 무방하다. 하지만 그것보다 더 간편한, 따로 `boolean mask`를 사용하지 않아도 되는 함수를 제공해준다. `.drop_duplicates()`

```python
df2 = df.drop_duplicates()
display(df2)
"""
  k1   k2
0 one   1
2 one   2
3 two   3
5 two   4
"""
```

이렇게 간단하게 나타낼 수 있다.



음.. 이번엔 조금 문제를 복잡하게 만들어보자.

새로운 `column` `k3`를 추가해보자.

```python
df['k3'] = range(7)   # k3의 값은 0부터 6까지임
display(df)
"""
   k1   k2   k3
0  one  1    0
1  one  1    1
2  one  2    2
3  two  3    3
4  two  3    4
5  two  4    5
6  two  4    6
"""
display(df.drop_duplicates()) 
# 전체적으로 중복된 행이 없음
# (중복의 기준 한 행의 k1, k2, k3에 대한 값이 그대로 동일하게 다른 행에도 있어야함)
# 당연히 드랍시켜도 원본 그대로임

```

하지만 많은 경우에서 우리는 하나의 `column`에 중복된 값을 제거하길 원한다. 

그럴땐 어떻게 하는가?

```python
# 컬럼 하나에 대해서 중복항을 제거하라! 라는 명령어를 알아보자
display(df.drop_duplicates(['k1'])) # 이렇게 인자에 [컬럼명]을 사용해주면 됨.
"""
   k1  k2  k3
0 one  1   0
3 two  3   3
"""
# 이렇게 k1의 값이 중복된 행을 제거해서 나타낸다.

# 이번엔 k1과 k2를 같이 써보자
display(df.drop_duplicates(['k1', 'k2']))  # k1과 k2를 고려했을 때 중복되는 것 제거
"""
   k1   k2  k3
0 one   1   0
2 one   2   2
3 two   3   3
5 two   4   5
"""
# 이렇게 k1과 k2의 값이 동일한 행만 제거한 뒤 dataframe을 보여준다.
```

이 외에도 `unique()`와 같은 함수를 통해 직, 간접적으로 중복값을 제어할 수 있는 방법은 있다.

일단은 여기까지.

끄읏. 👋 
