---
title: python으로 하는 수치미분 (Numerical Differentiation)
date: 2021-08-25 17:30:00 +0900
categories: [python, mathematics]
tags: [python, differentiation, numerical] 
math: true
comments: true
typora-root-url: ../
---

---

## 미분 (Differentiation)

"사실 미분은 크게 두 가지 종류가 있어요~"라고 말하며 **해석미분**은 무엇이고 **수치미분**은 무엇인지 장황한 설명을 시작으로 글을 쓰고 싶으나, 간단하게 수치미분이 무엇인지, 또 파이썬으로 하려면 필요한게 무엇인지만 설명한 뒤 바로 코드로 들어가자! 

**수치미분**은 해석 미분을 수행할 수 없을 때, 정답은 아니지만 숫자를 입력해서 근사값을 구하는 방법이다.

수치미분을 하기 위한 세 가지의 방법이 존재한다. (응 그렇고나 하고 넘어가자)

- **전향차분**: $\lim_{\Delta x \to 0} \frac{f(x + \Delta x) - f(x)}{\Delta x}$ 
- **후향차분**: $\lim_{\Delta x \to 0} \frac{f(x) - f(x - \Delta x)}{\Delta x}$ 
- **중앙차분**:  $\lim_{\Delta x \to 0} \frac{f(x + \Delta x) - f(x - \Delta x)}{2\Delta x}$ 

보통 제일 많이 쓰이고, 결괏값이 가장 정확한 것은 맨 마지막에 말한 "**중앙차분 (Centered Difference)**"이다.

아래의 그래프에 대한 설명은 지금 당장은 따로 하지 않겠다 (추후 수정을 통해 업데이트 하겠음!). 다만 이해에 도움이 되길 바랄 뿐이다.

![differentiation](/../assets/images/math/differentiation.png)



바로 아래의 코드를 살펴보자.

```python
def numerical_differentiation(f, x, method='central'):
  """
  첫번째 인자 f는 미분하려고 하는 함수
  두번째 인자 x는 x의 값 
  세번째 인자 method = 'central'  중앙차분이 디폴트.
                     'forward' 전방차분
                     'backward' 후방차분
  """
  delta_x = 1e-4  # 극한값을 고려. 델타x는 0에 최대한 가까워야함. 여기선 1 * 10^(-4)를 사용
  
  if method == 'central':
    result = (f(x + delta_x) - f(x - delta_x)) / (2 * delta_x)
  elif method == 'forward':
    result = (f(x + delta_x) - f(x)) / delta_x
  elif method == 'backward':
    result = (f(x) - f(x - delta_x)) / delta_x
  else:
    raise ValueError("Method must be either 'central', 'forward', or 'backward'")
  return result
```

이렇게 수치미분에 대한 함수를 정의했다. 

정의했다시피 `f`는 미분하려고 하는 함수이다. 

파이썬은 1급 함수를 지원하기때문에, 함수를 다른 함수에 인자로 넣어줄 수가 있다. 

한번 사용해보자.

```python
# f(x) = x^2   함수를 정의
def squared(x):
  return x ** 2

print("중앙차분의 값은 {} 입니다.".format(numerical_differentiation(squared, 5)))
print("전향차분의 값은 {} 입니다.".format(numerical_differentiation(squared, 5, 'forward')))
print("후향차분의 값은 {} 입니다.".format(numerical_differentiation(squared, 5, 'backward')))
```

```python
# 이렇게 리턴되는 것을 볼 수 있다.
중앙 차분의 값은 9.999999999976694 입니다.
전향 차분의 값은 10.000099999984968 입니다.
후향 차분의 값은 9.99989999996842 입니다.
```

실제 계산을 해본다면

정답은: $ f'(x) = \frac{df}{dx} = 2x $ 

​           $ f'(5) = 10 $ 

정답에 아주 가까운 근사치를 리턴받을 수 있다.

오늘은 여기까지.

끄읏트 👋 

