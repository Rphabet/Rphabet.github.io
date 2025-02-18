---
title: python Pandas (4) 데이터프레임 조작하기
date: 2021-08-21 18:19:00 +0900
categories: [python, pandas]
tags: [ML, python, numpy, pandas, 머신러닝] 
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

## 🐼  `DataFrame` 인덱스와 행(record, row, observation) 제어

`column`에 집중했던 지난 시간에 이어 이번엔 행을 제어해보자.

`column`은 상대적으로 제어하기가 용이했다. 왜냐면 `column` 하나하나가 `pd.Series`로써 각 컬럼당 하나의 데이터 타입만 갖고 있었기 때문이다.

하지만 행 (row)는 다르다. 행은 여러개의 데이터 타입이 한곳에 뭉쳐있기 때문이다. 

예를 한번 들어보자.

1번행은 이름(문자열), 성별(문자열), 나이(정수), 백신 접종 여부(boolean), 사용하는 컴퓨터 (문자열) 등등 **"여러가지"** `data type`을 갖고 있다.

우리가 하나의 `column`이나 `array` 또는 `list`의 값을 가져왔던 것을 생각해보면 모두 **동일한!!!!!!!** `data type`이었음을 기억하자.

그렇기 때문에 더 어렵고, 제어하는게 생각보다 골치아프다. 

물론 `pandas`에서 삶을 편하게 해주는 메서드를 제공한다. 이를 사용할 때 어떤 메커니즘으로 이것이 수행 가능한지 인지를 하고 있다면, 더 깊은 공부에 큰 도움이 되리라 생각한다.

두서가 길었다. 

바로 예제를 살펴보자.

```python
# 지난번과 같은 데이터와 library를 불러주자.
import numpy as np
import pandas as pd

data = {'이름': ['아이유', '김연아', '홍길동', '강감찬', '이순신'],
        '학과': ['수학과', '기계과', '철학과', '경영학과', '철학과'],
        '학년': [1, 2, 3, 4, 2],
        '평점': [1.5, 3.8, 2.9, 4.3, 4.4]}

df = pd.DataFrame(data,
                 columns=['학과', '이름', '학년', '평점', '등급'],    
                 index=['one', 'two', 'three', 'four', 'five']) 

#### 바로 row indexing & slicing 에 들어가기 보단, column에 대해 하나만 짚고 넘어가겠다.
# df['이름':'평점']  이렇게 column에 대한 슬라이싱은 안된다. 
# 에러가 출력됨을 인지하자 (뭐가 이렇게 복잡한지..)
# 그런데 희안하게도 column에 대해 fancy indexing은 된다.
# e.g.  df[['이름', '학년', '평점']]   
# 이렇기 때문에 참.. 골치 아프면서도 어렵다(?).

##### 반면에 row indexing은 어떨까?
df[1]   # 이거 에러다. 진짜 욕나온다. 뭐 어쩌라는거지?
        # 에러가 나오는 이유는 pandas series로 값을 내보내는데
        # 컬럼마다 data type이 다르기 때문이다.
        # 때문에 하나의 행안에 여러가지 data type이 있다.

# 그런데... 여기서 삶을 힘들게 만드는건.. row slicing은 허용된다는 점이다.
df['one':'three']  # ㅅㅂ 머야 지정인덱스 슬라이싱은 됨.
"""
        학과   이름  학년   평점   등급
one    수학과  아이유   1  1.5  NaN
two    기계과  김연아   2  3.8  NaN
three  철학과  홍길동   3  2.9  NaN
"""
df[1:3]  # 숫자인덱스 슬라이싱 또한 가능하다
"""
학과   이름  학년   평점   등급
two    기계과  김연아   2  3.8  NaN
three  철학과  홍길동   3  2.9  NaN
"""
## 근데 진짜 복잡하게 만드는 사실은.. fancy indexing이 안된다.. 
df[[1, 3]]  # 이거 에러임... 

```

자 왜그럴까?  🤬 

욕나오는 이 상황은 뭘까?

어떤건 슬라이싱이 되고, 반면에 `fancy indexing`은 안되고.. 일관성 없게 느껴질 수도 있다.

되는 이유를 설명해주겠다.

* 슬라이싱은 원본에 대한 `View`이다.  원본(`df`)의 데이터 타입이 무엇인가? `DataFrame` 아니던가?  `DataFrame`은 여러개의 `columns`으로 구성되어 있다. 그리고 각 `column` 들은 서로 다른 `data type` 일 수 있다.  그렇기 때문에, 슬라이싱을 한다면 `DataFrame`으로 값이 리턴이 된다. 그래서 허용이 되는것이다.

그렇다치더라도 `fancy indexing`이 안되는건 여전히 납득이 되지 않는다. 이것 또한 `dataframe`으로 나오는 것 아니던가? 

알다가도 모르는게 pandas 자료구조인거 같다.

지금 당장은 저 자체로 받아들이고 잠깐 넘어가도록 하자.

**자 그럼 row에 대해서 단일 인덱싱은 어떻게 하느냐?** 이게 오늘 포스팅에서 다룰 주제이다.

pandas의 두가지 메서드를 사용하는데 `.loc` 함수와 `.iloc` 함수이다.

사용법을 알아보자.

```python
# 데이터는 위와 동일.
df.loc['one'] # one 이라는 지정 인덱스를 가진 행을 호출
# 아웃풋:
학과    수학과
이름    아이유
학년      1
평점    1.5
등급    NaN
Name: one, dtype: object
```

눈썰미 좋은 사람은 이미 발견했겠지만, `pandas series`로 결과물이 나왔고, `data type`은 객체(`object`)로 떨어졌다.

와우..  😦 

하나 동일한 `data type`만 출력할 수 있는 `Series` 특성상 문자열과 숫자가 섞여 있는 행 데이터를 불러오는게 불가능했다. 그래서 객체로 떨궈주는거다. 

`.loc` 의 큰 특징중 하나는 **"지정 인덱스"**만 사용해야한다는 것이다. 

지난번 포스팅에서 내가 지정인덱스를 왜 써야하는가, 또 왜 배워야하는 것인가에 대해서 간단하게 "사용할 구석이 많다"정도로만 얘기하였는데, 그중 하나가 지금처럼 `.loc` 함수를 이용하기 위함이라고 해두자.

그러면 `slicing`은 되나요? 물론이다.

```python
df.loc['one':'five'] 
# df['one':'four']
# loc 없어도 됐잖아. 그러니깐 되지
```

어차피 `df`에 대해서 `slicing`을 한다면 `data frame`으로 출력되었기 때문에 `loc`함수 없어도 가능했다.

신세계는 따로있다.

```python
df.loc[['two', 'four']]  

   	  학과	   이름 	학년	평점	 등급
two 	기계과	  김연아 	2	  3.8 	NaN
four	경영학과	강감찬	  4	  4.3	 NaN
```

이렇게 불가능했던 `fancy indexing`도 가능하다는 점이다.

자 또 색다른 활용 방법을 계속해서 찾아보자.

```python
df.loc['two':'four', '학과']   # 이렇게 행과 열에 모두 컨디션을 주는 것도 가능하다.

	      학과	
two	   기계과	
three	 철학과	
four	경영학과	


# 위와 같은 방법인데 
df.loc['two':'four']['학과']
# 이렇게도 가능함.
# 해석하자면 
# "df아 loc로 two라는 인덱스를 가진 행부터 four라는 인덱스를 가진 행까지 필터링해줘!
# 그리고 열에 대해서 '학과'라는 열로 단일 인덱스를 해줘."
# 라고 이해하면 된다.

# 조금 더 깊게 생각해보자
# 그럼 행에 대해서 슬라이싱을 하고, 열에 대해서 fancy indexing은 가능한가?
df.loc['two':'four', ['학과', '학년']]  
# 결과물: 
	      학과	학년
two	   기계과	2
three	 철학과	3
four	경영학과	4

# 가능하다.
# 마찬가지로 행을 슬라이싱하고 열을 슬라이싱 하는것도 가능하다.
```

`boolean indexing`은 어떨까 그럼?

문제를 통해 알아보자.

**Q. 학점이 4.0을 초과하는 학생의 이름과 학점을 dataframe으로 출력하세요**

```python
import numpy as np
import pandas as pd

data = {'이름': ['아이유', '김연아', '홍길동', '강감찬', '이순신'],
        '학과': ['수학과', '기계과', '철학과', '경영학과', '철학과'],
        '학년': [1, 2, 3, 4, 2],
        '평점': [1.5, 3.8, 2.9, 4.3, 4.4]}

df = pd.DataFrame(data,
                 columns=['학과', '이름', '학년', '평점', '등급'],    
                 index=['one', 'two', 'three', 'four', 'five']) 

display(df)

display(df[ df['평점'] > 4.0 ].loc[:,['이름','평점']])
display(df.loc[df['평점'] > 4.0, ['이름', '평점']])   


       이름	평점
four	강감찬	4.3
five	이순신	4.4

# 둘 다 가능. 더 빨리 구할 수 있는거면 더 좋음
# 중요한건 결괏값과 속도
# 똑같은 결과라면 속도가 빠른게 장땡 
# 데이터의 세계는 그렇다.
```

**Q. 이름이 아이유인 사람을 찾아서 이름과 학과를 Data Frame 으로 출력하자.**

```python
display(df[df['이름'] == '아이유'].loc[:,['이름','학과']])  # 1번
display(df.loc[ df['이름'] == '아이유', ['이름', '학과']])  # 2번

   	이름	학과
one	아이유	수학과
```

**Q. 평점이 3.0을 초과하는 사람을 찾아서 등급을 'A' 로 설정하세요.**

```python
df.loc[ (df['평점'] > 3.0), '등급' ] = 'A'

   	   학과	이름	학년	평점	등급
one  	수학과	아이유	1	   1.5	NaN
two	  기계과	김연아	2	   3.8	A
three	철학과	홍길동	3	   2.9	NaN
four	경영학	강감찬	4	   4.3	A
five	철학과	이순신	2	    4.4	A
```

**Q. 학점 1.5이상 3.0미만인 사람을 찾아 학과 이름 평점을 출력하라.**

```python
import numpy as np
import pandas as pd
# Tips:
# boolean 을 pandas 에서 쓸때는 &, | 연산자를 써야한다.  and이나 or 쓰지 말길.
# 또한 & 과 | 컨디션이 들어간 이상 () 괄호로 확실하게 묶어서 표현해야 에러가 나지 않는다.

display(df[ (df['평점'] >= 1.5) & (df['평점']< 3.0) ].loc[:,['학과', '이름', '평점']]) # 1번
display(df.loc[ (df['평점'] >= 1.5) & (df['평점']< 3.0), ['학과', '이름', '평점'] ])  # 2번

# df.loc[    , ['fancy indexing']] 필자는 이 순서로 작업하는것을 선호함.
```

자 오늘은 여기까지. 

끄읏. 🙋‍♂️ 👋 