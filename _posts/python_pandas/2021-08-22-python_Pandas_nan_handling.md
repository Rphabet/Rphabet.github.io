---
title: python Pandas (5) NaN 핸들링
date: 2021-08-22 20:48:00 +0900
categories: [python, pandas]
tags: [python, numpy, pandas, 프로그래밍, 데이터] 
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

## NaN 핸들링

여러곳에서 데이터를 다루다보면 `DataFrame`내에 결치값을 심심치않게 보게 될 것이다. 

온전한 데이터를 갖기란 생각보다 굉장히 힘들기 때문이다.

이번 포스팅은 NaN을 어떻게 핸들링 하는지 넌지시 얘기해보는 시간을 갖자.

```python
import numpy as np
import pandas as pd

data = [[2, np.nan],
        [7, -3],
        [np.nan, np.nan],
        [1, -2]]

df = pd.DataFrame(data,
                  columns=['one', 'two'],
                  index=['a', 'b', 'c', 'd'])
display(df)
# 출력물
"""
  one	two
a	2.0	NaN
b	7.0	-3.0
c	NaN	NaN
d	1.0	-2.0
"""
```

이렇게 중간중간에 결치가 있음에도 불구하고, 집계함수를 사용하면 어떻게 될까? 

한번 `sum()`을 이용해보자.

```python
df.sum()
# 결과물:
one    10.0
two    -5.0
dtype: float64
```

이렇게 `column` 별로 `NaN` 값을 고려하지 않은채로 계산이 되어 값이 `Series` 형태로 반환되었다.

또한 `axis` 값을 주지 않는다면, `axis=0` 행방향 계산으로 디폴트되어 연산이 수행되는 것을 알  수 있다.

그렇다면 `axis`값을 줘보자.

```python
df.sum(axis=1) # 열방향 계산
# 결과물:
a    2.0
b    4.0
c    0.0
d   -1.0
dtype: float64
```

 마찬가지로 값이 `Series` 형태로 열방향으로 `NaN`을 고려하지 않고 계산이 되어 리턴되었다.

`NaN` 이 고려되지 않는건 기본값으로 들어와있는 `skipna=True` 라는 인자때문인데, 언제나 결치값을 무시하면서 연산을 수행할수는 없는 노릇이다.

특히나 데이터가 방대하고 결치의 비율이 상대적으로 적다면 이런 연산 수행은 무리없이 할 수 있는 경우가 많으나, 반대로 결치 값이 많고 데이터는 상대적으로 적다면 아주 편향된 결과를 야기할 수 있다.

또한 `skipna=False` 로 계산을 하게 되면, `숫자 + NaN` 의 경우가 되는데, 당연히 연산이 불가능하며 `NaN`의 값이 리턴된다.

**그렇다면 이러한 missing values는 어떻게 하면 좋을까?**

상황에 따라 적용할 수 있는 몇가지 선택지가 있다.

1. 결치값을 채워준다
   - 보통 평균 값을 많이 이용함
   - 최대값, 최솟값도 하나의 방안임
2. 결치값을 갖고 있는 행 데이터를 날린다.

지우자니 이상하고, 값을 채워주자니 애매하고.. 이게 정답이 없으니 어렵게 느껴질 수도 있다.

중론은 결치값이 상대적으로 많다면, 삭제하는 것보다는 중간 값으로 채우는것을 선호하는 경향이 있다. 

(계량경제학에서 다루는 데이터중에 `CPS` 같은 데이터는 결치 값은 삭제를 하는 경향이 있다. -- 필자의 의견은 아니고 많은 논문에 데이터 섹션을 보았을때를 기준으로 느낀점을 말해보았음.)

중요한건 지향하는 것보다 "**지양**"하는 부분은 확실하게 있다. 

이를테면 아무 의미도 없는 임의의 값 (e.g. 0) 으로 결치값을 대체하는건 삼가길 바란다. 논리적이지 못할뿐더러 결괏값을 심하게 편향시킬 수 있기 때문이다.

아무튼 다시 코드로 돌아와서, 결치값을 평균값으로 채워보자.

```python
data = [[2, np.nan],
        [7, -3],
        [np.nan, np.nan],
        [1, -2]]

df = pd.DataFrame(data,
                  columns=['one', 'two'],
                  index=['a', 'b', 'c', 'd'])

# 컬럼별 평균값
one_avg = df['one'].mean()   # 3.333333
two_avg = df['two'].mean()   # -2.5

# fillna를 사용해서 결치값을 추가해주자.
# na값을 fill 하라 라는 소리임.
df['one'].fillna(one_avg, inplace=True)  # inplace는 원본 데이터를 바꿀까요? 라고 물어보는 것이라 생각하면 이해하기 편하다.
df['two'].fillna(two_avg, inplace=True)

display(df)
# 출력물:
      	one	two
a	2.000000	-2.5
b	7.000000	-3.0
c	3.333333	-2.5
d	1.000000	-2.0

# 이렇게 NaN이 컬럼별 평균값으로 대체된 것을 볼 수 있다.



#### 방법 2.
df['one'] = df['one'].fillna(one_avg, inplace=False)
df['two'] = df['two'].fillna(two_avg, inplace=False)

display(df)
# 이런식으로 inplace 값을 False로 줘서 다시 원본 데이터에 칼럼별 값으로 넣어주는 방법도 있다는 것만 알아두자.
# 똑같은 방법임
```



