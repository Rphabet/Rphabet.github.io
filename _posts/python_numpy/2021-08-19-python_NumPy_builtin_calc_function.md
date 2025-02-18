---
title: python NumPy만의 집계함수
date: 2021-08-19 11:05:00 +0900
categories: [python, NumPy]
tags: [ML, python, numpy, 머신러닝] 
math: true
comments: true
image:
  src: /../assets/images/Numpy/logo.png
  width: 500
  height: 200
  alt: numpy logo
typora-root-url: ../
---

---

`NumPy`의 `ndarray`는 다양한 **집계함수**를 제공한다. 

조금이라도 `numpy`를 사용해보신 분들이라면, 아마 익숙한 기능일 것이다.

전반적으로 가볍게 "이런 기능이 있어요~" 라고 소개를 하겠지만, 2차원이상부터는 `axis`를 설정하면서 계산을 하기 때문에 조금 자세하게(?) 다루지 않을까 생각한다.

자 지지지지지(?)난번 포스팅에서 배웠던 `ndarray`를 만드는 여러가지 중 하나를 이용하여 예제를 만들어보자.

```python
arr = np.arange(1, 7, 1).reshape(2, 3).copy()  # 2x3 matrix
# copy를 쓰는 이유는 view가 아닌 원본이 필요해서임
# [[1 2 3]
#  [4 5 6]]

# 요소들의 합
print(arr.sum())  # 21
print(np.sum(arr)) # 21   이렇게 써도 됨.

# 요소들의 평균값
print(arr.mean()) # 3.5

# 요소들의 표준편차
print(arr.std()) # 1.7078.....

# 요소중 제일 큰 값
print(arr.max()) # 6

# 요소중 제일 작은 값
print(arr.min()) # 1

# 요소중 제일 큰 값의 index를 알려주는 함수 (위치를 찾아주는 함수)
print(arr.argmax())  # 5
```

흠...🤨  쉽네요? 그냥 단순 연산인데요?  맞다

하지만 눈치 빠르신 분들중에선 특히 `argmax` 와 같은 메서드를 사용하였을 때 이런 질문을 할 수 있을것이다.

"**아니 2x3 행렬인데 인덱스로 (1, 2)가 나와야죳!!! 왜 5가 나와요?**" `argmax()`, `argmin()` 함수는 어떤 형태의 행렬이던간에 1차원으로 풀어서 인덱스를 계산하기에 1차원으로 풀었을 때인 5가 값으로 나온다. `[1 2 3 4 5 6]`.

막간을 이용해 집계함수에 대한 칭찬을 하나 하자면, 연산 속도가 말도 안되게 빠르다. 우리가 보통 로직을 짜서 뭐 조건문을 걸어서 어쩌구 저쩌구 하는 것보다 훨씬 빠르다는 것을 알아두자. 

자 그러면 뭐가 까다로운 것인가?

1차원인 배열에선 문제될 부분이 하나도 없다. 그냥 집계함수를 있는 그대로 사용하면 되니깐.

하지만 2차원부터는 `axis`를 설정해줘야할 때가 있다.

아래 코드를 통해 살펴보자.

```python
###### 1차원
arr = np.array([1, 2, 3, 4, 5])
print(arr.sum())  # 출력물: 15
# 축이고 나발이고 없으니깐, np array안에 있는거 싹다 더하는거야
# 1차원은 1행 n열이잖아. 행이 하나만 있음. 그러니깐 열을 기준으로 연산 수행 

print(arr.sum(axis=0))  # 출력물: 15
# 1차원 축의 번호는 0이고, 이때 0의 의미는 열이다  그러니깐 열이 가는 방향으로 sum을 구하라는 소리이다
# 1차원에선 axis의 최댓값은 0이고 최솟값 또한 0이다. 

##### 2차원
arr = np.array([[1, 2, 3]
                [4, 5, 6]])   # 2x3 행렬

print(arr.sum()) # 출력물 21. axis=None by default

# 2차원부터 axis는 2개의 값을 가질수 있다. 0 또는 1
print(arr.sum(axis=0))  # axis=0 은 행방향으로 연산을 수행한다 
# 출력물:
# [5 7 9]
print(arr.sum(axis=1))  # axis=1 은 열방향으로 연산을 수행한다
# 출력물: 
# [6 15]
```

"**와 뭐야? 왜 결괏값이 저렇게 배열안에서 나와???**" 이게 열방향과 행방향 구분을 잘해야한다.

필자가 구별하는(?) 나름의 직관적인 방법을 공유해보자면..

1차원에선 행은 없고 열만 있다.  `axis=0` 는 열방향이다.

2차원에선 행과 열 모두 있다. `axis=0` 는 행방향이 되고, `axis=1` 은 열방향이 된다.    그러니깐 차원이 늘어나고 행이라는 개념이 새롭게 적용이 되니깐 기존의 열방향이었떤 `axis=0 ` 이 행방향이 된거다.   1차원에선 0살이었던 **열방향**이 2차원에선 한살 더먹어서 1살이 되었다고 생각하면 괜찮게 이해가 되는거 같았다.

마찬가지로 3차원에선 **면** 이라는 개념이 추가된다. 필자의 논리대로라면 `axis=0`은 0살인 면방향이, `axis=1`은 이제 한살이 된 행방향이, 그리고 `axis=2`는 이제 두 살이된 열방향이 가져가게된다.

처음엔 굉장히 헷갈릴 수 있으니, 이해하지 못하겠다면 있는 그대로로 받아들여 외워버리자. 결국 이해가 되는 순간이 오리라 믿는다.

자 "**그럼 열방향과 행방향이 뭐에요?**" 라고 생각할 수 있다

음... 간략하게 설명을 해보자면

```python

행방향: 행은 위에서부터 1행, 그다음 2행, ..... 맨밑에 있는 행이 가장 마지막 행이다.
      그렇기때문에 행방향이라고 한다면 "위에서 아래로" 라고 생각하면 이해하기 쉬울 것이다.
아까 수행했던 sum() 연산을 적용해보자면... 

⬇️ [[1 2 3]      
⬇️  [4 5 6]]
   ----------
    [5 7 9]

이렇게 화살표 방향으로 같은 열안에서 연산이 수행된다.

열방향: 열은 제일 왼쪽이 1열, 그 바로 오른쪽 열이 2열, 그 다음이 3열 .... 맨 오른쪽이 제일 끝 열이다.
     그렇기때문에 열방향이라고 한다면 "왼쪽에서 오른쪽" 이라고 생각하면 이해하기 쉬울것이다.
아까 수행했던 sum() 연산을 적용해보자면... 

➡️ [[1 2 3]     = 6
➡️  [4 5 6]]    = 15       그래서 [ 6 15 ] 라는 연산이 나왔던 것이다

```

자 그럼 `axis`를 이용한다면 3차원은 어떻게 연산이 되는걸까?

```python
# 이번엔 난수를 이용해서 3차원 array를 만들어보자.
# 난수를 이용하는 이유? 그냥 다양한 방법으로 np.array를 만들 수 있다는 것을 보여주고 싶어서
np.random.seed(1)   # 시드값을 잡아주자. 시드 값은 아무값이나 줘도 됨. 대신 변경하면 안됨

arr = np.random.randint(0, 5, (2, 2, 3))  
# randint 정수로 구성된 난수를 만들거다
# 0부터 4중 하나의 숫자로 요소의 값이 됨. (5는 미포함)
# 차원은 3차원
# shape은 (2,2,3) 또는 2x2x3 임.

print(arr)
# 출력물:
# [[[3 4 0]
#   [1 3 0]]

#  [[0 1 4]
#   [4 1 2]]]

# 크 벌써부터 예술로 헷갈리게 생겼다.

# 일전에 얘기했다시피 axis=0 은 새롭게 추가된 면 방향이다.
# 한번 사용해보자.
print(arr.sum(axis=0))  # 자 array의 sum을 구할건데 면방향으로 계산해줘!
# 면 방향은 내가 화살표로 표현하기 어렵다. 하지만 한 매트릭스에 있는 요소와 
# 다른 매트릭스지만 그 자리에 위치한 요소끼리 더해준다고 생각하면 된다!
# 출력물:
# [[3 5 4]
#  [5 4 2]]         

"""
쉽게 얘기하자면 

[[[a b c]
  [d e f]]

 [[A B C]
  [D E F]]]

a + A 
b + B 
...
...
f + F  
이런식으로 연산이 된다.
"""

# 자 그럼 행방향은 어떻게 되는 것인가?
print(arr.sum(axis=1))
# 출력물:
# [[4 7 0]
#  [4 2 6]]
# 여기서는, 각 면에 속한 매트릭스에서 행방향으로 더한 값을 나열한 값이다.

# 그렇다면 열방향은 어떻게 되는가?
print(arr.sum(axis=2))
# 출력물:
# [[7 4]
#  [5 7]] 
# 마찬가지로, 각 면에 속한 매트릭스에서 열방향으로 더한 값을 나열하면 된다.

```

음 물론 언젠가 이 포스팅을 수정을 해야할 시점이 올 수도 있다고 생각한다.

거의 나만의 복습용 저장소로 사용하기도 하거니와, 외부에 공개해놓은 다이어리(?) 형태이기 때문에 글의 문맥에 맞춰 가독성 높게 포스팅을 작성하진 못하였다. 

혹여나 다시 수정하는 날이 온다면 그림을 통해 추가적인 보충 설명을 해보겠다.

이 글을 읽는 모든 분들 **happy coding** 하시길..😃 

지금은 여기까지.. 그럼 안녕 🙋‍♂️ 👋 🤠 





