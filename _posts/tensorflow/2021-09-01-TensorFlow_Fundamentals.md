---
title: TensorFlow 기초
date: 2021-09-03 17:35:00 +0900
categories: [python, tensorflow]
tags: [python, regression, 회귀분석, ML, 머신러닝, tensorflow, 텐서플로우] 
math: true
comments: true
image:
  src: /../assets/images/tf/logo.png
  width: 500
  height: 200
  alt: tensorflow logo
typora-root-url: ../
---

---

# TensorFlow 

**머신러닝**을 직접 사용해본적 없지만, 인공지능에 조금이라도 관심이 있는 사람들이라면 **머신러닝**이란 키워드를 들었을 때 아마 `tensorflow` 또는 `pyTorch` 둘 중 하나를 가장 먼저 떠올리지 않을까 생각된다.

그만큼 너무 유명한 라이브러리이고 유명한만큼 유용한 라이브러리다.

텐서플로우(`tensorflow`)는 `Google`에서 제공한 오픈소스 라이브러리이고, 계산과정과 모델을 **데이터 흐름 그래프(data flow graph)**를 사용하여 표현한다는것이 큰 특징이다.

간단하게 Data Flow Graph에 대해 설명을 해보겠다.

## Data Flow Graph

**구성**: 

- Node: 수학적 연산, 입력, 출력 등등 을 담당
- Tensor: 동적 다중배열(dynamic multidimensional array)  
- Edge: node와 node 를 이어주는 방향성 있는 벡터 선 (텐서를 전달한다)

![tf_node](/../assets/images/tf/tf_node.png)

위 그림은 node를 표현한 것이다. node안에 들어있는 값은 수학적 연산 기호가 들어있거나, 저렇게 문자열이 들어있을 수도 있다. 문자열의 경우는 흔치 않다. 지금은 예제를 위해 노드안에 집어넣어봤다.

![tf_howtowork](/../assets/images/tf/tf_howtowork.png)

데이터 전달 과정은 이렇게 edge에 의해 방향이 결정이 되는데, 이 과정에서 데이터가 흘러다닌다. 그리고 이 흐르는 데이터를 `tensor`라고 일컫는다. (물론 저것이 tensor의 정확한 뜻은 아님)

좀더 구체적으로 살펴보자면

![tf_howtooperate](/../assets/images/tf/tf_howtooperate.png)

Node3은 '더하기 (+)' 연산 기호를 갖고 있고, `node3 = node1 + node2` 이렇게 구성되어 있다. 때문에 node3가 실행이 된다면, node1과 node2도 같이 실행이 된다.

위에서는 node1과 node2의 값이 상수(constant)였다는 점에서 이해하는데 크게 문제가 되진 않았다. 

하지만 데이터 입력은 대게 `variable`이나 `placeholder`를 통해 실행된다. 아래의 그림을 보자

![tf_howtooperate2](/../assets/images/tf/tf_howtooperate2.png)

이번에는 `node1`과 `node2`가 `placeholder`를 들고 있는 것을 볼 수 있으리라. 이때 `node3`을 그냥 실행시킨다면 에러가 발생하는데 node1과 node2의 값이 없기 때문이다. 그래서 실행을 시켜줄때 `feed_dict = {}`라는 인수를 통해 값을 넣어주어야 정상적으로 실행이 가능하다.

---

## 실행

자 지금까지 얘기했던 것들을 실행시켜보자. 

`session`이란 것을 생성시킨 후 실행이 가능한데, 이 `session`은 `data flow graph`가 주어졌을 때, node속 각 연산( `operation`)들을 컴파일하여 실행할 CPU나 GPU에 해당 operation을 전달하여 실행한다.

코드로 살펴보자.

```python
import tensorflow as tf   # aliasing 은 tf로 하자

# node 하나 생성
node = tf.constant('Hello World')

# print로 찍히는걸까?
print(node) # Tensor("Const:0", shape=(), dtype=string)
```

변하지 않는 값 (constant)를 갖고 있는 노드를 하나 만들어보았다. 프린트 노드를로 찍어보면, 노드가 갖고 있는 tensor를 출력할거라 기대를 하지만 그렇게 찍히지는 않고 해당 노드에 대한 설명이 나온다. 

`Tensor("Const:0", shape=(), dtype=string)` 라는게 찍혀 나오는데 간단하게 설명을 해보자면 "tensor는 상수이고 하나의 값을 갖고 있다. 상수는 scalar니깐 shape이 있을수가 없다. 그리고 데이터타입은 스트링이다." 뭐 이런식의 유용한 정보를 알려주기는 한다.

이젠 진짜 그래프 (node)를 실행시켜보자.

```python
sess = tf.Session() # tensor Session 클래스를 실행
sess.run(node) # 노드에 대해서 실행시키라는 run 이라는 클래스의 행위, 즉 메서드를 호출한다.

print(sess.run(node)) # 결과: b'Hello World'
print(sess.run(node).decode()) # 결과: Hello World

```



이번엔 여러개의 노드를 만들어서 사용해보자.

```python
node1 = tf.constant(10, dtype=tf.float32) # 노드에 상수값을 넣어줌. 데이터 타입은 float32
node2 = tf.constant(20, dtype=tf.float32)

node3 = node1 + node2

sess = tf.Session()
print(sess.run(node3))  # 30 ( 10 + 20 = 30 )
```

본 포스팅의 세번째 그림을 보면 이해가 쉽게 되리라 믿는다.



다음 예제는 마지막 그림에서 나왔었던 `placeholder`를 사용해보자.

```python
node1 = tf.placeholder(dtype=tf.float32)
node2 = tf.placeholder(dtype=tf.flaot32)

node3 = node1 + node2
```

이렇게 노드를 만들고 `session`을 통해 그래프를 그리면 어떻게 될까?

```python
sess = tf.Session()
sess.run(node3)
```

에러가 난다. 왜냐고? `node1`과 `node2`의 값이 비어있으니깐. 연산이 안되는거지.

그럼 어떻게 실행하느냐?

마지막 그림에 나왔듯이 `feed_dict`를 사용한다.

```python
print(sess.run(node3, feed_dict={node1: 50, node2: 100}))  
```

```python
그러면 값이
150으로 나온다.
```

