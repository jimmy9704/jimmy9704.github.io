---
layout: post
title:  "[Python] Google PageRank"
date:   2020-05-05
desc: "Quick test on writing code snippets in a blog post"
keywords: "Prime counting function"
categories: [Python]
tags: [PYTHON]
icon: icon-html
---


# Ex) Google PageRank

EE370: Software lab, Kyung Hee University. assignment
Jong-Han Kim (jonghank@khu.ac.kr)



The Google PageRank, named after Larry Page, one of the co-founders of Google, is an algorithm that ranks the webpages based on the hyperlink connections of the webpages on relevant topics, hence the Google search can return the list of the webpages in an appropriate order of the PageRank indices.

According to Google, the PageRank works by counting the number and quality of links to a page to determine a rough estimate of how important the website is. The underlying assumption is that more important websites are likely to receive more links from other websites.

  <img src="https://upload.wikimedia.org/wikipedia/en/8/8b/PageRanks-Example.jpg" width="500">

The PageRank algorithm outputs a probability distribution used to represent the likelihood that a person randomly clicking on links will arrive at any particular page.



Now a toy example. Suppose we have the small space of 11 webpages whose list of outbound link connections are given by

- Page A has links to A,B,C,D,E,F,G,H,I,J,K (everyone)
- Page B has links to C
- Page C has links to B
- Page D has links to A, B
- Page E has links to B, D, F
- Page F has links to B, E
- Page G has links to B, E
- Page H has links to B, E
- Page I has links to B, E
- Page J has links to E
- Page K has links to E


1.링크된 웹사이트의 수를 고려하여 M행렬을 만든다


```python
import numpy as np

M = ([[1/11,0,0,1/2,0,0,0,0,0,0,0],
      [1/11,0,1,1/2,1/3,1/2,1/2,1/2,1/2,0,0],
      [1/11,1,0,0,0,0,0,0,0,0,0],
      [1/11,0,0,0,1/3,0,0,0,0,0,0],
      [1/11,0,0,0,0,1/2,1/2,1/2,1/2,1,1],
      [1/11,0,0,0,1/3,0,0,0,0,0,0],
      [1/11,0,0,0,0,0,0,0,0,0,0],
      [1/11,0,0,0,0,0,0,0,0,0,0],
      [1/11,0,0,0,0,0,0,0,0,0,0],
      [1/11,0,0,0,0,0,0,0,0,0,0],
      [1/11,0,0,0,0,0,0,0,0,0,0]])
print(np.shape(M))
```


2.norm이 1인 랜덤한 행렬x를 만들고 result와 x의 차에 norm(2)를  한값이 1e-6보다 작을때 까지 while문을 돌린다.
  result=(1−𝜁)/𝑛*1+𝜁𝑀x 이다
  B웹사이트가 가장 점수가 높은 것을 알 수 있다.


```python
x = np.random.rand(11)
while(np.linalg.norm(x,1)== 1):
  x = np.random.rand(11)


result = 0.0136*np.ones(11) + 0.85*(M@x)
count = 1
while(np.linalg.norm(result - x,2)>=10e-6):
  x = result
  result = 0.0136*np.ones(11) + 0.85*(M@x)
  count += 1

print("a)",count)
index = 0
for i in result:
  if(i == np.max(result)):
    break
  else:
    index += 1
print("b) highest Page is",chr(65+index),":" ,np.max(result))
num = 0
for i in result:
  print(chr(65+num),":" ,i)
  num += 1



```
