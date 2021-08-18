---
title: python NumPy만을 위한 반복문 iter (loop)
date: 2021-08-18 20:38:00 +0900
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

오늘 `Numpy` 관련 포스팅을 두개나 올릴 예정이다ㅎㅎ (현재 8월 18일) 미리 부지런한 내 스스로에게 박수를 👏 

이번 포스팅은 반복분 (`loop`)에 관해서 다루려고 한다.

**Q**. 그냥 `for`, `while` 루프 쓰면 되지 않나요???  

조건부 가능하다. 1차원 데이터에서는 `for`루프 효율이 좋아보이기도 하고. 

그런데 `numpy`를 사용한다는 의미는 곧 데이터 분석을 위해 사용한다는 의미라고도 받아들여지는데 1차원인 데이터는 찾아보기 힘들다. 보통은 2차원 이상인 데이터가 있을뿐이지.

"**2차원도 왜요?? for문 두번 돌리면 되죠!!**"  => 음.. 그럼 3차원? 4차원은? for문을 계속 추가할거야?

`for` 루프가 1차원에선 효과적이라는데엔 이견이 없다. 하지만 2차원이상부터는 얘기가 다르다. 

간단한 예제를 통해 알아보자.

## 1차원: `for` vs `iter`

```python
import numpy as np

arr = np.array([1, 2, 3, 4, 5])  

# 자 n-dimensional array 안의 요소를 어떻게 출력할래?
# -> 포문 돌리면 되죠!
for element in arr:
    print(element, end=' ')  # end=' ' 는 한줄에 코드를 출력하기 위해 추가했다.
# 출력물: 1 2 3 4 5   
```

뭐야 일반 `for`루프잖아요. `numpy`에선 다른거 쓰라면서요??  --> 아래 코드를 보자

```python
arr = np.array([1, 2, 3, 4, 5])  

it = np.nditer(arr, flags=['c_index'])

while not it.finished:
    idx = it.index
    print(arr[idx], end=' ')
    it.iternext()
    
# 출력물: 1 2 3 4 5
```

"**뭐야 코드만 복잡해지고 더 비효율적인거 아닌가요 ㅡㅡ?**" 🤬 

1차원에선 그렇다. 하지만 믿어라. 2차원부턴 신세계를 경험한다. 

그전에 코드 설명부터 간략하게 하고 넘어가겠다.

`numpy`모듈안에는 `nditer()`라는 메서드가 있다. 간단하게 말하면 인자로 들어오는 array를 `iterator`(반복자) 라는 녀석으로 만들어주겠다는 소리이다.     `it = np.nditer(arr, flags=['c_index'])` 뒤에 `flags`에 `'c_index'`라는 친구가 값으로 들어온 것을 볼 수 있는데  이친구는 1차원 배열에 한해서만 쓰이는 `c`언어 기반의 index를 사용하겠다는 소리이다.   음 걱정마시라.  우리가 사용해온 인덱스는 c언어 인덱스와 같으니깐. 

"**그래서 어쩌라고요! 2차원에서 뭐가 효율적인건데요**" 😣 

자 아래의 예제를 보자.

## 2차원: `for` vs `iter`

```python
arr = np.array([[1, 2, 3],
                [4, 5, 6]]) 
print(arr)
# 출력물:
# [[1 2 3]
#  [4 5 6]]   

# 이 친구들 하나하나를 for 문으로 찍어보자

for row in range(0,arr.shape[0]):
    for col in range(0,arr.shape[1]):
        print(arr[row, col], end=' ') 
# 출력물: 1 2 3 4 5 6
```

자 무엇이 느껴지는가? 

"**어? 코드가 복잡해지네? array내 요소가 증가하면, 차원이 증가하면, 어떡하지?**" 이런 느낌이 오는가?

루프를 중복해서 돌렸을 때 큰 문제점을 꼽아보라면....

1. 코드 읽기가 굉장히 복잡해진다
2. 로직 구축하는게 어려워진다

맞다. 특히 백만개의 데이터를 돌린다고 생각해보자. 답도 없다 이건 => 이걸 `complexity` 가 증가한다고 표현한다.

자 이번엔 `iterator`를 사용해보자.

```python
arr = np.array([[1, 2, 3], [4, 5, 6]]) 
# 이렇게 oneline으로 코드를 쓰는것이 원칙(?)이긴하다.

it = np.nditer(arr, flags=['multi_index'])     
# 어 뭐야? flags 값이 변했네? 2차원이상부턴 multi_index를 쓴다.
# c_index는 1차원에서만.. 한마디로 쓸일이 없다는 소리지

while not it.finished:
    idx = it.multi_index
    print(arr[idx], end=' ')
    it.iternext()
# 출력물: 1 2 3 4 5 6
```

자 보시라. 훨씬 간단하지 않은가?  3, 4, 5, .... 차원 이상을 다룬다고 생각해봐라.  효율성 측면에서 `for`문이 반복자를 이길 수 있겠는가? 어림 반푼어치도 없다.

## 간략한 코드 설명

이제 간단하게 코드에서 사용된 커맨드 설명을 간단하게 해보겠다.

```python
# np.nditer() 
# n-dimensional iter 임. '반복자'라는 객체를 ndarray에서 하나 "푝" 하고 뽑아낼수 있음.
# 한번 iterator 객체를 뽑아내보자

it = np.nditer(arr, flags=['c_index'])

# iterator를 뽑아내는데 어떤 iterator객체냐? 지정된 flag형태를 가진 iterator 임.
# 1차원인 경우엔 c_index 라는 정해져있는 값을 사용할 수 있음
# 앞의 c는 프로그래밍 언어인 c이다. 뜻은 c언어에서 사용하는 index 형태로 iterator를 만들겠다.
# 이소리임.
# 이렇게 만든 iterator로 부터 우리가 index를 추출 할 수 있는데
# 이 index의 형태가 c언어에서 사용하는 index의 형태와 같다!
# c언어에서 사용하는 인덱스의 형태가 뭔데? ==> 0, 1, 2, 3, 4, .... 우리가 사용하는것과 같음.
# 반복자라는 값이 내부에 index를 갖고 있는데, 
# 이 iterator 로 뭐하는데? 
# finished 를 이용해서 반복처리가 끝났는지 물어볼 수 있음.
# 그럼 iterator 가 'ㅇㅇ 끝났음' 또는 'ㄴㄴ아직 시작도 안했어' 등과 같이 알려줌.
while not it.finished:    # 처음 루프를 돌릴때 not it.finished 를 쓰는 이유는
  
  # while 문은 it.finished가 참이라면 돌거야! 
  # 그런데 참일 수가 없지. 왜냐고? 시작하자마자 끝났어? 물어보는거잖아.
  # 당연히 안끝났지. 그래서 False로 값이 떨어지니깐 루프는 돌지도 못해보는거야
  # 그래서 앞에 not 처리를 해줘서 not False => True 이렇게 만들어 주는거다.
  idx = it.index
  # iterator가 갖고 있는 index를 뽑아내는거임
  print(arr[idx], end=" ") # 프린트로 찍고
  it.iternext()   # 다음 인덱스로 넘어가자.
  # 만약에 모든 인덱스를 다돌았다?
  # while not it.finished 에서 False 값이 나와 루프는 끝나게 된다.
```

`for`문과 `iterator`방식의 가장 큰 차이점은 `for`문은 자동으로 탁탁탁탁 넘어가지만, `iterator`는 아니라는 점이다.

이제 2차원 행렬에 쓰인 코드를 다시 살펴보자.

```python
it = np.nditer(arr, flags=['multi_index']) 
# multi_index로 바꿔준다.
# 이게 진짜 엄ㅁㅁㅁㅁ청 중요함
# multi index니깐 tuple 로 빠짐. 결괏값을 보면 알수 있다. 
# 이친구는 현재의 행과 열을 알려준다. 
# current row and column index 를 준다 이소리임ㅇㅇ.
while not it.finished:
    
    idx = it.multi_index         
    # 맨 처음엔 인덱스 값으로 (0,0) 을 주지. 순서대로 돌아간다 (0, 1) (0, 2) . . . .
    
    print(arr[idx], end=" ")

    it.iternext()  # iternext라는 함수를 통해 다음으로 보내주자.

```

`iter`를 사용하는게 정말 좋은 방식의 코딩인게

3차원, 4차원이 되어도 코드가 안바뀜. 진짜 너무나도 신세계임. 복잡도 안올라가, 로직도 훨씬 좋아. 걍 프로그램적 측면에서는
이친구가 훨씬 좋은 표현이고 방법이야.
내가 갖고 있는 데이터에 따라서 코드를 바꿀 필요가 없는거지. 

반면에 `for` 루프를 생각해보자. 차원이 올라간다? 로직 하나부터 열까지 다시 다 짜야함... 

객체지향이 뜬 이유는 유지 보수성이 압도적으로 좋기 때문이다. 유지보수에 맞춰서 코드를 짜는 습관을 들이자ㅋㅋ  <<< 갑자기??? 

아무쪼록 음.. 조만간 미분에 대한 프로그래밍을 할건데 그때도 이 `iter` 라는 친구를 사용해야한다. 

그때 다시 '실용적으로 어떻게 활용할 수 있는가'에 대해서 포스팅을 해보겠다.

오늘은 여기까지. 그럼 안녕 🖖 🌵 