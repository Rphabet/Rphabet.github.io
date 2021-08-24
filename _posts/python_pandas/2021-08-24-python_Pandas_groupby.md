---
title: python Pandas (11) DataFrame Grouping
date: 2021-08-24 16:47:00 +0900
categories: [python, pandas]
tags: [python, numpy, pandas, grouping, groupby] 
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

## DataFrame 그룹핑

그룹핑은 pandas`기능과 데이터 분석의 꽃이 아닐까 생각한다.

특정 집단의 집계함수나 유의미한 결과 추론은 어떤 형태의 데이터 분석이든간에 유용하게 쓰이기 때문이다.

`pandas` 자료구조인 `Series`와 `DataFrame`을 갖고 어떻게 그룹핑을 하는지 알아보자.

### Series 그룹핑

```python
import numpy as np
import pandas as pd

# 아래와같이 데이터 프레임을 하나 잡아주자!
df = pd.DataFrame({
    '학과': ['컴퓨터', '철학', '컴퓨터', '철학', '컴퓨터'],
    '이름': ['아이유', '김연아', '홍길동', '강감찬', '신사임당'],
    '학년': [1, 2, 3, 2, 3],
    '학점': [1.5, 2.7, 3.5, 1.9, 4.0]
})

display(df)
"""
   학과    이름  학년  학점
0 컴퓨터  아이유   1  1.5
1 철학    김연아  2  2.7
2 컴퓨터  홍길동  3   3.5
3 철학    강감찬  2  1.9
4 컴퓨터  신사임당 3  4.0
"""

# 1 단계 그룹핑
dept = df['학점'].groupby(df['학과'])
display(dept) # <pandas.core.groupby.generic.SeriesGroupBy object at 0x7feb21e27c10>
# dept는 SeriesGroupBy의 객체임.
# 평균값을 알려주는 메서드를 불러오자.
display(dept.mean())
"""
학과
철학     2.3
컴퓨터    3.0
Name: 학점, dtype: float64
"""
# 이렇게 학과별 평균 학점을 알려준다.

# dept는 또한 같은 그룹내 멤버들의 값을 알려주기도 하는데
computer = dept.get_group('컴퓨터') 
# 컴픁에 dept가 갖고 있는 '컴퓨터' 그룹의 특성 (여기선 학점)을 가져와서 computer로 assign 해라!
display(computer)
"""
0    1.5
2    3.5
4    4.0
Name: 학점, dtype: float64
"""

# 또한 그룹 사이즈를 알려주기도 함.
g_size = dept.size()
print(g_size)  # 각 그룹의 사이즈를 알려줌
"""
학과
철학     2
컴퓨터    3
Name: 학점, dtype: float64
"""
# 컴퓨터학과 그룹의 사이즈는 총 3명, 철학과 그룹의 사이즈는 총 2명이란 소리임.
```

2단계 그룹핑을 할 수도 있다.

```python
df = pd.DataFrame({
    '학과': ['컴퓨터', '철학', '컴퓨터', '철학', '컴퓨터'],
    '이름': ['아이유', '김연아', '홍길동', '강감찬', '신사임당'],
    '학년': [1, 2, 3, 2, 3],
   '학점': [1.5, 2.7, 3.5, 1.9, 4.0]
})

# 2단계 그룹핑
dept = df['학점'].groupby([df['학과'], df['학년']])
display(dept.mean())
"""
학과    학년
철학     2      2.30
컴퓨터    1      1.50
        3      3.75
Name: 학점, dtype: float64
"""
# 결괏값은 2차 인덱스로 떨어졋음.
# 인덱스가 두개란 소리임.
# pandas Series 임을 기억하자

# 이렇게 처리하면 2차 index를 이용하는 형식이 되기 때문에 사용하기 많이 불편!
# 많이 사용하는 방식은 최하위 인덱스를 column으로 만들어서 DATAFRAME으로 변환
# 이걸 해주는 함수가 하나 있는데 그게 바로 unstack()
display(dept.mean().unstack())  
"""
학년     1    2     3
학과
------------------------
철학   NaN   2.3   NaN
컴퓨터  1.5   NaN   3.75
"""
# 이렇게 하는게 더 깔끔함. 하지만 만능은 아니다. 
# 3, 4 차 넘어가면 dataframe 표현이 안되기에 불가.

```

### Data Frame 그룹핑

일반적으로 `Series`보다 흔하게 사용할 일이 많다.

```python
import numpy as np
import pandas as pd

df = pd.DataFrame({
    '학과': ['컴퓨터', '철학', '컴퓨터', '철학', '컴퓨터'],
    '이름': ['아이유', '김연아', '홍길동', '강감찬', '신사임당'],
    '학년': [1, 2, 3, 2, 3],
    '학점': [1.5, 2.7, 3.5, 1.9, 4.0]
})

# 아까 series때와 정확하게 같은 데이터 사용
# 전에는 칼럼 하나만 떼어와서 pandas series로 왔음.
# 이번엔 데이터프레임 전체에 대해서 groupby 할 것임.
dept = df.groupby(df['학과'])  # groupby 하는건 똑같음
display(dept) # <pandas.core.groupby.generic.DataFrameGroupBy object at 0x7feb21e2e0d0>
# DataFrameGroupBy 로 바뀜.

display(dept.get_group('컴퓨터')) # 컴퓨터 그룹에 대한 정보를 리턴
# 데이터 프레임으로 리턴! 
"""
   학과   이름   학년   학점
0 컴퓨터  아이유   1    1.5
2 컴퓨터  홍길동   3    3.5
4 컴퓨터  신사임당  3    4.0
"""

display(dept.size())  # 사이즈는 series 때와 똑같음
"""
학과
철학      2
컴퓨터    3
dtype: int64
"""
display(dept.mean()) # 왜 df로 나오나요? 학년도 숫자로 되어 있어서 평균값이 나옴.
                    # 그렇지만 학년을 str로 지정해줘도 df로나옴. 원본 데이터가 df라 df로 리턴
"""
      학점
학과	
철학   2.3
컴퓨터  3.0
"""
```

### 그룹핑 반복처리

각각의 그룹에서 반복 처리는 어떻게 하는가?

```python
import numpy as np
import pandas as pd

df = pd.DataFrame({
    '학과': ['컴퓨터', '철학', '컴퓨터', '철학', '컴퓨터'],
    '이름': ['아이유', '김연아', '홍길동', '강감찬', '신사임당'],
    '학년': [1, 2, 3, 2, 3],
    '학점': [1.5, 2.7, 3.5, 1.9, 4.0]
})

display(df)
"""
   학과    이름  학년  학점
0 컴퓨터  아이유   1  1.5
1 철학    김연아  2  2.7
2 컴퓨터  홍길동  3   3.5
3 철학    강감찬  2  1.9
4 컴퓨터  신사임당 3  4.0
"""

for dept, group in df.groupby(df['학과']):
    print(dept)
    display(group) # dept 그룹에 대한 dataframe이 나옴.
"""
i=0 (첫번째 iteration):  
(학과)
철학

  학과   이름   학년   학점
1 철학   김연아   2    2.7
3 철학   강감찬   4    1.9

i=1 (두번째 iteration):
(학과)
컴퓨터

   학과   이름   학년   학점
0 컴퓨터  아이유   1    1.5
2 컴퓨터  홍길동   3    3.5
4 컴퓨터  신사임당  3    4.0
"""
# 이렇게 반복 시킬 수 있음.

```



2개를 기준으로 그룹핑을 한다면 어떻게 될까?

```python
# 2개를 기준으로 그룹핑을 한다면???
for (dept, year), group in df.groupby([df['학과'], df['학년']]):
    print(dept, year)
    display(group)
"""
i=0 (첫번째 iteration):
(학과) (학년)
 철학     2

  학과   이름   학년   학점
1 철학   김연아   2    2.7
3 철학   강감찬   2    1.9


i=1 (두번째 iteration):
(학과) (학년)
 컴퓨터   1
 
   학과   이름   학년   학점
0  컴퓨터 아이유   1    1.5

i=2 (세번째 iteration):
(학과) (학년)
 컴퓨터   3
  
   학과   이름   학년   학점
2 컴퓨터  홍길동   3    3.5
4 컴퓨터  신사임당  3    4.0
"""
```

이렇게 `groupby`인자로 나왔던 **학과**와 **학년** 별로 나뉘어서 반복문이 수행된다. 

그리고 해당 그룹에 속한 멤버들을 `dataframe`으로 보여준다.

우선 그룹핑은 여기까지..

끄읏. 👋 