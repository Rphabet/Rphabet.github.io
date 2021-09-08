---
title: python으로 하는 머신러닝 로지스틱 회귀분석 예제 (tensorflow version)
date: 2021-09-08 20:00:00 +0900
categories: [python, regression]
tags: [python, regression, 회귀분석, ML, 머신러닝, tensorflow, 텐서플로우, logistic, 로지스틱] 
math: true
comments: true
typora-root-url: ../
---

---

# TensorFlow를 이용한 Logistic Regression 예제

바로 직전 포스팅에선 `sklearn`을 이용해서 Logistic Regression 모델을 만들고, validation dataset을 예측한 뒤 k-fold cross validation을 통해 모델의 분류성능을 평가해보았다.

이번엔 `tensorflow`를 이용하여 유방암 데이터를 분석해보자.

## Environment Setting

```python
import numpy as np
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler  # 정규화 진행할것임.
import tensorflow as tf
import pandas as pd
```

## Raw Data Loading

```python
cancer = load_breast_cancer()
print(cancer.DESCR)
print(cancer.data) # feature 값 확인
print(cancer.target) # label값 확인
```

Feature: `shape=(569, 30)`

## Standardization (정규화)

지난 포스팅에서 정규화는 따로 하진 않았었다. 왜냐하면 `sklearn`을 돌릴 때 자체적으로 해주기 때문이다.

지금은 tensorflow로 모델을 만들고 돌릴거니깐.. 꼭 해주는 게 좋다. 

하지만 이상치 처리는 일단은(?) 스킵하겠다... 너무 많음..ㅎㅎ

```python
df = pd.DataFrame(cancer.data)
display(df)
```

![cancer_df](/../assets/images/regression/cancer_df.png)

보다시피 스케일이 제각각이다. 각 컬럼별로 최소값과 최대값의 차이가 많이 나는 컬럼도 많기도 하거니와, 0~1사이의 값으로 맞춰주는게 회귀분석시 비교적 덜 편향된 결과를 도출하기 때문이다.

```python
x_data = cancer.data
t_data = cancer.target

# scaler 설정
scaler = StandardScaler()

scaler.fit(x_data)

# train and validation dataset
train_x_data, valid_x_data, train_t_data, valid_t_data = \
train_test_split(x_data, t_data, test_size=0.2, stratify=0.2, random_state=2)

# 정규화
train_x_data_norm = scaler.transform(train_x_data)
valid_x_data_norm = scaler.transform(valid_x_data)

display(pd.DataFrame(train_x_data_norm))

# 기존 train, valid _x_data 삭제!
del train_x_data
del valid_x_data
```

자세히 보면, `t`데이터에 관해서는 따로 정규화를 진행하지 않았다. 어차피 0과 1사이에서 값이 떨어지기 때문이다. 

## TensorFlow 구현

```python
X = tf.placeholder(shape=[None, 30], dtype=tf.float32)
T = tf.placeholder(shape=[None, 1], dtype=tf.float32)

# W and bias:
W = tf.Variable(tf.random.normal([30, 1]))  
b = tf.Variable(tf.random.normal([1]))

# Hypothesis
logit = tf.matmul(X, W) + b
H = tf.sigmoid(logit)

# loss function (cross entropy)
loss = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(logits=logit, labels=T))

# train
train = tf.train.GradientDescentOptimizer(learning_rate=1e-4).minimize(loss)

# Session 설정 및 variable 초기화
sess = tf.Session()
sess.run(global_variables_initializer())

# 반복학습
for step in range(300000):
    tmp, loss_val = sess.run([train, loss], feed_dict={X: train_x_data_norm, T: train_t_data})
    
    if step % 30000 == 0:
        print('loss: {}'.format(loss_val))
```

이렇게 구현하고 엔터를 빵! 치면 반복학습이 진행된다. 

지난번 포스팅에서 다뤘기도 했기도 했고 특별한 부분도 없기 때문에 그래프 그리는 코드에 대해서 설명을 따로 하진 않겠다.

나같은 경우엔  `loss: 0.09046346694231033` 로 값이 떨어졋다.

## 정확도 (Accuracy) 측정

학습이 종료되면, 얼마나 잘 만들어진 모델인지 확인해볼 필요가 있다.

분류성능평가를 `metric`으로 진행하자.

```python
# 위에서 정의해놓은 H 노드로 predict이라는 예측 노드를 하나 만들자.
predict = tf.cast(H > 0.5, dtype=tf.float32)
"""
인자 해석:
H가 0.5보다 크면, 값을 True로 빼는데 data type은 실수로 줘라.
즉 H가 0.5보다 크면 값이 1, 그렇지 않다면 값이 0인 셈이다.

tf.cast라는 텐서플로우 메서드는 데이터 타입을 바꿔주는 역할을 함
"""
```

이제 우리가 만든 `predict` 노드를 실측 데이터 `T`와 비교해보자.

```python
correct = tf.equal(predict, T)
"""
예를 들면 predict = [0, 0, 1, 1, 1, 0, 1]   << 이게 프레딕트의 결과값
              T = [0, 0, 1, 0, 1, 0, 0]
        correct = [T, T, T, F, T, T, F]
   correct_val = [1, 1, 1, 0, 1, 1, 0]
   correct_val평균 = 5/7  ~~>  71% accuracy임 
"""
```

```python
accuracy = tf.reduce_mean(tf.cast(correct, dtype=tf.float32))
acc_val = sess.run(accuracy, feed_dict={X: valid_x_data_norm, T: valid_t_data.reshape(-1, 1)})
# 정확도 값은..
print(acc_val)  # 0.98245615
```

정확도 값이 98.2%로 측정되었다! 

참고로 정규화 전에는 92%였음. 이만큼 정규화가 중요하다.

아무튼 오늘은 여기까지...

👋 