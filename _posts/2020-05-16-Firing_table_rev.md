---
layout: post
title:  "[Python] Firing table"
date:   2020-05-16
desc: "Quick test on writing code snippets in a blog post"
keywords: "Firing table"
categories: [Python]
tags: [PYTHON]
icon: icon-html
---


# Ex) Firing table

EE370: Software lab, Kyung Hee University.
Jong-Han Kim (jonghank@khu.ac.kr)


This example involves the kinematics of a projectile, whose dynamical relations can be described by the following differential equations


mV˙=−0.5ρ(V^2)SCd−mgsinγ


Vγ˙=−gcosγ


R˙=Vcosγ


h˙=Vsinγ



<img src="https://jonghank.github.io/ee370/files/projectile.png" width="600">


ρ(h)=1.225(1−2.256×10−5h)5.256


Suppose a projectile with the following specifications is launched.

- m: mass (=40kg)
- d: diameter (=16cm)
- S: cross-section area (=πd^2/4)
- C_d: drag coefficient (=0.2)
- V(0): initial velocity (=1000m/s)





**(Problem 1)**


미분방정식을 `scipy`모듈을 이용하여 풀고 미사일의 궤적을 plot한다.


```python
import numpy as np
import matplotlib.pyplot as plt

import scipy.integrate as spi
```
```python
g=9.8
m=40
D=0.16
S=0.25*np.pi*D**2
Cd=0.2
```
주어진 식에 맞게 변수값을 지정해준다
```py
def model(z,t):
  v,r,R,h = z
  rho = 1.225*(1-2.256*1e-5*h)**5.256
  vdot=-0.5*rho*(v**2)*S*Cd/m -g*np.sin(r)
  hdot=v*np.sin(r)
  Rdot=v*np.cos(r)
  rdot=-g*np.cos(r)/v
  return [vdot,rdot,Rdot,hdot]
```
모듈 함수를 만들어준다
```py
t = np.arange(0,140,0.1)
r = 20


plt.figure(num = 1,figsize=(8,6), dpi=100)
plt.figure(num = 2,figsize=(8,6), dpi=100)
```
시간축을 지정해주고 고도-거리 그래프와 속력-시간 그래프를 그릴 공간을 할당해준다


```py
while r <= 70:
  ic = [1000,r*np.pi/180,0,0]
  X = np.array([])
  Y = np.array([])
  V = np.array([])
  for i in range(len(t)-2):
    states = spi.odeint(model,ic,t[i:i+2])
    X = np.append(X,states[0,2]/1000)
    Y = np.append(Y,states[0,3]/1000)
    V = np.append(V,states[0,0])
    ic = [states[1,0],states[1,1],states[1,2],states[1,3]]
    if states[1,3]<0:
      plt.figure(num = 1)
      plt.plot(X,Y,label=f'FPA={r}deg')
      plt.figure(num = 2)
      plt.plot(t[0:i+1],V,label=f'FPA={r}deg')
      break
  r += 5
```
적분범위를 모든 시간 범위에서 표현하게 되면


고도가 0 이하일때의 값들 또한 포함되어 메모리를 효율적으로 관리할 수 없다.


따라서 수치해석적으로 구간을 나눠 적분을 하면서 고도가 0이상인 값만 append하였다.


![Fring_table](https://jimmy9704.github.io/image/Fring_table.png)


```py
plt.figure(num = 1)
plt.legend()
plt.grid()
plt.xlabel('range(km)')
plt.ylabel( 'altitude(km)')

plt.figure(num = 2)
plt.legend()
plt.grid()
plt.xlabel('time(sec)')
plt.ylabel('speed(m)')
plt.show()

```
그래프를 그린다


**(Problem 2)**
도달 거리가 최대일 각도를 구해보았다.


```python
r = 50
t = np.arange(0,140,0.03)
plt.figure(num = 1,figsize=(4,3), dpi=100)
Max = 0
Maxdegree = 0
while r < 55:
  ic = [1000,r*np.pi/180,0,0]
  X = np.array([])
  Y = np.array([])
  for i in range(len(t)-2):
    states = spi.odeint(model,ic,t[i:i+2])
    if (states[0,2]/1000) > 37.5:
      X = np.append(X,states[0,2]/1000)
      Y = np.append(Y,states[0,3]/1000)

      if states[1,3]<0:
        plt.figure(num = 1)
        plt.plot(X[:],Y[:],label=f'FPA={r}deg')
        if X[len(X)-1] >Max:
          Max = X[len(X)-1]
          Maxdegree = r
        break
    ic = [states[1,0],states[1,1],states[1,2],states[1,3]]
  r += 1
print('Maximum range : ',Max)
print('degree : ',Maxdegree)
plt.legend()
plt.grid()
plt.xlabel('range(km)')
plt.ylabel( 'altitude(km)')
plt.show()
```
![Fring_table_2](https://jimmy9704.github.io/image/Fring_table_2.png)


**(Problem 3)** 미사일에 4000뉴턴의 엔진을 달고 낙하하기전에 낙하산을 펼쳐 속도를 줄이는 모델을 시뮬레이션 해보았다.


mV˙=−0.5ρV2SCd−mgsinγ+T


Vγ˙=−gcosγ


R˙=Vcosγ


h˙=Vsinγ


T=4000N if t≤10s


Cd=10 if γ≤0 and h≤1000m


Cd=0.2 otherwise


Find a sample trajectory when


- m: mass (=40kg)
- d: diameter (=16cm)
- S: cross-section area (=π d^2/4)
- V(0): initial velocity (=1)
- γ(0): launch angle (=50deg)


```python
g=9.8
m=40
D=0.16
S=0.25*np.pi*D**2
T = 0

def model(z,t):
  v,r,R,h = z

  if t<= 10:
    T = 4000
  else:
    T = 0
  if r<=0 and h<=1000:
    Cd=10
  else:
    Cd = 0.2
  rho = 1.225*(1-2.256*1e-5*h)**5.256
  vdot=-0.5*rho*(v**2)*S*Cd/m -g*np.sin(r) +T/m
  hdot=v*np.sin(r)
  Rdot=v*np.cos(r)
  rdot=-g*np.cos(r)/v
  return [vdot,rdot,Rdot,hdot]
```
문제 1번에서와 동일하게 model함수를 만들어주고 변수들을 지정해준다.


단 엔진과 낙하산에 맞춰 문제1번에서의 model과 다르게 식을 만들어준다


```py
t = np.arange(0,140,0.1)
r = 50

plt.figure(num = 1,figsize=(8,6), dpi=100)
plt.figure(num = 2,figsize=(8,6), dpi=100)

ic = [1,r*np.pi/180,0,0]
X = np.array([])
Y = np.array([])
V = np.array([])
for i in range(len(t)-2):
  states = spi.odeint(model,ic,t[i:i+2])
  X = np.append(X,states[0,2]/1000)
  Y = np.append(Y,states[0,3]/1000)
  V = np.append(V,states[0,0])
  ic = [states[1,0],states[1,1],states[1,2],states[1,3]]
  if states[1,3]<0:
    plt.figure(num = 1)
    plt.plot(X,Y,label=f'FPA={r}deg')
    plt.figure(num = 2)
    plt.plot(t[0:i+1],V,label=f'FPA={r}deg')
    break

plt.figure(num = 1)
plt.legend()
plt.grid()
plt.xlabel('range(km)')
plt.ylabel( 'altitude(km)')

plt.figure(num = 2)
plt.legend()
plt.grid()
plt.xlabel('time(sec)')
plt.ylabel('speed(m)')
plt.show()

```
문제1번과 동일하게 구간을 나누어 적분하여 고도가 0미만일 구간에 대해서는 고려하지 않는다


![Fring_table_3](https://jimmy9704.github.io/image/Fring_table_3.png)
