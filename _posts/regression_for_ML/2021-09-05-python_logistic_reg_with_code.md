---
title: python으로 하는 머신러닝 로지스틱 회귀분석
date: 2021-09-05 18:34:00 +0900
categories: [python, regression]
tags: [python, regression, 회귀분석, ML, 머신러닝, tensorflow, 텐서플로우, logistic, 로지스틱] 
math: true
comments: true
typora-root-url: ../


---

---

# Logistic Regression

이번 포스팅에선 로지스틱 회귀분석을 `python`으로, `tensorflow`로, 그리고 `sklearn`으로 직접 구현해보자.

공부시간과 시험 합격에 대한 데이터를 간단하게 직접 만들어서 사용하자.

## 1. python으로 하는 logistic regression

```python
# 환경설정
import numpy as np

# 수치미분 
def numerical_derivative(f, x):

    delta_x = 1e-4   
    derivative_x = np.zeros_like(x)  
    
    it = np.nditer(x, flags=['multi_index'])
    
    while not it.finished:
        idx = it.multi_index
        tmp = x[idx]   
        
        x[idx] = tmp + delta_x 
        fx_plus_delta_x = f(x) 
        
        x[idx] = tmp - delta_x 
        fx_minus_delta_x = f(x)
        
        derivative_x[idx] = (fx_plus_delta_x - fx_minus_delta_x) / (2 * delta_x)
        
        x[idx] = tmp         
        it.iternext() 
        
    return derivative_x
```

```python
# training data set 준비
# 그냥 내 마음대로 데이터 값을 주었다.

x_data = np.arange(2, 21, 2).reshape(-1, 1) # 공부 시간
t_data = np.array([0, 0, 0, 0, 0, 0, 1, 1, 1, 1]).reshape(-1, 1)  # 0은 불합격, 1은 합격
```

위 데이터에 따르면 공부시간이 12시간일 때는 시험에 불합격하지만, 14시간이라면 합격을 한다. 그렇다면 실측 데이터가 없는 13시간일 땐 어떻게 되는걸까? 그걸 알아보자!

```python
# Weight and bias 설정
W = np.random.rand(1, 1)  # 1x1 2차원 행렬로 설정
b = np.random.rand(1) 

#### 각종 함수 정의
# 예측함수 정의
def logistic_predict(x):
    
    z = np.dot(x, W) + b  # linear
    y = 1 / (1 + np.exp(-1 * z))  # sigmoid 함수
    
    result = 0
    if z >= 0.5:
        result = 1
    else:
        result = 0

    return result, y        # 0과 1 결과값 뿐만 아니고 확률 y 까지 같이 리턴한다.

# loss 함수 정의
def loss_func(input_value):

    input_w = input_value[0].reshape(-1, 1)
    input_b = input_value[1]
    
    z = np.dot(x_data, input_w) + input_b
    y = 1 / (1 + np.exp(-1 * z))
    
    delta = 1e-7   # 혹여나 y의 값이 1이 되는것을 방지하기 위해서
    
    return -np.sum( (t_data * np.log(y + delta)) + ((1 - t_data) * np.log(1 - y + delta)) )
    # return 값은 cross entropy 
    
# learning rate 정의
learning_rate = 1e-4
```

이제 설정은 다 끝났다! 반복학습 고고!

```python
for step in range(300000):
    input_param = np.concatenate(W.ravel(), b.ravel(), axis=0)  # [w의 값, b의 값]
    result_derivative = learning_rate * numerical_derivative(loss_func, input_param)
    
    # update W and b
    W = W - result_derivative[0].reshape(-1, 1)
    b = b - result_derivative[1]
    
    if step % 30000 == 0:
        print("loss: {}".format(loss_func(input_param)))
```

```python
결괏값:
loss값은 37.363400822887364
loss값은 2.8965998492103546
loss값은 2.1229618648269253
loss값은 1.7758889573534715
loss값은 1.5682037282670798
loss값은 1.4256076774651414
loss값은 1.319509267965889
loss값은 1.236302240833145
loss값은 1.1685835845187729
loss값은 1.1119343375856396
```

썩 낮은 loss 값은 아니다. 0에 그리 가깝지 않기 때문이다. 이런 결과가 도출된 까닭은 데이터 양이 매우 적기 때문일 확률이 크다. 

아무튼 지금은 넘어가도록 하자.

이제 예측을 해보자.

```python
result = logistic_predict([[13]])
print(result)
"""
출력물:
(1, array([[0.54444063]]))

1은 시험에 합격을 예측했다는 것이다. 뒤에 온 실수 0.54444 는 시험에 합격할 확률을 나타낸다.
"""
```

---

## 2. SciKit Learn으로 구현

지난번 포스팅을 보았다면 짐작하겠지만, `sklearn`으로 구현하는건 엄청 간단하고 편리하고 빠르다!

```python
from sklearn import linear_model

model = linear_model.LogisticRegression()   # 로지스틱 회귀분석 모형 객체를 하나 만들어준다.

model.fit(x_data, t_data.ravel())   # sklearn에서 로지스틱을 할 땐 t_data를 1차원으로 풀어쓰자
result = model.predict([[13]])

print(result)

result_proba = model.predict_proba([[13]])
print(result_proba)
```

```python
result 결과값:   0      sklearn은 불합격하리라 예측했다.
result_proba 결과값: [[0.50009391 0.49990609]]
  
  0일 확률이 약 50.009 퍼센트
  1일 확률이 약 49.991 퍼센트이다.
```

도깨비 방망이로 뚝딱! 한 것처럼 엄청 빠르고 간결하게 결과가 나왔다.

텐서플로우는 어떨까?

---

## 3. tensorflow로 구현

```python
import numpy as np
import tensorflow as tf

# 데이터를 입력받기 위한 노드를 만들고 시작.
X = tf.placeholder(shape=[None, 1], dtype=tf.flaot(32))
T = tf.placeholder(shape=[None, 1], dtype=tf.float(32))

# Weight and bias 설정
W = tf.Variable(tf.random.normal([1,1]), name='weight')  # 초기값을 랜덤하게 하나 받음
b = tf.Variable(tf.random.normal([1]), name='bias')  # 초기값을 랜덤하게 하나 받음

# hypothesis 설정
logit = tf.matmul(X, W) + b  
H = tf.sigmoid(logit)  # tf는 sigmoid 함수를 직접 제공한다. 따라서 따로 수식을 잡아주지 않아도 된다.

# loss function
loss = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(logits=logit, labels=T))
# 마찬가지로 loss 함수에 대해서도 따로 cross entropy를 주는 메서드가 있다.

# train node 설정
train = tf.train.GradientDescentOptimizer(learning_rate=1e-4).minimize(loss)
   # w와 b를 최적화할건데, gradient descent를 이용해서 loss 값을 최소화할거다.
# graph를 실행시켜줄 session이 필요.
sess = tf.Session()
sess.run(tf.global_variables_initializer())   # 그래프 실행전 반드시 해야하는 초기화

# 반복학습 고고
for step in range(300000):
    tmp, loss_val = sess.run([train, loss], feed_dict={X: x_data, T: t_data})
    
    if step % 30000 == 0:
        print("loss: {}".format(loss_val))
```

```python
결과값:
loss: 5.002998352050781
loss: 0.46201056241989136
loss: 0.4189934730529785
loss: 0.3852218985557556
loss: 0.3580852746963501
loss: 0.3357999920845032
loss: 0.3171757757663727
loss: 0.3013768792152405
loss: 0.2877642512321472
loss: 0.275882750749588
```

이렇게 쭉쭉 떨어지는 loss값을 볼수 있으리라. 다시한 번 말하지만, loss값이 결코 낮은게 아니다. 더 0에 가까워야한다.

높게 나온 이유는 뭐라고? 데이터 값이 터무니없이 적기 때문일것이다.

자 그럼 마지막으로 예측을 해보자

```python
result = sess.run(H, feed_dict={X: 13})

print(result)
```

```python
결과값:
[[0.57707125]]   ==> 합격!!!
```

이렇게 결과값이 도출되었다.

오늘 다룬 주제같은 경우에는 따로 데이터 값이 많지 않았기때문에 loss 값도 높게 나왔고, `sklearn`, `python`, 그리고 `tensorflow`간의 예측값이 크게 달랐다. 

양질의 데이터를 갖고 있다면 분명 일치하는 결과가 나왔으리라 생각한다.

아무튼 오늘은 여기까지..

👋 