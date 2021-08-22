---
title: python Pandas (3) 데이터프레임 조작하기
date: 2021-08-21 17:15:00 +0900
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

## 🐼  `DataFrame` 인덱스와 칼럼명 제어

`pandas` 세 번째 포스팅이다.  지난번 포스팅과는 다르게  이번에는 간단한 데이터를 하나 만들어서 어떤식으로 `column`을 추가하고 삭제하고 변경하고 마찬가지로 `index`를 효과적으로 조작할 수 있는지 한번 알아보자.

```python
import numpy as np
import pandas as pd

data = {
  			'이름': ['아이유', '김연아', '홍길동', '강감찬'],
        '학과': ['수학과', '기계과', '철학과', '경영학과'],
        '학년': [1, 2, 3, 4],
        '평점': [1.5, 3.8, 2.5, 4.3]
				}

# 이걸 데이터 프레임으로 바꿔주면...

df = pd.DataFrame(data)
display(df)
# 출력물:
#    이름	 학과	 학년	평점
# 0	아이유	수학과	 1	 1.5
# 1	김연아	기계과	 2	 3.8
# 2	홍길동	철학과	 3	 2.9
# 3	강감찬	경영학과 4	 4.3

# 이렇게 나오게 된다.
# dictionary의 key값이 컬럼의 이름으로 기입되었다.

# df를 만드는 과정에서 column의 이름을 다르게 줄 수 있을까?
# 물론 가능하다 => 'columns=[]' 값을 주면 된다.

df = pd.DataFrame(data,
                 columns=['학과', '이름', '학년', '평점'])
# 이런식으로 설정할 수 있다.
```

음... "**난 근데 생각해보니깐 열을 하나 추가해야겠는데요??**" 그럴땐 `dict` 값을 바꾸지 않고 dataframe을 만들때 columns 값을 하나 추가해주면 된다!

```python
# 성별이라는 칼럼을 하나 추가해주자
df = pd.DataFrame(data,
                 columns=['학과', '이름', '학년', '평점', '성별'])
display(df)
#    이름	 학과	 학년	평점    성별 
# 0	아이유	수학과	 1	 1.5   NaN
# 1	김연아	기계과	 2	 3.8   NaN
# 2	홍길동	철학과	 3	 2.9   NaN
# 3	강감찬	경영학과 4	 4.3   NaN
```

**"뭐야 저 `NaN`은 뭔가요?"**  => `Not a Number`의 약어로 결치라는 소리다. 왜 저게 나오냐고?? 우리 저거 값을 안줬잖아. 

이번엔 옆에 숫자로 되어있는 인덱스를 바꿔보자.

```python
df = pd.DataFrame(data,
                 columns=['학과', '이름', '학년', '평점'],
                 index=['one', 'two', 'three', 'four'])  
# 마찬가지로 'index=[]'에 값을 넣어주면 된다! 
# 인덱스 값인 0, 1, 2, 3 은 사라지는건 아님. 
# 지정 인덱스인 'one', ..., 'four'가 생기는것임.
# 한번 확인해보자!

display(df)
#        학과	 이름	   학년	 평점
# one 	수학과	아이유	   1	 1.5
# two	  기계과	김연아	   2	 3.8
# three	철학과	홍길동	   3	 2.9
# four	경영학과 강감찬	4	   4.3
```

이렇게 '**숫자 인덱스**'가 '**지정 인덱스**'로 변환된 것을 볼 수 있다.

**"이게 근데 왜 필요해요?"** 쓸일 엄청 많지.. 인덱스를 갑자기 날짜로 변환한다거나 등등... 데이터 제어할 때 되게 많이 사용되니깐 꼭 알아두자.

자 그럼 `NumPy`때부터 주구장창 설명했던 `fancy indexing`, `boolean indexing` 을 한번 적용해보도록 하자.

---

## 🐼: DataFrame Indexing and Slicing

```python
# 위에와 같은 data를 사용하자.

import numpy as np
import pandas as pd

data = {'이름': ['아이유', '김연아', '홍길동', '강감찬', '이순신'],
        '학과': ['수학과', '기계과', '철학과', '경영학과', '철학과'],
        '학년': [1, 2, 3, 4, 2],
        '평점': [1.5, 3.8, 2.9, 4.3, 4.4]}

df = pd.DataFrame(data,
                 columns=['학과', '이름', '학년', '평점'],
                 index=['one', 'two', 'three', 'four', 'five']) 

# '이름'이라는 컬럼에 대해 인덱싱을 해보자.
print(df['이름'])
"""
one      아이유                  ****이렇게 Pandas Series로 리턴해줌
two      김연아
three    홍길동
four     강감찬
five     이순신
Name: 이름, dtype: object
"""

# 기억할진 모르겠지만 이렇게 인덱스만 지정해서 불러온다면, 원본 값을 보여주는 view의 형태로 
# 우리에게 보여준다.
# 그래서 만약에 어떤 변수 (s_name) 를 df['column name'] 으로 지정해주고 
# 그 값을 변경한다면, 원본 데이터인 df의 값도 변한다.
# 한번 살펴보자

s_name = df['이름']
s_name['one'] = '장범준'   # one 이라는 인덱스의 value를 장범준으로 바꿔주겠음. 
print(s_name)
"""
one      장범준         <<<<< 이렇게 값이 변한걸 볼 수 있다.
two      김연아
three    홍길동
four     강감찬
five     이순신
Name: 이름, dtype: object
"""

# 여기서 헷갈리는 부분은 df 의 값도 변했다는 점이다.
display(df)
"""
       학과     	이름    	학년    	평점
one	  수학과	    장범준	      1	     1.5            <<< 아이유에서 장범준으로 바뀌었음
two	  기계과	    김연아	      2	     3.8
three	철학과	    홍길동	      3	     2.9
four	경영학과	 강감찬	      4	     4.3
five	철학과	    이순신	      2     	4.4
"""

# 이렇게 원본 데이터도 아이유에서 장범준으로 바뀌었음을 볼 수 있다.

```

처음에 기초를 탄탄히 다져놓지 못한다면, 나중에 데이터 핸들링을 할 때 꽤 애를 먹을 수 있는 부분이 바로 이런점이다.

분명 새로운 데이터를 만들어서  (e.g. `new_series = df['column']`) 따로 이것저것 조작을 해봤는데, 어느샌가 원본 데이터의 값도 바뀌어 있을 수 있다. `view`의 개념에 늘 주의하자! ⚠️ 

---

### fancy indexing with respect to columns

```python
import numpy as np
import pandas as pd

data = {'이름': ['아이유', '김연아', '홍길동', '강감찬', '이순신'],
        '학과': ['수학과', '기계과', '철학과', '경영학과', '철학과'],
        '학년': [1, 2, 3, 4, 2],
        '평점': [1.5, 3.8, 2.9, 4.3, 4.4]}

df = pd.DataFrame(data,
                 columns=['학과', '이름', '학년', '평점'],
                 index=['one', 'two', 'three', 'four', 'five']) 

display(df[['이름', '평점']]) # 이름과 평점으로 fancy indexing
"""
	       이름	평점
one	    아이유	1.5
two	    김연아	3.8
three	  홍길동	2.9
four	  강감찬	4.3
five	  이순신	4.4
"""
# 이렇게 두개의 칼럼의 값들만 들고 올 수가 있다.
```

일반 인덱싱과 유독 다른점은 무엇인지 눈치 챈 사람이 있는가?

바로 `pandas Series`로 리턴이 되지 않고, `pandas DataFrame`으로 리턴이 되었다는 점이다. 

----

자 그러면 `data frame`에서 특정 `column`의 값을 수정하면 어떻게 될까?

```python
data = {'이름': ['아이유', '김연아', '홍길동', '강감찬', '이순신'],
        '학과': ['수학과', '기계과', '철학과', '경영학과', '철학과'],
        '학년': [1, 2, 3, 4, 2],
        '평점': [1.5, 3.8, 2.9, 4.3, 4.4]}

df = pd.DataFrame(data,
                 columns=['학과', '이름', '학년', '평점', '등급'],    
                  # 등급이라는 NaN 컬럼을 하나 더줬다
                 index=['one', 'two', 'three', 'four', 'five']) 
display(df)
"""
       학과     	이름    	학년    	평점     등급
one	  수학과	    아이유	      1	     1.5      NaN      
two	  기계과	    김연아	      2	     3.8      NaN
three	철학과	    홍길동	      3	     2.9      NaN
four	경영학과	 강감찬	      4	     4.3      NaN
five	철학과	    이순신	      2     	4.4     NaN
"""

# 만약 내가 df['등급'] 에 'A'를 assign 하면 어떻게 될까? 
# 에러가 뜰까?
# 아니다 broadcasting 되어 전체에 적용된다.

df['등급'] = 'A'
display(df)

"""
       학과     	이름    	학년    	평점     등급
one	  수학과	    아이유	      1	     1.5       A      
two	  기계과	    김연아	      2	     3.8       A
three	철학과	    홍길동	      3	     2.9       A
four	경영학과	 강감찬	      4	     4.3      A
five	철학과	    이순신	      2     	4.4      A
"""
# 이렇게 모두 A등급으로 바뀐것을 볼 수가 있다.

# 만약 각기 다른값을 주고 싶다면 list 또는 np.array로 준다면 쉽게 해결된다.
df['등급'] = np.array(['A','B','C','E','B'])   
```

Q. 새로운 column을 dataframe에 추가하려면 또 어떻게 해야하나요?

아까처럼 `broacasting`을 이용해서 만들어줘도 되긴하나 더 효율적인 방법은 존재한다.

```python
#    나이를 추가해보자
data = {'이름': ['아이유', '김연아', '홍길동', '강감찬', '이순신'],
        '학과': ['수학과', '기계과', '철학과', '경영학과', '철학과'],
        '학년': [1, 2, 3, 4, 2],
        '평점': [1.5, 3.8, 2.9, 4.3, 4.4]}

df = pd.DataFrame(data,
                 columns=['학과', '이름', '학년', '평점', '등급'],    
                 index=['one', 'two', 'three', 'four', 'five']) 
# -----------1번 방법------------------
df['나이'] = [20, 25, 21, 24, 29]   # 모든 레코드의 값을 채워줘야 error 가 나지 않음.
                                   # 만약 네개의 값만 채웠다면 error임
display(df)

"""
	      학과	  이름  	학년	평점	 등급	  나이
one	   수학과	 아이유	  1	  1.5  	NaN 	20
two	   기계과	 김연아	  2	  3.8	  NaN 	25
three	 철학과	 홍길동	  3 	2.9	  NaN 	21
four	 경영학과	강감찬	   4	  4.3	  NaN 	24
five	 철학과	 이순신	  2	  4.4	  NaN 	29
"""
# 방법 자체는 꽤 간단해보인다. 
# 두번째 방법을 보자
# -------2번 방법 ----------------
s = pd.Series([20, 25, 21, 24, 29])
df['나이'] = s
display(df)  
"""
	      학과	  이름  	학년	평점	 등급	  나이
one	   수학과	 아이유	  1	  1.5  	NaN 	NaN
two	   기계과	 김연아	  2	  3.8	  NaN 	NaN
three	 철학과	 홍길동	  3 	2.9	  NaN 	NaN
four	 경영학과	강감찬	   4	  4.3	  NaN 	NaN
five	 철학과	 이순신	  2	  4.4	  NaN 	NaN
"""
# 나이 값이 안들어가졌다.. 왜?? 대체 왜??
# 자 이 series는 인덱스가 숫자 인덱스밖에 없다.
print(s)
# 0    20
# 1    25
# 2    21
# 3    24
# 4    29
# dtype: int64

# 지금 df는 우리가 따로 정한 인덱스를 갖고 있다. 
# 만약 이렇게 따로 지정한 인덱스가 있다면, 무조건 index기반으로 맵핑시켜야함
# 그래서 지정인덱스를 제공해줘야 에러가 나지 않는다.

s = pd.Series([20, 25, 21, 24, 29],
              index=['one', 'two', 'three', 'four', 'five'])

df['나이'] = s
display(df)  # 짜잔. 해결!
"""
	      학과	  이름  	학년	평점	 등급	  나이
one	   수학과	 아이유	  1	  1.5  	NaN 	20
two	   기계과	 김연아	  2	  3.8	  NaN 	25
three	 철학과	 홍길동	  3 	2.9	  NaN 	21
four	 경영학과	강감찬	   4	  4.3	  NaN 	24
five	 철학과	 이순신	  2	  4.4	  NaN 	29
"""

# 근데 그럼 시리즈 안쓰면 더 편하잖아?
# ㄴㄴ 시리즈를 쓰면 가능한게 있음.
# Series를 이용하면 모든 데이터를 넣지않고 column을 추가할 수 있음. 단!!!! index matching이 되어야함.
s = pd.Series([20, 24, 29],
              index=['one', 'four', 'five'])
df['나이'] = s 
display(df)    # 이렇게 데이터 삽입이 가능. 이게 필요한 순간이 온다
"""
	      학과	  이름  	학년	평점	 등급	  나이
one	   수학과	 아이유	  1	  1.5  	NaN 	20
two	   기계과	 김연아	  2	  3.8	  NaN 	NaN
three	 철학과	 홍길동	  3 	2.9	  NaN 	NaN
four	 경영학과	강감찬	   4	  4.3	  NaN 	24
five	 철학과	 이순신	  2	  4.4	  NaN 	29
"""

```

---

혹자는 아마 `DataFrame` 안에 있는 값(values)들만 추출하고 싶을 것이다. 어떻게 하겠는가?

```python
# 이렇게 아래와 같이 데이터가 있다고 가정하자. 
data = {'이름': ['아이유', '김연아', '홍길동', '강감찬', '이순신'],
        '학과': ['수학과', '기계과', '철학과', '경영학과', '철학과'],
        '학년': [1, 2, 3, 4, 2],
        '평점': [1.5, 3.8, 2.9, 4.3, 4.4]}

df = pd.DataFrame(data,
                 columns=['학과', '이름', '학년', '평점', '등급'],    
                 index=['one', 'two', 'three', 'four', 'five']) 
display(df)
"""
       학과     	이름    	학년    	평점     등급
one	  수학과	    아이유	      1	     1.5      NaN      
two	  기계과	    김연아	      2	     3.8      NaN
three	철학과	    홍길동	      3	     2.9      NaN
four	경영학과	 강감찬	      4	     4.3      NaN
five	철학과	    이순신	      2     	4.4     NaN
"""

# 데이터프레임의 고유 속성인 values를 사용하여 값을 추출할 수 있다.
print(df.values)
# 출력물:
# [['수학과' '아이유' 1 1.5 nan]
#  ['기계과' '김연아' 2 3.8 nan]
#  ['철학과' '홍길동' 3 2.9 nan]
#  ['경영학과' '강감찬' 4 4.3 nan]
#  ['철학과' '이순신' 2 4.4 nan]]

# 음 그럼 아까 배웠던 fancy indexing으로도 먹히나요? 
# 그럼 물론이다.
print(df[['이름', '평점']].values)
# [['아이유' 1.5]
#  ['김연아' 3.8]
#  ['홍길동' 2.9]
#  ['강감찬' 4.3]
#  ['이순신' 4.4]]

# 이 df.values와 같은 값을 리턴하는 함수가 하나 더 있다. 
# to_numpy()를 사용하면 된다.
print(df[['이름','평점']].to_numpy())

# [['아이유' 1.5]
#  ['김연아' 3.8]
#  ['홍길동' 2.9]
#  ['강감찬' 4.3]
#  ['이순신' 4.4]]

```

---

이번에는 연산을 통해 column의 값을 수정해보자.

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
"""
       학과     	이름    	학년    	평점     등급
one	  수학과	    아이유	      1	     1.5      NaN      
two	  기계과	    김연아	      2	     3.8      NaN
three	철학과	    홍길동	      3	     2.9      NaN
four	경영학과	 강감찬	      4	     4.3      NaN
five	철학과	    이순신	      2     	4.4     NaN
"""
display(df['평점']) # 칼럼이 하나니깐 pandas series로 리턴
"""
one      1.5
two      3.8
three    2.9
four     4.3
five     4.4
Name: 평점, dtype: float64
"""
df['평점'] = df['평점'] * 1.1   # 브로드캐스팅 됨
display(df)
"""
       학과     	이름    	학년    	평점     등급
one	  수학과	    아이유	      1	     1.65      NaN      
two	  기계과	    김연아	      2	     4.18      NaN
three	철학과	    홍길동	      3	     3.19      NaN
four	경영학과	 강감찬	      4	     4.73      NaN
five	철학과	    이순신	      2     	4.84     NaN
"""
# 이렇게 변경된 평점의 값을 볼 수 있다.

# 이번엔 새로운 칼럼을 만들어보자.
# 성적이 4.0을 넘는다면, 장학금을 주는 장학여부 컬럼을 주자
df['장학여부'] = df['평점'] > 4.0
display(df)
"""
       학과     	이름    	학년    	평점     등급     장학여부
one	  수학과	    아이유	      1	     1.65      NaN     False
two	  기계과	    김연아	      2	     4.18      NaN     True
three	철학과	    홍길동	      3	     3.19      NaN     False
four	경영학과	 강감찬	      4	     4.73      NaN     True
five	철학과	    이순신	      2     	4.84     NaN     True
"""
# 값이 boolean으로 떨어진걸 볼 수 있으리라.
```

---

## DataFrame에서 열(column)을 지워보자

지난 여러번의 포스팅으로 나는 여태까지 `CRUD`, create, read, update 까지 해보았다. 마지막 D를 나타내는 삭제 (delete)도 한번 해보자.

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

# NaN 으로 되어 있는 등급을 지워보자.

new_df = df.drop('등급', axis=1, inplace=False)   
# drop은 삭제의 의미를 갖는다. 
# 이게 문제가 좀 있음.
# index를 원투쓰리,,... 파이브까지 줬잖아?
# index 컬럼명도 네이밍을 할 수 있음.
# 그래서 드랍할때 컬럼이냐 (열방향) 레코드냐 (행방향) 를 axis값을 통해 알려줘야함.
# axis=0: 행방향 
# aixs=1: 열방향
# inplace = True >>> 원본도 지움  False 는 원본은 보존 
# inplace=True 인 경우 new_df 값은 None임 (원본이 아니기에..)
                    
display(df)
"""
       학과     	이름    	학년    	평점     등급
one	  수학과	    아이유	      1	     1.5      NaN      
two	  기계과	    김연아	      2	     3.8      NaN
three	철학과	    홍길동	      3	     2.9      NaN
four	경영학과	 강감찬	      4	     4.3      NaN
five	철학과	    이순신	      2     	4.4     NaN
"""
display(new_df)
"""
       학과     	이름    	학년     평점
one	  수학과	    아이유	      1	     1.5 
two	  기계과	    김연아	      2	     3.8 
three	철학과	    홍길동	      3	     2.9 
four	경영학과	 강감찬	      4	     4.3 
five	철학과	    이순신	      2     	4.4
"""
# 이렇게 '등급' 칼럼의 값을 지웠다.

# 메모리 낭비나 이런걸 감안하더라도 복사본을 만들어서 사용하는게 안전하긴하다.

# 삭제를 할때는 drop을 이용하고, 축을 명시, 그리고 inplace 값까지 줘야한다.
# 여기까지가 DataFrame의 column에 대한 CRUD 였음

```

