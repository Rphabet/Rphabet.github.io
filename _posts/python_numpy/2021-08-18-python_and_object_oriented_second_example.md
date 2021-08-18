---
title: 파이썬과 객체지향, 그리고 절차지향의 이해 (2)
date: 2021-08-18 17:03:00 +0900
categories: [python, class]
tags: [object-oriented, python, 파이썬, 프로그래밍, 클래스, class, 생성자] 
math: true
comments: true
typora-root-url: ../
---

---

## 파이썬의 Class (2)

지난 포스팅에서 `class`를 이용해서 학생을 만들어보았다. 어떻게 객체가 만들어지는지, 또 왜 `class` 를 사용해야하는지에 대해 살펴보았다.

오늘은 포스팅의 시작을 `__init__` 이라는 `class`안에서 가장 처음으로 정의된 메서드에 대해 얘기해보고자 한다.

파이썬에서 이친구는 `initializer`라고 불리는데, 통칭적으로 **"생성자"**라고 불린다. 

어떨때 쓰여요?  

아래 예제를 통해 한번 살펴보자.

```python
# 자동차 class를 만들어보자.
class Car(object):
    def set_car_spec(self, brand, model):
        self.brand = brand
        self.model = model
        
    def info(self):
        context = {'brand': self.brand,
                   'model': self.model}
        print(context)
# ***여기서 self는  class 자체를 지칭한다.         

my_car = Car()   # my_car라는 내 자동차 객체를 만들고
my_car.set_car_spec('hyundai', 'genesis')  # 현대의 제네시스라는 차로 입력을 해주자.
my_car.info()    # 이렇게 정보 호출 메서드를 사용한다면....:
# {'brand': 'hyundai', 'model': 'genesis'}
# 브랜드는 현대, 차종은 제네시스 라는 정보를 dictionary 값으로 리턴을 해준다

# 참고로 파이썬 class안의 속성과 함수는 
# 모두 dot operator('.')를 사용하여 호출할 수 있다.
```

오 좋은데? 근데 그래서 뭐???

근데 만약 `Car()` 클래스의 인스턴스 `my_car`에  `set_car_spec` 을 사용하지 않고 `info` 메서드를 호출한다면 어떻게 될까?

한번 살펴보자.

```python
my_car = Car()
my_car.info()
```

```python
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-7-86e27dd64ac5> in <module>
     16 
     17 my_car = Car()
---> 18 my_car.info()

<ipython-input-7-86e27dd64ac5> in info(self)
      6 
      7     def info(self):
----> 8         context = {'brand': self.brand,
      9                    'model': self.model}
     10         print(context)

AttributeError: 'Car' object has no attribute 'brand'
```

`AttributeError`, 즉 오류가 발생한다. 내용을 살펴보면 **Car 라는 객체는 브랜드라는 속성이 없어!!!!**  당연하다. 우리가 정해준적이 애당초 없었으니깐.

코린이로써 프로그래밍을 하면서 객체에 초깃값을 설정해야 할 필요가 있을 때를 많이 마주치진 않았지만, 분명히 프로젝트를 하다보면 설정을 해야하는 순간이 온다. 

그럴때마다 `set_car_spec()` 과 같은 메서드를 호출해서 생성을 해주기는 너무 비효율적일 수 있다.

이때 쓰이는 것이 바로 **생성자** (`__init__`)이다. 자 긴말 필요 없이 코드를 보자.

```python
class Car(object):
    def __init__(self, brand, model):
        self.brand = brand   
        self.model = model
  
    def set_car_spec(self, brand, model):
        self.brand = brand
        self.model = model
        
    def info(self):
        context = {'brand': self.brand,
                   'model': self.model}
        print(context)

my_car = Car('hyundai', 'genesis')   # my_car라는 내 자동차 객체를 만들고
my_car.info()   
# 출력물:  {'brand': 'hyundai', 'model': 'genesis'}

# 초기에 에러없이 값을 줘서 차에 대한 정보를 받아 온 것을 볼 수 있을것이다.

# 이어서 클래스가 효율적인 이유를 하나 더 설명해보겠다.
# 만약에 본인이 돈을 열심히 모아서 차를 바꿨다고 가정을 해보자.
# 어? 뭐야 그럼 차종 업데이트 해줘야지!!! 클래스만의 코드의 유지 보수성이 여기서 
# 빛을 발한다. 
# 일전에 만들어놓은 메서드를 호출하며 정보를 입력해주자.

my_car.set_car_spec('Tesla', 'model S')
my_car.info()
# 출력물: {'brand': 'Tesla', 'model': 'model S'}

# 이렇게 업데이트 된 결괏값을 볼 수 있을것이다.
```

아직까진 모호하다. 음 "**그래서 클래스가 뭐 어쩌라고 어떻게 활용하라고**"!! 🤷‍♂️  싶을 수 있다. 나 또한 그랬고 처음 접하는 대부분의 사람들이 그러리라 생각한다.  필자의 얕은 코딩 지식과 필력으로 설명하기 힘든 부분이 있는 것도 사실이다. 

그럼에도 불구하고 하나 첨언을 하자면 `class`는 함수와 다르게 독립적인 값을 유지할 수 있다는 점이다. 이것 또한 메모리 스택과 heap 영역에 대해 언급하지 않고서는 쉬이 설명할 수 없는 부분이니 지금 당장은 깊게 설명을 하지 않겠다.  

쉽게 얘기하면 

```python
# 두 개의 자동차 객체를 만들었을때
my_car = Car('hyundai', 'genesis')
your_car = Car('Kia', 'k5') 

# 이놈들의 id 값을 찍어보면
print(id(my_car))    # 140571136213200
print(id(your_car))  # 140571136176336   

# 각기 다른 메모리주소에 찍힌 것을 확인할 수 있을것이다.
```

 메모리 주소가 다른것이 굉장히 유의미하다. 왜냐고? 왜냐하면 `my_car`라는 객체를 지지고 볶고 고치고 날리고 난리 부르스를 쳐도 `your_car `객체에 어떠한 영향도 미치지 않기 때문이다.

함수안에 정의되지 않고 뜬금없이, 밑도 끝도 없이 `class` 바로 밑에 변수가 하나 있는 것을 볼 수도 있는데

```python
class Car(object):
    my_variable = 100

def __init__(self, brand, model):
    ...
    ...
    ...
```

이때 저 `my_variable` 이란 친구는 **class variable**이라고 불리는 클래스 변수이다.

Car객체의 모든 인스턴스는 저 친구를 공유하고 있고 따라서 "인스턴스 변수 (instance variable)" 와 다르게 항시 같은 메모리 주소를 갖고 있다는 것을 알아두자.  (`Car()`에서 파생된 모든 인스턴스는 같은 클래스 변수에 한해서 같은 메모리 주소를 갖고 있다. `id()` 함수로 메모리 주소를 출력하면 늘 같은 값을 출력함)

필자가 생각하는 `class` 의 가장 중요한 개념중 하나였던 생성자에 대해서 얘기를 간략하게 해보았다.

그럼 오늘도 수고링! 🙋‍♂️ 
