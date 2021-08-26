---
title: python으로 하는 수치미분 (Numerical Differentiation) (2)
date: 2021-08-26 16:30:00 +0900
categories: [python, mathematics]
tags: [python, differentiation, numerical] 
math: true
comments: true
typora-root-url: ../

---

---

## python으로 편미분하기

지난번 포스팅 ([python으로 하는 수치미분 (Numerical Differentiation)](https://rphabet.github.io/posts/numerical_differentiation/)에서 어떻게 파이썬으로 수치미분을 하는지 알아보았다.

이번엔 **다변수**로 넘어가서 어떻게 편미분을 하여 미분 계수를 구하는지 한번 알아보자.

다변수이기 때문에 코드가 비교적 더 복잡해진다.

다음과 같은 식을 편미분해본다고 생각해보자.

$$ f(x, y) = 2x + 3xy + y^3 $$ 

음.. 코드 어제 배운거 있고, 
**수치미분** 함수 하나 정의하고, 
저 식에 대한 함수 하나 정의해서  "**땋!**" 적용하면 바로 결괏값 나오는거 아님????

정답이다. 
하지만 이상적인 정답이라고 할 순 없다!

왜냐고?? 만약에 변수가 $ f(x, y, z) $ 라면? 또는 $ f(a, b, c, d, e, f, g, h, i, j, k, l, ..., z) $ 라면? 
식에 대핸 새로운 함수를 정의하고 그 함수에 대응하는 수치미분 함수를 또 다시 정의하고 코드를 뜯어 고쳐야하기 때문이다.

파이썬이 흥행한 이유가 무엇인가?? **유지 보수성**이 탁월하기때문 아니던가! 

이번 포스팅은 이런 점을 고려하여 어떤 식이든, 또 어떤 차원의 행렬을 인수로 받던간에 바로 **수치미분 함수**에 적용할 수 있는 **일반화**된 **수치미분 함수**를 만들어 볼 것이다.


그럼 바로 코드로 고고 🚀 

```python
import numpy as np   # numpy 패키지를 이용하여 차원에 대응할것임.

# 수치미분을 수행할 함수를 만들어보자
def numerical_derivative(f, x):
    """
    편미분을 여러번 돌려야함 (vector of partial derivatives)
    f: 우리가 돌리려고 하는 식을 함수로 적어놓은것. e.g. f(x, y)
    x: 함수에 쓰일 독립변수. 행렬이 될 수도, 1차원 변수가 들어올 수도 있음.
    """
    
    delta_x = 1e-4   # x 변화량의 크기 (극한 대신 아주 작은 수로 대체)
    derivative_x = np.zeros_like(x)  # x와 똑같은 shape을 가진 배열(혹은 행렬) 생성 (요소값은 0임)
    
    it = np.nditer(x, flags=['multi_index']) # iterator 하나 생성. 플래그는 멀티 인덱스 설정
    
    while not it.finished:
        idx = it.multi_index
        tmp = x[idx]   # 임시 변수로 x[idx] 의 원값을 저장
        
        x[idx] = tmp + delta_x  # 전향 차분
        fx_plus_delta_x = f(x)  # 첫번째 인자로 들어온 함수를 대입
        
        x[idx] = tmp - delta_x  # 후향 차분
        fx_minus_delta_x = f(x)  # 첫번째 인자로 들어온 함수를 다시 대입
        
        # 중앙 차분
        derivative_x[idx] = (fx_plus_delta_x - fx_minus_delta_x) / (2 * delta_x)
        
        x[idx] = tmp # x값 원상태로 복구
        
        it.iternext()  # 다음 인덱스로 넘어가자
        
    return derivative_x
    
```

자 이렇게 **수치미분**을 계산해주는 함수를 하나 정의해보았다. ☝️ 

이제 첫번째 인자로 들어갈 수학적 식을 하나 만들어보자. 

```python
def f(input_values):
    """
    이 함수는 f(x, y) = 2x + 3xy + y^3 의 수식을 가진 함수이다.
    이 함수는 numerical derivatives 와 같은 수학적 계산 목적을 가진 함수에서만 
    사용될 수학적 식을 담은 함수이다.
    input_values를 사용한 까닭은 독립변수 개수와 상관없이
    어떤 경우에도 일반화할 수 있는 함수를 만들기 위함이다.
    """
    x = input_values[0]
    y = input_values[1]
    
    return 2*x + 3*x*y + np.power(y, 3)
    
```

오케이 👌 

잘 작동하는지 한번 살펴보자.

```python
result = numerical_derivative(f, np.array([1.0, 2.0]))  # x = 1.0, y= 2.0 의 값을 갖고 있음
print(result)
```

```python
결과값은:
[ 8.         15.00000001]
```

저 결과값이 극한을 이용한 해석미분으로도 값이 비슷할까? 한번 살펴보자!

$$ f(x, y) = 2x + 3xy + y^3 $$

$$ \frac{\delta f}{\delta x} =  2 + 3y $$

`x = 1.0` `y=2.0`의 값을 대입해보면... 

$$ f_{x}'(1.0, 2.0) = 2 + 3(2.0) = 2 + 6 = 8 $$ 

마찬가지로 `y`에 대한 편미분을 해보면

$$ \frac{\delta f}{\delta x} = 3x + 3y^2 $$

$$ f_{y}'(1.0, 2.0) = 3(1) + 3(2)^2 = 3 + 3(4) = 15 $$

수치미분으로 얻은 결괏값이 위 극한에 의해 얻어진 결괏값과 매우 근접한 것을 알 수 있다! 💃 



마지막으로 조금 더 복잡한 `2x2` 행렬을 대입해보자!

우선 `f` 함수는 다시 재정의를 해줘야한다. 왜냐고? 수식이 변했기때문

**변하기 전 수식**: $ f(x, y) = 2x + 3xy + y^3 $

**새로운 수식**: $ f(a, b, c, d) = 2ab + 6a^2bc + 5cd + 2bd^2 $

```python
def g(input_values):
    a = input_values[0, 0]
    b = input_values[0, 1]
    c = input_values[1, 0]
    d = input_values[1, 1]
    
    equation = (2 * a * b) + (6 * np.power(a, 2) * b * c) + (5 * c * d) + (2 * b * np.power(d, 2))
    return equation
```

```python
result = numerical_derivative(g, np.array([[1.0, 2.0],
                                           [3.0, 4.0]]))

print(result)
```

```python
결과값: 
[[76. 52.]
 [32. 47.]]
```

이렇게 구할 수 있다. 



오늘의 기록 끄읏. 👋 
