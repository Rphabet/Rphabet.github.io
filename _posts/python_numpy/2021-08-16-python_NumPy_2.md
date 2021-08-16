---
title: [python] NumPy 기초 (2)  
date: 2021-08-16 18:00:00 +0900
categories: [python, NumPy]
tags: [ML, python, numpy] 
math: true
comments: true
typora-root-url: ../

---

---

# NumPy

지난번 포스트와 <a> https://rphabet.github.io/posts/python-_NumPy_1/ </a> 마찬가지로 `numpy` 모듈에 대한 기본적인 것들을 짚어보는 시간을 갖겠다.

결국 지난번 포스팅의 요점은 "`ndarray` 라는 `numpy`만의 data type 을 잘다룰 줄 알아야 data handling을 수월하게 할 수 있다" 라는 것으로 귀결되었는데, 이번 포스팅에서도 마찬가지로 handling 에 도움이 될만한 그리고 근본적인 부분을 건드리고자 한다.

우선 아래의 코드를 보자.

```python
import numpy as np
arr = np.arange(0, 12, 1)  # 12개의 요소를 가진 array임.
print(arr)  # [0 1 ... 12]

# 위 벡터를 매트릭스로 바꾼다면?
result = arr.reshape(3, 4)   # arr 백터를 3x4 매트릭스로 바꾸겠다.  (12개 요소 맞추는거 중요함 size가 맞야아함! 틀릴시 에러 리턴)
print(result) 
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]   이렇게 3행 4열 매트릭스로 바뀌어있음.

# 위 result는 view라고 불린다. 왜 view 냐?
# 만약 원본인 arr의 첫번째 요소가 arr[0] = 100 으로 바뀐다면, result에도 따로 변경사항을 저장하지 않아도 반영되기 때문이다.
# 쉽게 말하면, result (view)는 데이터를 갖고 있지 않음.
# 데이터는 오직 원본 (arr)이 갖고 있다.
# 따라서
# [[ 100  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]   이렇게 출력이 됨.
# view 에 관해서는 이따가 후반부에서 다시 얘기하기로 하고, reshape()를 살펴보자.

result = arr.reshape(2, -1)  
result = arr.reshape(-1, 3)  
# 당최 이 -1은 무엇을 의미할까?
# 쉽게 얘기하면 "I don't give a shit!" 이란 소리다.
# 예를 들면 (2, -1)의 의미는 행은 무조건 2행으로 고정하고 -1의 값을 2행일때의 알맞는 shape으로 변화시켜라! 이런 소리임.
# 만약 3차원이라면 (3, 2, -1) 은 가능하지만 (3, -1, -1) 과 같이 두개 이상의 -1은 사용할 수 없다.

#################################################

# 마찬가지로 다른 view 형태인 ravel() 이라는 numpy 기능이 있다.
a = [[1, 2, 3], [4, 5, 6]]  # list
arr = np.array(a)  # numpy.ndarray
print(arr) #  [[1 2 3]
           #   [4 5 6]]


# 무조건 1차원 ndarray로 변경시키는 함수가 있음
result = arr.ravel()   # 무조건 1차원 벡터로 만들어줌
print(result)

# 한가지 기억해야할 사항은 ravel() 로 변경된 결과는 ndarray가 아니라 view임.
arr[0,0] = 100
print(arr)
print(result) # result 또한 원본 데이터가 바뀌었으니, 바뀐 값을 보여준다.

```

"이걸 굳이 알아야하나?" 싶은 생각이 들 수도 있다. 필자 또한 그랬으니깐.

음.. 중요하다. 중요한 이유는 딱 하나 `resize()` 라는 메서드를 소개하면서 간략하게 언급하겠다. 아래의 코드를 보자.

```python
# 형태를 변경시키기 위한 것으로 resize() 라는 메서드가 있다.

arr = np.arange(0, 12, 1)  # [0 1 2 ... 11]
print(arr)
print(arr.size) # 12개. 사이즈는 12

result = arr.reshape(3, 4)  # 1 x 12 벡터가 3x4 매트릭스로 바뀜
print(result)

# resize를 써보자
result = arr.resize(3, 4)    # resize는 대상을 바꾸지 않음. 자기 자신을 바꿈.
print(result) # None을 리턴함

print(arr)  # 뷰고 나발이고 얘는 원본이 바뀜. 이게 reshape랑 완전 다른점임. view가 아님.
# 그래서 굉장히 조심해야함. 나중에 에러가 나지 않기 때문에, 잘못된 사용은 결괏값에 대한 문제를 야기할 수 있음.

result = np.resize(arr, (3, 4))  # 이번엔 numpy가 갖고 있는 resize 기능을 사용하겠단 소리. 대상이 있어야함. arr가 대상이고 (3,4)로 바꿈
print(result)  # 이 친구는 리턴이 있는 아주 좋은 친구임

arr[0] = 100
print(arr)
print(result) # 원본 (arr) 이 바뀌었음에도 result 는 바뀌지 않음. view 가 아니란 소리임. 다른 ndarray가 생성됨.

# 나중에는 numpy 자료 구조가 굉장히 까다로움. 
# 데이터 핸들링이 힘들어함... 모델링은 그나마 패키지가 있어서 괜찮다. 그런데 머신러닝 입력으로 때려박기도 전에 
# 데이터 핸들링이 안되니 문제가 생김.
# numpy가 손에 익어야함. 진짜 무조건 익어야함.

# 마지막으로 하나만 더 살펴보자

arr = np.arange(0, 12, 1)
print(arr)

result = np.resize(arr, (3, 5))   # 원래는 안되지. 왜냐고? 12개 요소인데 15개 짜리 매트릭스로 어떻게 만들어?
print(result)                     # 근데 resize는 그걸 가능하게 함. 오류가 아닌 부족한 요소를 0 1 2 순으로 채움. 
#                                  resize 는 부족한 요소를 채워준다. 기억하자. 이게 좋은게 아님. 어쩌면 오류가 난걸 모르고 편향된
                                  # 데이터 셋을 만들어 분석을 할 수도 있는거임.

result = np.resize(arr, (2, 3))   # 이런게 가능하기 때문에 resize는 항상 조심..
print(result)                     # 가능한 resize 대신에 reshape 을 쓰자.  여러가지 오류에 대비해야하기때문. 
                                  # 우리가 오류 포착을 자동으로 못함
    
# 지금 우리 여기까지 배운것이 numpy의 기본. 상중하 중에 기본. 가장 기본
# 무조건 알아야함.
# 이제부터 머신러닝의 끝까지 기본 자료구조는 numpy. 파이썬이 아니고 Numpy.. 무조건 잘해야함. 
```

다음엔 어떻게 numpy로 인덱싱을 하는지 한 번 예를 통해 알아보도록 하자.



