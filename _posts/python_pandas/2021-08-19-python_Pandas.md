---
title: python Pandas (1): 데이터프레임으로 데이터 가져오기
date: 2021-08-20 14:00:00 +0900
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

# pandas  🐼 

이 글을 읽고 있는 당신이 데이터와 연관된 직업에 종사하길 원한다면 그 어떤 직업군보다도 `pandas`와 친해야하며, 능숙해야하며, 그리고 이것이 세상과 소통하는 하나의 매개체가 되어야한다.

파이썬의 라이브러리중에 가장 완성도 높고 훌륭한 라이브러리로 매년 꼽혀오는 `pandas` 이다.

우선 `NumPy`를 모른다면 `pandas`를 배우기전에 `numpy` 부터 공부하고 오자. [NumPy in my blog](https://rphabet.github.io/posts/python_NumPy_1/)



## pandas 너 뭐하는 친구니?? 👯‍♂️ 

`pandas`는 파이썬에 국한되어있지 않고, 전 스크립트 언어를 통틀어서 데이터 분석에 최고봉중 하나로 꼽힌다.

데이터 불러오고, 파일로 가져오고, 네트워크에서 땡기고 등등 이런 모든 것들을 효과적으로 `pandas`에서 처리할 수 있다. 

이런 효과적인 분석 및 데이터 핸들링 배경에는 `Data Frame` 이라는 `R` 기반 자료구조가 있는데, 내가 다루는 `pandas`에 대한 포스팅은 어찌됐든 이 친구에 대해 지지고 볶고 해볼 예정이다. 

자 우선 `pandas`에 관하여 두가지만 간단하게 알고 넘어가자.

`pandas`는 고유하게 정의된 **두 개**의 자료구조를 갖고 있다. 

1. `pandas Series` : 1차원 (즉 벡터)의 배열을 사용. `numpy`를 확장시킨 개념. 같은 `data type`만 시리즈내 요소로 들어올 수 있음.
2. `pandas DataFrame`: 2차원 (즉 행렬)을 사용. `Series` 여러개를 모아서 `Table` 로 만들었다고 생각하면 됨. 각 Series 마다 data type이 다를 수 있음.

자 바로 한 번 코드를 살펴보자.

## pandas Series

```python
import numpy as np   # numpy는 pandas가 쓰인다면 항상 같이 import 해주자.
import pandas as pd   # alias 는 항시 pd 로 잡을것 (글로벌 공통임)

s = pd.Series([-1, 5, 10, 99], dtype=np.float64)  # pandas Series
# dtype 은 data type임. np.float64 는 실수로 데이터 타입을 잡겠다! 이거임
# 출력물: 
# 0    -1.0
# 1     5.0
# 2    10.0
# 3    99.0
# dtype: float64

"""
엥 저 뜬금없이 왼쪽열에 찍혀있는 0 1 2 3 숫자는 뭔가요?
--> index임. Series에선 자동으로 잡힌다.
그래서 python의 dictionary 처럼 따로 출력이 가능함.
"""

print(s.values)  # 출력물 : [-1.  5. 10. 99.]  
# 뒤에 . 소수점이 붙은 이유는 실수라는 것을 표기하기 위함임.

print(s.index) # 출력물 : RangeIndex(start=0, stop=4, step=1)   0으로 시작해서 1씩 증가.
# 숫자로 나오진 않네요?
# <class 'pandas.core.indexes.range.RangeIndex'> 
# 판다스의 클래스중 하나이다. 일단은 이렇게 이해하고 넘어가자
```

이게 `Series` 이다. 사실 `Series` 는 정말 잘이해해야하는 개념중 하나이긴 하지만, `NumPy`를 배워서 그런진 몰라도 아직까지 문제는 크게 없어보인다.

음.. 뭐하는지 당장은 와닿지 않을 수도 있겠는데 `Series`는 저 숫자 index를 문자열로 `지정 index `로 치환을 시켜줄 수 있다.

```python
s = pd.Series([-1, 5, 10, 99], dtype=np.float64, index=['b', 'k', 't', 'c']) 
# 인덱스 문자는 임의로 아무거나 넣었음
# 출력물:
# b    -1
# k     8
# t    10
# c    99
# dtype: int32

print("s[1]의 값은 : {}".format(s[1]))  
# 출력물: s[1]의 값은 : 8
# 위에서 Index값을 다르게 주었지만, 본질적인 숫자 인덱스가 바뀌진 않음 그래서 호출가능

print("s[k]의 값은 : {}".format(s['k']))  
# 출력물: s[k]의 값은 : 8
# 이렇게 문자로 인덱스값을 주었던걸로 indexing을 할수도 있음.
```

미친짓이지만 그렇다면 같은 문자열로 지정 index를 주면 어떻게 될까?

```python
s = pd.Series([-1, 8, 10, 99], dtype=np.int32, index=['b', 'k', 'b', 'c'])
print(s)
# b    -1
# k     8
# b    10
# c    99
# dtype: int32

# 와 인덱스가 'b' 'b' 해서 나옴... 
# 그럼 실제 인덱싱을 해보자

print("s[b]의 값은: \n{}".format( s['b']))  
# series로 결과를 토해냄. 값이 두개야. 데이터 타입도 나오고
# 출력물: 
# s[b]의 값은: 
# b    -1
# b    10
# dtype: int32
```

이렇게 지정 인덱스가 `b`인 요소를 출력해낸다.

### pandas Series 슬라이싱 & 인덱싱

아래의 예제를 보자.

```python
# series에 대해서 slicing을 해보자.
# numpy랑 다름.. 그렇기때문에 언급을 하는거다.

s = pd.Series([-1, 8, 10, 99], dtype=np.int32, index=['b', 'k', 't', 'c'])
print(s[0:2])   # series를 slicing했으니깐 series가 나옴. 
                # slicing을 했을 때 결괏값은 늘 원본의 형태를 따라가거든.
                # 이 점은 Numpy와 똑같음. ndarray와 slicing
# 출력물:
# b   -1
# k    8
# dtype: int32
  
  
# 이번엔 우리가 갖고 있는 index를 통해서 slicing 을 해보자
print(s['b':'t'])  # 문자 index를 이용하면 upper limit이 inclusive임 
# 출력물:
# b    -1
# k     8
# t    10
# dtype: int32

```

그렇다면 `numpy`에서 해보았던 `boolean indexing`과 `fancy indexing` 도 먹힐까?

```python
s = pd.Series([-1, 8, 10, 99], dtype=np.int32, index=['b', 'k', 't', 'c'])

print(s[[0,2]])   # fancy indexing
# 출력물:
# b  -1
# t  10
# dtype= int32

print(s[ s > 8] )   # boolean indexing
# 출력물:
# t    10
# c    99
# dtype: int32

# 특이점(?)을 꼽자면 fancy와 boolean 둘다 series로 나옴.

# pandas 에서도 numpy에서 쓰던 집계함수를 그대로 쓸 수 있다.
print(s.sum())  # 출력물: 116

```

### pandas Series 를 만드는 또 다른 방법

`python` 의 `dict`를 이용해서 `pd.Series`를 만들어보자!

```python
my_dict = {'서울': 1000, '부산': 2000, '제주': 3000}
s = pd.Series(my_dict)
print(s)
# 출력물: 
# 서울    1000
# 부산    2000
# 제주    3000
# dtype: int64


# 이건 뭐 크게 중요한건 아닌데 Series에 이름을 붙일 수 있음.
# series 자체에 이름을 부여하는것임.
s.name = '지역별 가격'  # s안에 이름이라는 속성을 호출하고 값을 준거임.
print(s)
# 서울    1000
# 부산    2000
# 제주    3000
# Name: 지역별 가격, dtype: int64


# 이번엔 시리즈 인덱스에 이름을 하나 붙여보자.
s.index.name = 'region'
print(s)
# region             <<<<<<<<<<< 이렇게 region 이라는 인덱스 이름이 붙었다.
# 서울    1000
# 부산    2000
# 제주    3000
# Name: 지역별 가격, dtype: int64

s.index = ['Seoul', 'Pusan', 'Jeju']  # 이렇게 index값을 바꿀 수 있음.
print(s)
# Seoul    1000
# Pusan    2000
# Jeju     3000
# Name: 지역별 가격, dtype: int64
```

"**그래서 이 `data Series` 로 뭐 하는건데요???**" 

=> 아까 잠깐 언급했지만, 이 친구로 이루어진 자료구조가 `pandas DataFrame`이다. 

## pandas DataFrame

데이터프레임에 대한 이해의 중요도는 두말 할 필요가 없다. 어떤 분석을 하던, 데이터를 다루던 파이썬을 사용한다면 `pandas`를 통해 데이터를 핸들링해야한다. 잘못된 핸들링은 잘못된 결괏값을 도출한다는 사실을 절대 잊지 말자.

우선 `python`의 `dict`를 하나 만들어보자.

```python
import numpy as np
import pandas as pd

# 각 키값과 밸류값은 하나의 시리즈임.
data = {'이름': ['아이유', '김연아', '홍길동', '장범준'],   # 이름이라는 시리즈
        '학과': ['Econ', 'Math', 'Physics', 'Music'], # 학과라는 시리즈가
        'age': [23, 21, 22, np.nan]}    # np.nan = NaN   not a number

# 이 dict를 dataframe으로 바꿔보자

df = pd.DataFrame(data)   # 이렇게 pd.DataFrame() 을 사용하면 데이터프레임으로 변환이 가능하다.
display(df)    # print문보다 dataframe에 한정해서 display()를 사용한다.
               # 출력물은 아래에 screen shot 으로 첨부해두었다.

# 전반적인 df의 모양에 대해 알아볼 때:
print(df.shape)  # (4, 3)   4x3 행렬이라고 생각하면 됨
print(df.size)  # 12   총 12개의 요소값이 있음.
print(df.ndim)  # 2차원   몇차원인지 알려줌   (df는 어차피 2차원. 쓸일 많이 없음)

# df의 칼럼에 대해 알아볼 때:
print(df.columns) # Index(['이름', '학과', 'age'], dtype='object')
print(df.index) # RangeIndex(start=0, stop=4, step=1)
print(df.values) 
# 출력물:
# [['아이유' 'Econ' 23.0]
#  ['김연아' 'Math' 21.0]
#  ['홍길동' 'Physics' 22.0]
#  ['장범준' 'Music' nan]]

# 음? 어떻게 nd.array안에 각기 다른 데이터 타입이 들어올 수 있지?
# 한번 dtype을 찍어보자.
print(df.values.dtype) # object
# 아하. 객체로 받아들였구나.
# 일단은 이렇게만 이해하자.
```

<img src="/../assets/images/pandas/df.png" alt="df" style="zoom:50%;" />

"**Q. 근데 지금처럼 `DataFrame`을 늘 손으로 작성하진 않을거잖아요. 왜 저렇게 비효율적으로 할까요?**"

--> 어떻게 만들어지는지, `dict`로 만들었는지, 나중에 `json`을 긁어온다면 어떻게 data frame 만들지 알 수 있기 때문이다.

### csv파일을 읽어보자

```python
import numpy as np
import pandas as pd

# pandas 의 csv 읽는 기능인 read_csv 메서드를 쓴다.
df1 = pd.read_csv('./data/movie/movies.csv') # 경로설정만 하면 됨
df2 = pd.read_csv('./data/movie/ratings.csv')

display(df2.head())  # default로 상위 다섯개만 보여줌. 몇개를 보여줄지 조정하고 싶다면 인자로 값만 넣어주자
display(df2.tail())  # 하위 다섯개
```

음 오케이 이런식으로 읽는거구나?

그렇다면 저 불러들인 파일로 무얼 하나요? 당연히 분석을하지. 핸들링을 하고. 

가령... 

> 1980년대 영화중 장르가 로맨스인 영화중에 평균 평점이 가장 높은 영화는 무엇인가요? 

이런 질문에 대한 해답을 찾아야한다.  
어떻게 필터링을 해서 어떤 식으로 데이터를 가져와서 어떻게 할건지 등등... 이런 식의 고찰과 데이터를 다뤄야한다.
즉 한마디로 데이터 핸들링할 때 가장 첫 출발점은 여기서부터이며 항간에서는 `EDA`라고 잘 알려져있다. 
로맨스 장르를 어떻게 가져올까? 한번 생각해보길 바란다. 위와 관련한 내용은 다른 포스팅에서 본격적으로 다루겠다.

### Database에서 데이터를 긁어와보자

뭐 맨날 당신이 데이터를 다룬다면 가장 친해야져야할 xxx. 이런식으로 소개를 했는데... `database` 또한 마찬가지이다. 이름에서 벌써 느껴지지 않는가?

보통 순서를 나열해보자면..

​	`database` 제어 > 원하는 `dataframe` 생성 > 분석

이런 식으로 `database`에 접근해서 데이터를 긁어와 분석을 한다.

한번 내 개인 로컬 서버에 있는 데이터를 불러와보겠다. (책에 대한 데이터를 사용할것임) 

```python
import pymysql.cursors  # 필자는 mysql을 쓰기 때문에 위 모듈을 불러왔다.
import numpy as np
import pandas as pd

# 데이터베이스를 기동해야함! MySQL 서비스를 기동해보자

# 데이터베이스에 접속하자
# 접속에 성공하면 접속성공객체 (connection)을 하나 return 한다

conn = pymysql.connect(host='localhost',
                      user='python',    
                      password='python',
                      db='library',
                      charset='utf8')

print(conn) 
# <pymysql.connections.Connection object at 0x7fc45e63f190>
# 이렇게 에러가 뜨지 않고,
# 결국 객체가 출력 된다면 접속 성공이라고 볼 수 있다.
# 접속에 성공했으니 SQL을 이용해서 질의를 해보자
# 질의의 결과를 dataframe으로 만들어보자.

# SQL 명령어 문법을 문자열로 만들어놨음.
sql = 'SELECT bisbn, btitle, bauthor, bprice FROM book'
# "책번호, 책 이름, 저자, 값을 book이라는 db에서 선택해라!" 라는 sql 명령어임.

df = pd.read_sql(sql, con=conn)   
# 인자를 설명하자면
# sql 구문 실행하고, 데이터베이스 연결은 conn 이놈으로 해! 라는 소리임.
display(df)  
# database 에 있는 data 땡겨서 이렇게 출력이 된다.  
# 748 x 4 임. 2차원
# 아래 스크린샷을 보자.

```

<img src="/../assets/images/pandas/df_book.png" alt="df_book" style="zoom:50%;" />

이렇게 `data frame`이 출력이 된다.

이야기가 많이 길어졌다. 

다음 포스팅에선 `pandas`를 어떻게 `.json`으로 저장하는, `open api` 불러와보기 등과 같은 `dataframe` 로딩에 연관된 부분을 좀더 자세하게 다뤄보고자한다.

일단 끄읏. 👋 