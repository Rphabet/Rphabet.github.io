---
title: Kaggle Machine Learning Titanic
date: 2021-09-09 18:03:00 +0900
categories: [python, ML]
tags: [python, ML, 머신러닝, tensorflow, 텐서플로우, kaggle, 캐글] 
math: true
comments: true
typora-root-url: ../

---

---

# Titanic Machine Learning from Disater 

처음 머신러닝을 시작하고, [Kaggle](https://www.kaggle.com/)에서 데이터를 뒤적뒤적, competition을 기웃거리다보면 빈번하게 접하는 데이터가 바로 이 `titanic.csv`이지 않나 생각한다.

실제 타이타닉에 승선한 탑승객들에 대한 데이터이며, 각종 정보(**features**)를 갖고 탑승객이 사망하였는지, 살았는지를 기계학습을 통해 예측하는 로직을 구현할 수 있다.

데이터는 [여기](https://www.kaggle.com/c/titanic/data)에서 다운받자.

---

## Logistic Regression Label

`label`값은 `0` 아니면 `1`이다.

- `0`: 해당 승객이 죽었다.
- `1`: 해당 승객이 살았다.  

---

## Environment Setting

```python
import numpy as np   # 행렬 곱연산
import pandas as pd  # csv 읽기 및 dataframe 을 통한 데이터 핸들링
import matplotlib.pyplot as plt # EDA for outliers
import tensorflow as tf  # tensorflow 기계학습
from scipy import stats  # zscore
from sklearn.model_selection import train_test_split  # train and validation data set split
```

## Raw Data Loading

```python
df = pd.read_csv('./data/titanic/train.csv')  # train 데이터 세트를 사용할 것임.
display(df)
```

![titanic_df](/../assets/images/regression/titanic_df.png)

`891 x 12`의 데이터프레임을 볼 수가 있다. 

독립변수 즉 컬럼에 대한 아래에서 간단하게 설명을 하겠다. 자세한 건 해당 데이터가 있는 kaggle 사이트에서 data dictionary를 읽어보길 바란다.  

## Feature Engineering

사실 이부분이 가장 중요한 부분중 하나인데, 어떤 **feature**를 사용해야하는가는 정답이 없기 때문이다. 

하지만 논리적으로 확실하게 왜 저 컬럼을 배제했는지 근거에 관해서 설명을 해야하며, 감으로 그냥~ 😏  이런식의 태도는 ML 또는 계량분야에 있어서는 용납이 되지 않는다는 것을 알아두자.

살았는지, 죽었는지에 대해 영향을 주지 않는 컬럼을 제외해보자.

- `PaseengerId`: 아이디가 낮은 숫자이면 살고, 높은 숫자이면 죽는가? 아니다. 전혀 연관이 없다 과감하게 배제하자.
- `Pclass`: 이건 좌석 등급이다. 1등석 2등석 3등석 정도로 해석하면 된다. 등급에 따라 살 확률? 음 있을거 같다. 높은 등급일 수록 배 갑판 부분에 위치해 있고, 낮은 등급이라면 배 밑부분에 위치해있기 때문이다.
- `Name`: 얼핏보면 이름이랑 삶과 죽음이랑 무슨 상관이 있겠는가 싶다. 하지만 이름안에 있는 타이틀은 결측치나 이상치 처리시에 요긴하게 쓰일 수 있을거 같다. `Mr.`, `Mrs`, `Miss`, `Dr`, etc.   결측치가 발견하면 타이틀 그룹으로 묶어서 그룹의 평균값을 넣으면 나름 괜찮은 로직에 의거해서 결측치 처리를 했다고 할 수 있을거 같다. 🧐 
- `Sex `: 중요하다. 실제로 여자가 더 많이 살기도 했고.
- `SibSp`: 같이 탄 형제 자매 유무 -> 살린다
- `Parch`: 같이 탄 부모님 혹은 자녀 유무 -> 살린다
- `Ticket`: 제거하자. 티켓 번호에 따라 사람이 사나? 뭐 심오하게 들어가서 티켓 번호를 통해 클래스를 알아낸다면 그럴수 있다. 하지만 우린 이미 `Pclass`라는 걸출한 feature를 갖고 있다.
- `Fare`: "요금은 당연히 살려야지~"라고 생각할 수 있겠으나, 우린 이미 `Pclass`를 갖고 있다. 너무 비슷한 성격의 독립변수를 두개다 갖고 있다면 `overfitting`의 문제점이 야기될 수 있다. "어떤게 더 좋은건가요?" 이 질문을 갖고 데이터의 연관성을 찾아보고 하나를 제거하도록 하자. 여기선 `fare` 를 제거한다.
- `Cabin`: 케빈도 제거한다.
- `Embarked` : 어디서 탔는가를 알려주는 변수이다. 이게 과연 중요할까? 중요하다. 지역 편차가 있을수 있기 때문. 특정 항구에서 부자가 더 많이 탔을 수도 있다.

```python
# 이름에서 타이틀 추출 
df['Name'] = df['Name'].apply(lambda x: x.split(',')[1].strip())
df['Title'] = df['Name'].apply(lambda x: x.split('.')[0].strip())

# strip()은 문자열 맨 앞과 맨 뒤에 나오는 화이트스페이스 (띄어쓰기 ' ', 탭 '\t', 엔터'\n')를 제거해줌
```

이렇게 `Title`을 알려주는 컬럼을 하나 따로 만들었다!

이제 필요없는 컬럼을 제거하자.

```python
train_df = df.drop(['PassengerId', 'Ticket', 'Fare', 'Name', 'Cabin'], axis=1, inplace=False)
```

그리고 `SibSp`와 `Parch`를 하나로 묶으면 어떨까? 음.. 변수는 `Family`로 잡아주자.

```python
train_df['Family'] = train_df['SibSp'] + train_df['Parch']  # 뉴 컬럼 만들고 
train_df.drop(['SibSp', 'Parch'], axis=1, inplace=True)   # 기존 컬럼 삭제
```

이젠 기존 문자열로 표기된 컬럼의 값들을 숫자로 치환하자. 

```python
# 성별 0/1 분류
gender_mapping = {'female': 1, 'male': 0}
train_df['Sex'] = train_df['Sex'].map(gender_mapping) 
# 맵 함수를 이용해서 여자면 1의 값을, 남자면 0의 값을 주자.
```

```python
# 항구 마크 숫자로 치환
# 바로 항구를 숫자로 치환하려고 보니깐, NaN 값이 있다. 
# 결치값 먼저 매꿔주자. 어떻게? 최빈값으로 넣어주자.

train_df['Embarked'].value_counts()
"""
S    644    << 최빈값
C    168
Q     77
Name: Embarked, dtype: int64
"""
train_df['Embarked'] = train_df['Embarked'].fillna('S')

# 항구 마크 숫자로 치환
location_mapping = {'S':0, 'C': 1, 'Q': 2}

train_df['Embarked'] = train_df['Embarked'].map(location_mapp
```

## Data Handling

결치값 처리하자. 

`Embarked` 칼럼 같은 경우에는 위에서 먼저 결치값을 처리하였다.

다음 어떤 컬럼에 결치값이 있나 확인해보았더니, `age`컬럼에서 결치값을 발견할 수 있었다.  

```python
# 결치값 컬럼 확인
print(train_df.isna().sum(axis=0))  # 행방향으로 na 값 카운트 

"""
Survived      0
Pclass        0
Sex           0
Age         177
Embarked      0
Title         0
Family        0
dtype: int64
"""
```

"아 `Age`밖에 `NaN`값을 가진 컬럼이 없구나~" 근데 되게 많네? 삭제할 순 없잖아? 값을 대체해줘야지!

아까전에 따로 빼놓았던 `title`을 활용하자. 그래서 집계 함수 이용해서 `title`별로 평균값 구하고 넣어주면 되지 않을까? 각 집단의 특성을 잘 살린만큼 로직도 뒷받침된다!

바로 ㄱㄱ🚗 

```python
# 나는 title 카테고라이징을 괜히 한번 해주었다.

i = 0
title_mapping = {}
for idx in train_df['Title'].value_counts().index:
    title_mapping[idx] = i
    i += 1
    
print(title_mapping)

train_df['Title'] = train_df['Title'].map(title_mapping)
"""
{'Mr': 0, 'Miss': 1, 'Mrs': 2, 'Master': 3, 'Dr': 4, 'Rev': 5, 'Mlle': 6, 'Major': 7, 'Col': 8, 'the Countess': 9, 'Capt': 10, 'Ms': 11, 'Sir': 12, 'Lady': 13, 'Mme': 14, 'Don': 15, 'Jonkheer': 16}
"""
```

타이틀이 많기도 해라.

```python
# age 결치값 처리

# title별 나이 평균값 연산
age_group_mean = train_df.groupby('Title')['Age'].mean()
print(age_group_mean)

# nan값만 찾아서 boolean mask로 사용!
nan_mask = train_df['Age'].isnull()
# print(nan_mask)
display(train_df.loc[nan_mask, :])

# 결치값이 있는 Index 별로 찾아서 그룹민을 값으로 넣어준다.
i_list = train_df.loc[nan_mask, :].index
print(i_list)
for i in i_list:
    idx = train_df.loc[nan_mask, 'Title'][i]   
    # nan값이 있는 컬럼에 직접 가서 타이틀 값을 idx로 가져온다
    train_df.loc[nan_mask, 'Age'] = age_group_mean[idx]  
    # 원본 데이터에 nan 값이 있는 행에 직접 액세스해서 가져온 타이틀 값을 age_group_mean에 넣어주어 
    # 집계 함수 평균을 주었다.
```

이상치에 관한 아이디어는 EDA를 통해 개별적으로 알아보도록 하자. 포스팅을 작성하다보니 길이가 ㅎㄷㄷ

이 포스팅에선 `Age`컬럼에 대한 이상치만 제어할 것임.

```python
zscore_threshold = 1.96  # (97%의 데이터를 살린다)

outlier = train_df['Age'][ np.abs(stats.zscore(train_df['Age']) > zscore_threshold) ]

# print(len(outlier)) # 42개
# print(outlier)
train_df = train_df.loc[ ~train_df['Age'].isin(outlier), :]
display(train_df)  # 849 rows × 7 columns
```

`Age` binning 처리를 하자

```python
# age 카테고라이징

train_df.loc[train_df['Age'] < 8, 'Age'] = 0      # 8세 미만 아동
train_df.loc[(train_df['Age'] >= 8) & (train_df['Age'] < 20), 'Age'] = 1     # 청소년 층
train_df.loc[(train_df['Age'] >= 20) & (train_df['Age'] < 60), 'Age'] = 2    # working age
train_df.loc[train_df['Age'] >= 60, 'Age'] = 3    # 노년층

display(train_df)
```

그리고 임무를 다한 `Title`을 삭제해주고 이제 텐서플로우를 구현해보자.

```python
train_df.drop('Title', axis=1, inplace=True)
```

## TensorFlow 모델 구현

```python

# 1. train and validation data split
x_data = train_df.iloc[:,1:]
t_data = train_df.iloc[:,0].values
# print(y_data) # 우선 1차원으로
numcols = len(x_data.columns)

print(np.unique(t_data, return_counts=True))  # (array([0, 1]), array([549, 342]))

train_x_data, valid_x_data, train_t_data, valid_t_data = \
train_test_split(x_data, t_data, test_size=0.2, stratify=t_data, random_state=2)
```



```python
# 2. node 생성
X = tf.placeholder(shape=[None, numcols], dtype=tf.float32)
T = tf.placeholder(shape=[None, 1], dtype=tf.float32)

# W & b
W = tf.Variable(tf.random.normal([numcols, 1]))
b = tf.Variable(tf.random.normal([1]))

# Hypothesis
logit = tf.matmul(X, W) + b
H = tf.sigmoid(logit)

# loss function
loss = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(logits=logit, labels=T))

# train 
train = tf.train.GradientDescentOptimizer(learning_rate=1e-4).minimize(loss)

# Session
sess = tf.Session()
sess.run(tf.global_variables_initializer())

# repeatition
for step in range(300000):
    tmp, loss_val = sess.run([train, loss], feed_dict={X: train_x_data, T: train_t_data.reshape(-1,1)})
    
    if step % 30000 == 0:
        print('loss: {}'.format(loss_val))
```

```python
loss: 0.9275939464569092
loss: 0.5920800566673279
loss: 0.5484859347343445
loss: 0.527471125125885
loss: 0.5122371315956116
loss: 0.5007190108299255
loss: 0.49186426401138306
loss: 0.48495933413505554
loss: 0.4795133173465729
loss: 0.47516167163848877
```

```python
# 모델 프레딕션 고고 
# accuracy 얼마나 나왔을까?

predict = tf.cast(H > 0.5, dtype=tf.float32)

correct = tf.equal(predict, T)  # predict과 T의 값을 비교, boolean값을 토해냄
accuracy = tf.reduce_mean(tf.cast(correct, dtype=tf.float32))

acc_val = sess.run(accuracy, feed_dict={X: valid_x_data, T: valid_t_data.reshape(-1, 1)})
print(acc_val)   # 0.8117647
# 이상치를 제거했더니 높아졌다.
```

이렇게 `tensorflow`로 후다닥 구현을 해보았다. 

`kaggle`에서 데이터를 불러왔을때 다른 `.csv` 파일 두개가 더있었던 것을 기억하는가?

 `validation data set`대신 `test.csv`를 넣어주어 평가를 해보면 된다.

귀찮음이 사라진다면 이 포스팅에 추가적으로 어떻게 `test.csv`를 돌리는지 기재해놓도록 하겠다.

일단 오늘은 여기까지.

👋 

