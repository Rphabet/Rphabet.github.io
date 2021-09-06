---
title: python으로 하는 머신러닝 다변수 로지스틱 회귀분석 
date: 2021-09-06 15:18:00 +0900
categories: [python, regression]
tags: [python, regression, 회귀분석, ML, 머신러닝, tensorflow, 텐서플로우, logistic, 로지스틱] 
math: true
comments: true
typora-root-url: ../
---

---

# Multivariate Logistic Regression

지난번 단일변수를 이용한 로지스틱 회귀분석 포스팅을 올린 뒤, 그대로 다른 영역에 대해 소개하려 했으나 연습겸 한번 더 로지스틱을 다루는 컨텐츠를 올리기로 마음을 먹었다. 

이번엔 `python`, `sklearn`, 그리고 `tensorflow`를 이용해 각기 다른 3번의 회귀분석을 해보도록 하자. 

대학원 입학에 관련된 데이터를 다룰것이며, 레이블은 합격/불합격으로 합격일 때 1의 값을, 불합격일 때 0의 값을 취하는 admission 데이터를 사용할 예정이다. 

데이터는 추후 github에 올려두겠다.

## 환경설정

```python
import numpy as np
import pandas as pd
from scipy import stats # 이상치 처리 시 사용될 zscore
from sklearn import linear_model  # sklearn의 logistic regression
import tensorflow as tf
from sklearn.preprocessing import MinMaxScaler  # 정규화 때 사용될 min max scaler
```

## Data Preprocessing

### 데이터 로딩. 

```python
df = pd.read_csv('./data/admission/admission.csv', sep=',')
display(df)
```

### 결치값 처리

```python
training_data = df.dropna(how='any')  # NaN이 있는 행 무조건 삭제
```

### 이상치 처리

(이상치를 처리 판단에 도움을 주는 산점도나 박스 그래프는 우선은 넘어가겠다.)

```python
# zscore를 꽤나 관대하게 잡아보았다. 약 98%의 샘플을 사용.
zscore_threshold = 2.0

# gpa에 있는 이상치 처리
b_mask = np.abs(stats.zscore(df['gpa'])) > zscore_threshold
outlier1 = df['gpa'][ b_mask ]

# gre에 있는 이상치 처리
b_mask2 = np.abs(stats.zscore(df['gre'])) > zscore_threshold
outlier2 = df['gre'][ b_mask2 ]

li = []
li.append(outlier1.index.values)
li2 = []
li2.append(outlier2.index.values)
result = []

for idx in li[0]:
    result.append(idx)
for idx in li2[0]:
    result.append(idx)
print(result)
outliers = result

# pandas의 iloc을 사용하여 outlier값이 없는 행들로만 data frame을 구성
clean_data = df.iloc[ ~df.index.isin(outliers) ,:]

display(clean_data)   
```

<img src="/../assets/images/regression/clean_df.png" alt="clean_df" style="zoom:50%;" />

- `admit`: 1은 합격, 0은 불합격   (`admit`은 레이블로 쓰임)
- `gre`: 대학원 입학시험 `GRE` 점수
- `gpa`: 학부 학점 평균
- `rank`: 지원하는 학교의 순위

### 정규화 (Normalization)

정규화는 `python`이나 `tensorflow`로 작업할 때 필수적으로 사용됨을 알아두자.

`sklearn`같은 경우에는 자체적으로 정규화를 해주기때문에 굳이 상관이 없다.

```python
scaler_x = MinMaxScaler()
scaler_x.fit(clean_data.iloc[:, 1:].values)

new_df = clean_data.iloc[:, :].copy()
new_df.iloc[:, 1:] = scaler_x.transform(new_df.iloc[:, 1:])
```

label인 `admit`의 값은 어차피 0, 1 둘중하나 또 스케일 자체가 0~1 사이에 있는 값을 취하기때문에 굳이 정규화 작업을 해줄 필요는 없다.

### 최종 데이터 셋 준비

```python
# 독립변수 데이터, 레이블 데이터 따로 분리
x_data = new_df.iloc[:, 1:]   # n x 3 2차원 행렬
t_data = new_df.iloc[:, 0].reshape(-1, 1)   # n x 1  2차원 행렬로 변환

# 가중치와 bias 난수로 차원 맞춰서 설정
W = np.random.rand(3, 1)
b = np.random.rand(1)
```

## 1. Using Python

```python
# 수치미분 함수가 있어야 함
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
# 파이썬으로 쓸 loss function과 예측 함수를 정의하자.
# logistic 예측함수
def logistic_predict(x):
    z = np.dot(x, W) + b
    y = 1 / (1 + np.exp(-1 * z))
    
    result = 0
    if y < 0.5:
        result = 0
    else: 
        result = 1

    return result, y

# loss function
def loss_func(input_value):
    
    input_w = input_value[:-1].reshape(-1,1)
    input_b = input_value[-1]
    
    z = np.dot(x_data, input_w) + input_b
    
    y = 1 / (1 + np.exp(-1 * z))
    
    delta = 1e-7
    
    return -np.sum( (t_data * np.log(y + delta)) + ((1 - t_data) * np.log(1 - y + delta)) )

# 상수 설정
learning_rate = 1e-4
```

(따로 loss 함수와 logistic의 sigmoid에 대해서 수식적인 부분은 적지 않았다. 그건 전 포스팅 [로지스틱 회귀](https://rphabet.github.io/posts/python_logistic_regression/)를 참고하자.)

```python
# 반복학습 고고
for step in range(300000):
    
    input_param = np.concatenate((W.ravel(), b.ravel()), axis=0)
    result_derivative = learning_rate * numerical_derivative(loss_func, input_param)

    W = W - result_derivative[:-1].reshape(-1, 1)
    b = b - result_derivative[-1]
    
    if step % 30000 == 0:
        print("loss값은 {}".format(W, b, loss_func(input_param)))
```

```python
loss값은 430.97272711844545
loss값은 1.6738163473583776
loss값은 0.8337907472538344
loss값은 0.5551179253193748
loss값은 0.4160607461242618
loss값은 0.3327186321044007
loss값은 0.27719575662511164
loss값은 0.2375552132020219
loss값은 0.2078347927438515
loss값은 0.18472466508814028
```

이제 예측을 해보자.

```python
scaled_predict_data = scaler_x.transform([[600, 3.8, 1]])   # 예측값에 대해 정규화 실시          
print(scaled_predict_data)
print(scaler_x.data_max_)

scaled_result = logistic_predict(scaled_predict_data)
print(scaled_result) 
```

이렇게 마지막 출력문을 찍으면 0과 1의 레이블 결과값과 logistic확률 값이 `tuple`로 반환되어 찍힌다! 

---

## 2. tensorflow

이번엔 tensorflow로 실행해보자.

```python
# 데이터 입력받기 위한 placeholder node 설정
X = tf.placeholder(shape=[None, 3], dtype=tf.float32)   # 독립변수 3개 쓰임. n x 3 행렬
T = tf.placeholder(shape=[None, 1], dtype=tf.float32)   # 종속변수는 어차피 하나밖에 없음

# W and b 설정
W = tf.Variable(tf.random.normal([3, 1]))  # 난수를 3x1 행렬로 생성
b = tf.Variable(tf.random.normal([1])) 

# Hypothesis 노드 생성
logit = tf.matmul(X, W) + b
H = tf.sigmoid(logit)

# loss 함수 노드 생성
loss = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(logits=logit, labels=T))

# 학습 train 노드 생성
train = tf.train.GradientDescentOptimizer(learning_rate=1e-3).minimize(loss)

# session 객체 하나 잡아주자.
sess = tf.Session()
# session 변수 초기화
sess.run(tf.global_variables_initializer())

# 반복학습 고고
for step in range(300000):
    tmp, loss_val = sess.run([train, loss], feed_dict={X: x_data, T: t_data})
    
    if step % 30000 == 0:
        print('loss is {}'.format(loss_val))
```

```python
loss is 0.8436719179153442
loss is 0.6416785717010498
loss is 0.6002219915390015
loss is 0.5884969830513
loss is 0.5846560597419739
loss is 0.5832125544548035
loss is 0.5826002359390259
loss is 0.582312285900116
loss is 0.5821653604507446
loss is 0.582085371017456
```

예측을 해보자.

```python
# tensorflow prediction

scaled_predict_data = scaler_x.transform([[600, 3.8, 1]])

result = sess.run(H, feed_dict={X:scaled_predict_data})  # 확률값을 알려줌. 이 확률값을 기반으로 0, 1 threshold를 다시 정해줘야함
print(result)  # [[0.564846]]   ==> 합격!!! 
```

---

## 3. `sklearn`으로 구현

`sklearn`으로는 매우 간단하게 구현할 수 있다.

```python
# LogisticRegression 객체 설정
model = linear_model.LogisticRegression()

model.fit(x_data, t_data.ravel())

scaled_predict_data = scaler_x.transform([[600, 3.8, 1]])

result = model.predict(scaled_predict_data)
print(result)   # [1]

result_proba = model.predict_proba(scaled_predict_data)
print(result_proba)  # [[0.45454108 0.54545892]]
```

`sklearn`은 이렇게 약 54.5%의 확률값과 함께 합격한다고 예측하였다.



이번 포스팅에선 각기 다른 패키지를 이용해서 로지스틱 회귀분석을 사용하여 ML을 구현해보았다. 

데이터가 빈약한 데이터 셋과 예제를 위해 건너뛴 많은 부분이 있다. 그렇기 때문에 다소 빈약한 결과가 나온거 같다.

다른 예제에선 양질의 데이터를 활용하여, 좀 완성도 높은 결괏값을 도출해보겠다.

그럼 끄읏. 👋 
