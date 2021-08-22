---
title: 파이썬 클래스 상속 
date: 2021-08-22 16:20:00 +0900
categories: [python, class]
tags: [python, 파이썬, 클래스, class, 상속] 
math: true
comments: true
typora-root-url: ../
---

---

## 상속 (inheritance)

간단하게 상속에 대해서 알아보자.

상속은 기존 `class` 를 확장하여 멤버를 추가하거나 동작을 변경하는 방법이다.

비슷한 `class`가 있다면 처음부터 다시 만들 필요 없이 상속받아 약간씩 입맛에 맞게끔 확장 및 변형하여 사용하면 된다.

사람을 예로 들어보자.

```python
class Human:
    def __init__(self, age, name):
        self.age = age
        self.name = name

    def intro(self):
        print("{}살 {}입니다.".format(self.age, self.name))

        
kim = Human(29, "김상형") # kim이라는 사람을 만들었음
kim.intro() # 메서드를 불러보자.
# 결과물: 29살 김상형입니다
```

이렇게 사람을 `class`로 표현했다.

이번엔 학생이란 객체를 만들려고 한다.

그런데 학생도 사람이란 범주에 속해있고, 또 사람이면서 더 특수하고 구체적인 속성과 동작을 가질 수 있다. 

이때 사람으로부터 상속을 받으면 더 편하게 학생을 만들 수 있다.

```python
class Student(Human):    # 상속을 받을때는 항상 매개변수로 부모 Class를 사용해야함
    def __init__(self, age, name, stunum):
        super().__init__(age, name)     # super()라는 메서드를 통해서 부모 메서드를 호출할 수 있음.
        self.stunum = stunum   # 학번은 부모 class 에서는 없던 속성이니 새롭게 추가해주자.

		def intro(self):
        super().intro()   # 마찬가지로 부모 메서드를 호출
        print("학번 : {}".format(self.stunum))
        
    def study(self):
        print("얄리얄리 얄라셩 얄라리 얄라")
        
lee = Student(34, "이승범", 1657637)
lee.intro() 
# 34살 이승우입니다
# 학번 : 1657637

lee.study()
# 얄리얄리 얄라셩 얄라리 얄라
```

이렇게 사람이라는 부모를 통해 학생을 만들어보았다. 

위 코드에서 부모 메서드를 호출할 때 `super()` 라는 메서드를 사용하였는데, `super()`를 사용하지 않고 직접 수정을 해줘도 된다. (e.g.)

```python
def __init__(self, age, name, stunum):
    self.age = age
    self.name = name
    self.stunum = stunum  
```

하지만 이렇게 되면 부모의 속성을 변경할 때 자식도 같이 수정해야 하는 불편함이 있다.

객체의 장점이 무엇인가? 유지보수성의 탁월함 아니던가. 저렇게 직접 생성자를 초기화한다면 상속의 장점이 반감되어 코드 관리에 에로사항이 있을 수 있다.

또한 필요하다면 같은 이름의 메서드를 재정의 (override)하여 부모의 동작을 원하는 대로 수정할 수 있다.

```python
def intro(self):
    print("저는 {}학번 {}살 {}라고 합니다.".format(self.stunum, self.age, self.name))
```

이외에도 상속에 대한 특징이 몇가지 더 있다.

- 여러 개의 부모로부터 클래스를 파생시키는 다중 상속이 가능하며, 
- 한 클래스로부터 상속되는 자식 클래스의 개수에는 제한이 없다.

물론 다중 상속은 프로그램을 복잡하게 만드는 부작용이 있어 사용하지 않는 것이 권장된다고 들었다.

오늘은 여기까지.

👋 
