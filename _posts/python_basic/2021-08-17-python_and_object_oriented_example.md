---
title: 파이썬과 객체지향, 그리고 절차지향의 이해 (2)
date: 2021-08-17 15:00:00 +0900
categories: [python, class]
tags: [object-oriented] 
math: true
comments: true
typora-root-url: ../

---

---

## 파이썬의 `class` 

지난 포스팅에서도 언급했다시피 `class`는 객체모델링의 수단이다. (현실세계의 객체를 프로그래밍적으로 묘사하는 방법)

이번 포스팅에선 **학생**이라는 객체를 프로그래밍 적으로 표현을 해보자. 

두가지 관점에 유의하며 `class` 를 만들어보자.

	1. 상태  (인스턴스)
	2. 행위  (메서드)

좀더 구체적으로 묘사하며 **학생**을 떠올려보자. 

학생은 학년이 있을것이며, 학번이 있고, 전공, 학교, 키, 몸무게, 이름, 주소 등등 여러가지 상태를 갖고 있을 것이다. 이러한 상태는 **"변수"**를 만들어줘서 저장을 하게 되는데 이것을 파이썬에서는 **"instance variable (인스턴스)"** 라고 부른다. (자바나 다른 언어에선 다른 방식으로 이런 상태를 부른다.)

그렇다면 행위엔 무엇이 있을까?   학생이 할 수 있는 동작들이다.  예를 들면, "수업을 듣다, 수강 신청을 하다, 등교를 하다" 등과 같은 학생들이 할 수 있는 동작들을 상상하면 된다.  ===> 이 행위들은 **함수**에 저장된다. 이런 객체만의 저장된 함수를 **"메서드 (method)"**라 부른다.

이제 학생을 한 번 만들어보자.

```python
class Student(object):   
  
    def __init__(self, name, dept, num, grade):
        self.name = name
        self.dept = dept
        self.num = num
        self.grade = grade
        
        # name: 이름
        # dept: 학과
        # num: 학번
        # grade: 성적
```

`class`로 정의하고자하는 객체의 첫 letter는 무조건 **대문자**로 써야함을 기억하자. (소문자도 가능함 그렇지만 통상적으로 소문자는 함수, 대문자는 클래스로 사용된다)

처음 `class` 를 만들면 `class`안에 함수가 내재되어 있는 것을 심심치 않게 볼 수 있다. 아까도 말했지만 함수는 객체의 행위를 위해 정의된다. 

필자는 처음 `class`를 접했을 때 "뭐야 근데 함수 이름이 왜저래??? `__init__`???? " 이런 생각을 했었는데 `__init__`은 단순히 `initializer` 라는 친구이며 `class` 사용시 제일먼저 등장하는 함수임을 꼭 인지하자. (추가적인 설명은 다음 포스팅에서 하겠다)

또한 `__init__` 의 인자로 가장 먼저 `self` 가 나오는 것도 필히 알고 있어야한다.

음.. 사실 위 코드를 설명하려면 메모리 구조부터 `stack` 이 어쩌구저쩌구 `heap` 이 어쩌구저쩌구 주저리주저리 해야할 말이 되게 많기는 하다. 또 그런 설명없이 코드풀이를 한다는게 어렵게 느껴진다. 지금 당장은 `self.<variable name>` 은 **인스턴스 변수**임을 인지하고 로컬 변수(local variable)와는 메모리 측면에서 다르다는 것만 어렴풋이 알아두고 넘어가자.

자 그럼 학생 한명을 가볍게 만들어보자. 👨‍🎓 

```python
stu1 = Student('Gildong', 'CS', '12345', 3.5) 
```

자 이걸 어떻게 보면 되겠는가? 이름은 길동, 학과는 컴싸, 학번은 12345, 학점은 3.5이다.

`instance` 하나안에 데이터가 4개 들어있는것임. 스트링 세 개, 실수 하나.
 자료구조측면에선 새로운 자료 하나를 만들어내는 거라 볼 수 있음.

다시 말하자면, `class` 는 자료형 또는 자료구조를 이용하여 새로운 자료형을 만들어내는 것이라고 표현할 수 있겠다. 이런걸 **Abstract Data Type (ADT)**라고 칭한다는 것만 알아두자.

이번에는 `class` 를 조금 더 발전시켜보자.

```python
class Student(object):
  
    def __init__(self, name, dept, num, grade):
        self.name = name   # 이름
        self.dept = dept   # 학과
        self.num = num     # 학번
        self.grade = grade # 성적
        
    def get_student_info(self):      # 마찬가지로 첫번째 인자엔 self 가 들어와야함
        print(self.name, self.dept, self.num, self.grade)
    
    def set_stu_name(self, name):
        self.name = name
   
  
#### 아까와 같이 학생 객체를 만들어보자.

stu1 = Student("Gildong", "CS", "12345", 3.5)

print(stu1.name)   
# '.' 을 이용해서 속성을 출력할수 있다.
# 출력물:  Gildong
                   
stu1.get_student_info()    # class에서 정의한 함수를 호출하면 method를 사용하는 것임.
# 출력물: Gildong CS 12345 3.5 

stu1.set_stu_name('AAA')  # 객체의 이름이 AAA로 바뀐다.
stu1.get_student_info()   # 다시 메서드를 호출
# 출력물: AAA CS 12345 3.5
```

사실 `Class variable` 이라던가 올바르게 함수 사용 방법, 그리고 `class(object)` 의 `object` 최상위 객체에 대한 설명은 따로 하지 않았다. 추후 포스팅을 수정하면서 그리고 다음 포스팅에서 최대한 다뤄볼 수 있는 방향으로 글을 써보겠다.

오늘은 여기까지...



Q. 함수를 호출할 때랑 굉장히 비슷하게 생기지 않았는가? A. 맞다.

Q. 그럼 `class` 는 왜 필요한가?  A. 우리는 `class` 를 기반으로 instance (= object)를 만듦. 우리가 실제 프로그램에서 사용하는 것은 `instance`임

Q. 아니 그러면 함수랑 다를게 없는데 함수가 더 편하지 않나요? 왜 복잡한 class를 써요? 

​	A:

	- 함수 기반보다 class 기반이 더 편함. 효율성이 좋고 메모리 활용도가 더 좋음. 
	- 전 포스팅에서도 말했지만 유지 보수성이 훨씬 좋다.



자 이제 이해를 했다면 꾸준히 `class`를 활용해보자. 익숙해지는 것이 중요한 지금이다.