---
layout: post
title:  "[Python] Random walk in financial markets"
date:   2020-05-23
desc: "Quick test on writing code snippets in a blog post"
keywords: "Random walk in financial markets"
categories: [Python]
tags: [PYTHON]
icon: icon-html
---


# Ex) Random walk in financial markets

EE370: Software lab, Kyung Hee University.
Jong-Han Kim (jonghank@khu.ac.kr)




```py
import numpy as np
import matplotlib.pyplot as plt
from pandas_datareader import data

alpha = data.DataReader('GOOGL', 'yahoo', start='8/19/2004')
price = alpha['Adj Close'].values

plt.figure(figsize=(10,3), dpi=100)
plt.plot(price)
plt.grid(True)
plt.xlabel('Days')
plt.ylabel('Adjusted close price')
plt.title('Alphabet Inc. (GOOGL)')
plt.show()
```

구글의 주식데이터를 불러와 plot 한다
![Google](https://jimmy9704.github.io/image/Google.png)

r(t) = {p(t)−p(t−1)}/p(t−1) ×100 (%)

```py
def r(t):
  return (price[t]-price[t-1])/price[t-1]

T = len(price)

R = []
for i in range(1,T):
  R.append(r(i))

v=np.std(R)*np.sqrt(252)
print(v)
```
주어진 r(t)수식을 통해 주식의 다음날 성장률을 구한다


v는 주식의 변동률 이며 이는 r의 표준편차에 루트252를 곱한것이다


252는 1년간 주식시장이 열린 날의 수이다


```py
X = price[T-1]/price[0]
CAGR = (((X)**(252/T))-1)
print(CAGR*100,"%")
```

연평균 성장률 CAGR=((p(τ)/p(1))^(252/τ) −1)  ×100 (%)


로 정의된다


```py
def p(t):
  return fprice[t]*(1+rt[i])

plt.figure(figsize=(10,3), dpi=100)
plt.plot(price)
ftime = range(T,T+253)

result_price = []
for i in range(10000):
  rt = np.random.normal(CAGR/252,v/np.sqrt(252),252)
  fprice = [price[T-1]]
  for i in range(252):
    fprice.append(p(i))
  plt.plot(ftime,fprice)
  result_price.append(fprice[len(fprice)-1])
plt.grid(True)
plt.xlabel('Days')
plt.ylabel('Adjusted close price')
plt.title('Alphabet Inc. (GOOGL)')
plt.show()
```


![1000randGoogle](https://jimmy9704.github.io/image/1000randGoogle.png)


rt는 평균이 CAGR/252이고 분산이 v/np.sqrt(252) 인 가우시안 분포에서의 랜덤한 수이다


`rt = np.random.normal(CAGR/252,v/np.sqrt(252))`


rt를 이용하여 1년뒤의 주식 동향을 예측하여 plot 하고 이를 1000번 시행한다


```py
Expected_value = np.mean(result_price)

nf_Val = np.percentile(result_price,5)
count = 0

for i in result_price:
  if i >= 2*price[T-1]:
    count += 1


percent = count/len(result_price)

print("Expected_value : ",Expected_value)
print("95% probability : ",nf_Val)
print("probability:",percent*100,"%")

plt.figure(figsize = (10,3))
plt.hist(result_price,bins = 100)
plt.axvline(Expected_value,color = 'r',label='Mean')
plt.axvline(nf_Val,color = 'g',label='95% probability')
plt.grid(True)
plt.legend()
plt.xlabel('Evaluate Price')
plt.ylabel('Frequency')
plt.show()
```


![histGoogle.png](https://jimmy9704.github.io/image/histGoogle.png)


result_price는 1년뒤 주식시장의 예상 가격이다


위 히스토그램은 result_price를 나타낸 것이다


초록 선은 상위 5퍼센트의 기댓값중 가장 낮은 값을 나타낸 것이다


빨간 선은 평균값을 나타낸다


```py
for i in result_price:
  if i >= 2*price[T-1]:
    count += 1


percent = count/len(result_price)
```
위 코드를 이용하여 1년뒤 가격이 현재의 2배가 될 확률을 계산할 수 있다.
