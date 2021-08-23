---
title: python Pandas (7) DataFrame 정렬
date: 2021-08-23 09:51:00 +0900
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

## 정렬

바로 코드를 통해 알아보자.

```python
import numpy as np
import pandas as pd

# 난수를 무작위로 줘서 데이터 정렬을 해보자.
np.random.seed(1)  # seed로 처음 난수 값을 고정
df = pd.DataFrame(np.random.randint(0, 10, (6, 4)))  # 6x4 매트릭스 생성 (요소 값은 0에서 9사이)
df.columns = ['A', 'B', 'C', 'D']  # 컬럼값 추가
df.index = pd.date_range('202010801', periods=6) 
# 2021년 8월 1일부터 시작하는 6개의 날짜를 인덱스로 넣음.

display(df)
"""
           	A	B	C	D
2021-08-01	5	8	9	5
2021-08-02	0	0	1	7
2021-08-03	6	9	2	4
2021-08-04	5	2	4	2
2021-08-05	4	7	7	9
2021-08-06	1	7	0	6
"""

# index를 랜덤하게 섞어보자
random_date = np.random.permutation(df.index)  # 랜덤하게 섞고 복사본을 만듬. shuffle은 view임.

# 날짜가 무작위로 섞인 새로운 df를 만들어보자
new_df = df.reindex(index=random_date, columns=['B', 'A', 'D', 'C'])  
# 컬럼순서도 임의로 변경
display(new_df)   # 이렇게 원본 df의 값은 그대로지만, 순서가 뒤죽박죽 엉킨걸 볼 수 있다.
"""
           	B	A	D	C
2021-08-03	9	6	4	2
2021-08-01	8	5	5	9
2021-08-04	2	5	2	4
2021-08-05	7	4	9	7
2021-08-02	0	0	7	1
2021-08-06	7	1	6	0
"""
```

자 이제 섞어놓은 데이터를 갖고 정렬을 해보자. 인덱스 기반으로 정렬할 수도 있고, 값을 기반으로 정렬할 수도 있다.

```python
sort_df = new_df.sort_index(axis=1, ascending=True) # 열방향 정렬 ABCD순으로 컬럼이 바뀌었다
display(sort_df)
"""
            A	B	C	D
2021-08-03	6	9	2	4
2021-08-01	5	8	9	5
2021-08-04	5	2	4	2
2021-08-05	4	7	7	9
2021-08-02	0	0	1	7
2021-08-06	1	7	0	6
"""

# axis=0인 행방향 정렬도 한번 살펴보자.
sort_df = new_df.sort_index(axis=0, ascending=True) # 행방향 정렬 날짜순으로 값이 바뀌었다.
"""
           	B	A	D	C
2021-08-01	8	5	5	9
2021-08-02	0	0	7	1
2021-08-03	9	6	4	2
2021-08-04	2	5	2	4
2021-08-05	7	4	9	7
2021-08-06	7	1	6	0
"""

# 최종적으로 두개를 합쳐서 사용한다면.. 정렬된 원본 데이터를 구할수 있으리라.

sort_df = new_df.sort_index(axis=0, ascending=True)  # 행방향으로 날짜 정렬
df_sorted = sort_df.sort_index(axis=1, ascending=True) # 다시 열방향으로 최종 정렬
display(df_sorted)
"""
          	A	B	C	D
2021-08-01	5	8	9	5
2021-08-02	0	0	1	7
2021-08-03	6	9	2	4
2021-08-04	5	2	4	2
2021-08-05	4	7	7	9
2021-08-06	1	7	0	6
"""
```

이렇게 `column`이름과 `index` 이름을 기준으로 정렬하는 방법이 있는가하면, 특정 column의 값(value)으로 정렬하는 방법도 있다.

```python
display(new_df)  # 위와 같은 new_df를 사용
"""
            B	A	D	C
2021-08-03	9	6	4	2
2021-08-01	8	5	5	9
2021-08-04	2	5	2	4
2021-08-05	7	4	9	7
2021-08-02	0	0	7	1
2021-08-06	7	1	6	0
"""

# D의 값을 기준으로 오름차순 정렬은 한다고 하면..
# 이번엔 sort_values 메서드를 이용하자
sort_df = new_df.sort_values(by='D')  
# D라는 컬럼을 갖고 정렬
display(sort_df)
"""

            B	A	D	C
2021-08-04	2	5	2	4
2021-08-03	9	6	4	2
2021-08-01	8	5	5	9
2021-08-06	7	1	6	0
2021-08-02	0	0	7	1
2021-08-05	7	4	9	7
"""

# 이렇게 행(row)들이 2의 오름차순 순으로 정렬이 된 것을 볼 수 있다.
```

**Q. 그럼 만약에 오름차순 정렬을 하는데 두개이상의 값이 동일한 값이면 어떻게 하나요?**

- 추가적인 조건을 걸어주지 않았다면, 시스템이 임의로 결정
- 동률값이 나온다면 '`<컬럼 이름>`'을 기준으로 정렬하라고 알려줄 수 있음.

같은 값이 있는 `A`를 기준으로 정렬을 해보자!

```python
sort_df = new_df.sort_values(by=['A', 'D'], ascending=True)
display(sort_df)
# by=['A', 'B']는  by='A'로 먼저 정렬을하고, 동률값이 나온다면 'D'를 갖고 정렬해라. 이소리임

"""
	B	A	D	C
2021-08-02	0	0	7	1
2021-08-06	7	1	6	0
2021-08-05	7	4	9	7
2021-08-04	2	5	2	4
2021-08-01	8	5	5	9
2021-08-03	9	6	4	2
"""
```



끄읏. 👋 