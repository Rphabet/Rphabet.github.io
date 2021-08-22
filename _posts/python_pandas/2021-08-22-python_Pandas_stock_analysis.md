---
title: python Pandas (5) pandas_datareader 주식분석
date: 2021-08-22 19:45:00 +0900
categories: [python, pandas]
tags: [ML, python, numpy, pandas, 머신러닝, 주식] 
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

## pandas를 응용해보자!

이번 포스팅에선 가볍게 주식 데이터를 불러와서 공분산과 상관계수를 사용해보고 해석해보는 시간을 갖겠다.

오늘 사용할 모듈중 하나는 `pandas_datareader` 인데, 설치가 되어있지 않다면 `pip install pandas_datareader` 명령어를 통해 반드시 설치하자!

```python
import numpy as np
import pandas as pd
import pandas_datareader.data as pdr # data 라는 서브패키지를 사용할것임
from datetime import datetime 

# 객체화 시켜서 날짜를 하나 잡아주자
start = datetime(2018, 1, 1)   # 2018년 1월 1일부터
end = datetime(2018, 12, 31)  # 2018년 12월 31일까지

# kospi 지수를 dataframe으로 불러오자
df_kospi = pdr.DataReader('^KS11', 'yahoo', start, end)
# 삼성전자 주가를 dataframe으로 불러오자
df_se = pdr.DataReader('005930.KS', 'yahoo', start, end)

# 인자를 설명하자면
# 첫번째 인자는 '종목 고유 번호'
# 두번째 인자는 어디서 데이터를 가져올건지, 우리는 '야후 파이낸스'를 이용
# 세번째 인자는 시작일 
# 마지막으로 네번째 인자는 마감일임.

display(df_kospi)   # 어떻게 구성되었나 한번 살펴보자면...
```

<img src="/../assets/images/pandas/stock.png" alt="stock" style="zoom:50%;" />

2018년 1년 동안의 데이터를 불러올 수 있다.

간단하게 컬럼명에 대한 설명을 하자면:

- `High`: 일 최고가
- `Low`: 일 최저가
- `Open`: 시초가
- `Close`: 종가
- `Volume`: 거래량
- `Adj Close`: 수정종가

간단하게 KOSPI 지수와 삼성전자 주가의 상관관계를 알아보자면..

```python
series_close_kospi = df_kospi['Close']      # pandas Series로 리턴
series_close_se = df_se['Close']  # pandas Series로 리턴

# 시리즈를 한번 살펴보자면..
print(series_close_kospi)
"""
Date
2018-01-03    2486.350098
2018-01-04    2466.459961
2018-01-05    2497.520020
2018-01-08    2513.280029
2018-01-09    2510.229980
                 ...     
2018-12-21    2061.489990
2018-12-24    2055.010010
2018-12-26    2028.010010
2018-12-27    2028.439941
2018-12-28    2041.040039
Name: Close, Length: 242, dtype: float64
"""

np.cov(series_close_kospi.values, series_close_se.values)  
# series의 속성인 values를 이용해서 인덱스를 제외한 값으로만 비교하게 해줌.
# np.cov는 numpy 에서 제공하는 공분산 메서드임
# 결과물:
[[   24045.56247333   489067.36939603]
 [  489067.36939603 11918322.34148349]]
```

우리에게 필요한 건 대각선 diagonal 값이 아닌 off-diagonal 값임 (*489067.36939603*)

대각선의 값은 `variance` 를 나타냄. 한마디로 상관관계를 보여주는 값은 아니다 이거지.

공분산 계수가 *489067* 로 나왔는데, 중요한건 양수냐 음수냐만 알면 된다. 

- 양수일 시, 하나가 오르면 다른 하나도 오르는 경향이 있다는 걸 알려줌 (⚠️  인과관계는 아님!!!!)
- 음수일 시, 하나가 오르면 다른 하나는 내려가는 경향이 있다는 걸 알려줌 (⚠️  인과관계는 아님!!!!)

공분산의 크기는 시사하는 바가 크진 않음.. 아니 사실상 해석적인 측면에서는 아예 없다고 할 수 있다.

그렇기에 상관관계의 정도를 알고 싶다면 `correlation coefficient`을 사용해서 알아볼 수 있다.

***상관관계*** (***correlation***) : 두 대상이 서로 연관성이 있다고 추측되는 관계

예를 들자면.. 

- 그룹별 평균 교육 기간과 범죄율
- 아이스크림 판매량과 해수욕장 관광객 수

상관계수는 [-1, 1] 사이의 실수로 나오며
0과 가까워질수록 연관성이 낮다!
-1이나 1로 가까워질수록 연관성이 높아진다.  (참고로 이건 피어슨 상관계수임)

거듭 말하지만, 상관계수를 이용해서 상관관계를 분석할 때 조심해야하는 것중 하나가  인과관계는 상관계수로 설명할 수 없다. 

인과관계는 regression 을 기반으로 유추할 수 있는데 이건 나중에 심도있게 다뤄보도록 하겠다.

너무 고맙게도 `numpy`에서 상관관계를 알려주는 함수를 제공해준다.

```python
print(np.corrcoef(series_close_kospi.values, series_close_se.values))
결과물: 
[[1.         0.91357384]
 [0.91357384 1.        ]]
```

이렇게 간단하게 구할 수 있다.

공분산과 마찬가지로, off-diagonal에 위치한 값이 중요하다.

삼성전자와 코스피지수의 상관계수는 굉장히 높으며, 해석해보자면 "삼전주가가 올라갈수록 코스피 지수도 올라가는 경향을 보인다." 이정도로 해석할 수 있겠다.

지금은 우리가 `series`를 이용하여 상관관계의 값을 가져왔다.

**data frame**에서 바로 추출할 수 있진 않을까? 물론 가능하다

```python
# 다시 데이터 프레임을 하나 만들어주자.
# 컬럼 명은 KOSPI와 SE(삼성전자)가 되고
# 컬럼의 값은 코스피 지수의 값과 삼성전자 주식가격이 되겠다.
# 이때 인덱스는 날짜가 아닌 0 1 2 3 4 순으로 나열됨을 인지하자!

df = pd.DataFrame({
    'KOSPI': series_close_kospi.values,
    'SE': series_close_se.values
})

display(df)

display(df.corr())  # 상관계수 출력

        KOSPI    	SE
KOSPI	1.000000	0.913574
SE	  0.913574	1.000000
```



이번 포스팅에선 많은 것을 다루진 않았다.

주식 과거 데이터를 `pandas_datareader`를 통해 불러와서 공분산과 상관계수 메서드를 활용해보았다. 

다음번 `pandas` 응용 포스팅에선 데이터 시각화를 포함해서 데이터 분석에 깊이를 더해보겠다.

이상 오늘은 여기까지.

끄읏 👋 

