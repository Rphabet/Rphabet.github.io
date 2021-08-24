---
title: python Pandas (10) DataFrame 특정값 처리 (이상치, 결치)
date: 2021-08-24 15:52:00 +0900
categories: [python, pandas]
tags: [python, numpy, pandas, null, na] 
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

## DataFrame NaN 처리

`NaN`을 처리하기 용이하게끔 만들어주는 몇 가지 함수를 알아보자.

```python
import numpy as np
import pandas as pd

np.random.seed(1)  # 난수 사용을 위해 seed 설정
df = pd.DataFrame(np.random.randint(0, 10, (6, 4))) # 6x4 matrix 생성

df.columns = ['A', 'B', 'C', 'D'] # 컬럼명 설정
df.index = pd.date_range('20210801', periods=6) # 인덱스를 날짜로 주자.
display(df)
"""
            A B C D
2021-08-01  5	8	9 5
2021-08-02  0 0 1 7
2021-08-03  6 9 2 4
2021-08-04  5 2 4 2
2021-08-05  4 7 7 9
2021-08-06  1 7 0 6
"""

# 새로운 column을 추가해보자
df['E'] = [7, np.nan, 4, np.nan, 2, np.nan]  # 결치값이 들어가 있음.
display(df)
"""
            A B C D  E
2021-08-01  5	8	9 5  7
2021-08-02  0 0 1 7 NaN
2021-08-03  6 9 2 4  4
2021-08-04  5 2 4 2 NaN
2021-08-05  4 7 7 9  2
2021-08-06  1 7 0 6 NaN
"""
```

이렇게 결치값이 존재한다면 어떻게 해야하는가?

### 결치값 (NaN)을 가진 행 삭제

가장 간단한 방법은 결치값 (`NaN`)을 갖고 있는 행 삭제다.

물론 이게 경우에 따라선 좋은 방법일 수도 있지만, 보통은 그렇지 않다.

일단은 코드를 통해 알아보자!

```python
# NaN은 missing values로 표기.
# 가장 간단한 방법은 NaN 값을 행을 삭제
# 상대적으로 데이터양이 많고 NaN이 비율적으로 적을때
# 이게 절대적인 기준은 없음. empirically observed.
# df.dropna를 이용해서 지울 수 있음.
df2 = df.dropna(how='any', inplace=False) # how가 any면 "nan값이 하나라도 등장하면 지워라" 이소리임.
                                   # inplace=True 면 원본을 날림, False면 복사본
                                   # 복사본일 경우 새로운 df에 assign해줘야함.
display(df2)
"""
            A B C D  E
2021-08-01  5	8	9 5  7
2021-08-03  6 9 2 4  4
2021-08-05  4 7 7 9  2
"""
```

`df.dropna()` 첫번째 인자로 들어오는 `how=`에 관해서는 대표적으로 `all` 또는 `any`를 받을 수 있다.

- `all`일 경우: 

  해당 행에 대한 칼럼값들이 전부 `NaN`일 경우 해당 행을 삭제한다.

- `any`일 경우:

  해당 행에 대한 칼럼값 중 하나라도 `NaN`일 경우 해당 행을 삭제한다.

### 결치값 (NaN)을 다른 숫자로 대체

두 번째 방법은 결치값을 다른 숫자로 대체하는 것이다.

가장 이상적인 방법은 `Machine Learning`기법을 통해 결치값을 예측하여 값을 채워넣는것이다. 

또한, 데이터에 대한 해당 칼럼(변수)의 평균값이나 중위값 등등 이런 통계치도 상황에 따라 괜찮은 대안이 될 수 있다.

가장 기피해야할 방법중 하나는 임의의 수를 분석자가 직접 정해서 넣는 것인데, 이를테면 그냥 `NaN`을 0으로 대체하는 등 이런 논리적이지 못한 방법은 기피해야함을 인지하자!

하지만 여기서는 우선 0으로 대체를 해보겠다.

```python
df2 = df.fillna(value=0)
display(df2)

"""
            A B C D  E
2021-08-01  5	8	9 5  7
2021-08-02  0 0 1 7  0
2021-08-03  6 9 2 4  4
2021-08-04  5 2 4 2  0
2021-08-05  4 7 7 9  2
2021-08-06  1 7 0 6  0
"""
```

이렇게 결치값이 0으로 대체된 것을 볼 수 있다.

**Q. E column의 값이 NaN인것을 찾아 A, B column의 값을 출력하라.**

어떤식으로 로직을 짜야할까? 우선 `NaN`에 관해선 `==` 연산자를 사용할 수 없다.  

`isnull()`이라는 함수를 이용해서 `NaN`을 찾자!

```python
display(df.isnull())  # 전체 데이터 프레임에 대해서 boolean mask가 나옴.
"""
              A     B     C     D     E
2021-08-01  False False False False False
2021-08-02  False False False False True
2021-08-03  False False False False False
2021-08-04  False False False False True
2021-08-05  False False False False False
2021-08-06  False False False False True
"""

# 이번엔 'E'칼럼에서만 NaN을 찾아보자
display(df.isnull()['E'])   # series로 리턴
display(df['E'].isnull())  # 위와 똑같은 값을 리턴 
"""
2021-08-01   False
2021-08-02    True
2021-08-03   False
2021-08-04    True
2021-08-05   False
2021-08-06    True
Freq: D, Name: E, dtype: bool
"""

# 인덱싱으로 찾아보자.
display(df.loc[ df.isnull()['E'], ['A', 'B'] ])  
"""
            A B
2021-08-02  0 0
2021-08-04  5 2
2021-08-06  1 7
"""
# 이렇게 NaN 값이 있는 행에 대해서 A, B 컬럼만 출력하였다.
```



## 이상치 Handling

이번엔 `NaN` 말고 이상치를 바꿔보자.

학생들의 성적이 `s`라는 `pandas Series`에 기록되어 있다고 가정을 해보자.

```python
s = pd.Series([97, 50, 78, 89, 45, -20, 80])   # 성적이라고 하자.
# 영어성적이라고 간주를 한다면, 이상한 값이 하나 들어가있다.
# 성적이 어떻게 0점 밑으로 떨어질 수 있겠는가???
# 이럴 경우, 이상치를 가만히 놔두면 집계함수등 기본적인 통계치에 영향을 미치기 때문에 
# 대체를 해주던 삭제를 해줘야한다.
new_s = s.replace(-20, 100)  # replace로 값을 바꿔주자.
print(new_s)
"""
0    97
1    50
2    78
3    89
4    45
5    100
6    80
dtype: int64
"""
```

 이런식으로 `replace` 메서드를 사용하여 다룰수 있다. 

거듭 말하지만 어떤 값으로 바꿔주느냐는 처한 상황에 맞는 `logic`에 근거하여 유추하여야한다.



포스팅 끗. 👋 