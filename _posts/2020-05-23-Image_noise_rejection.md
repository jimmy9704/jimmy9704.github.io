---
layout: post
title:  "[Python] Image noise rejection"
date:   2020-05-23
desc: "Quick test on writing code snippets in a blog post"
keywords: "Image noise rejection"
categories: [Python]
tags: [PYTHON]
icon: icon-html
---


# Ex) Image noise rejection

EE370: Software lab, Kyung Hee University.
Jong-Han Kim (jonghank@khu.ac.kr)


```py
import numpy as np
import matplotlib.pyplot as plt

X = plt.imread('https://people.sc.fsu.edu/~jburkardt/data/png/lena.png')
X_noise = X.copy()
n = np.zeros(X.shape)
for i in range(128):
  for j in range(128):
    n[4*i:2+4*i:, 4*j:2+4*j] = 1
X_noise = X + n

plt.figure(figsize=(16, 9), dpi=100)
plt.subplot(121)
plt.imshow(X, cmap = 'gray')
plt.title("Lena original image")
plt.subplot(122)
plt.imshow(X_noise, cmap = 'gray')
plt.title("Lena with noise")
plt.show()
```


![Lena](https://jimmy9704.github.io/image/Lena.png)


이미지와 이미지에 노이즈를 씌워서 plot한다


```py
x_fft = np.fft.fftshift(np.fft.fft2(X))
plt.figure(figsize=(16, 9), dpi=100)
plt.imshow(np.log(abs(x_fft)+1),cmap = 'gray')
plt.show()
```


노이즈 없는 이미지를 푸리에 변환하여 plot한다


이미지에서 주파수란 이미지가 변화하는 정도를 의미하며


푸리에스펙트럼의 가운데는 저역을 멀어질수록 고역 주파수를 나타낸다


![Lena_fft](https://jimmy9704.github.io/image/Lena_fft.png)


```py
X_noise_fft = np.fft.fftshift(np.fft.fft2(X_noise))
plt.figure(figsize=(16, 9), dpi=100)
plt.imshow(np.log(abs(X_noise_fft)+1),cmap = 'gray')
plt.show()
```


![Lena_fft_noise](https://jimmy9704.github.io/image/Lena_fft_noise.png)


노이즈가 있는 이미지를 푸리에 변환하여 plot한다


주기적으로 나타나는 흰색 점을 확인할 수 있다.


```py
# your code here
X_recon = np.copy(X_noise_fft)

for i in range(0,len(X_recon),int(len(X_recon)/4)):
  for j in range(0,len(X_recon),int(len(X_recon)/4)):
    if i != int(len(X_recon)/2) or j != int(len(X_recon)/2):
      X_recon[i-5:i+5,j-5:j+5].fill(0)


plt.figure(figsize=(16, 9), dpi=100)
plt.subplot(121)
plt.imshow(np.log(abs(X_recon)+1),cmap = 'gray')
plt.title("Not erase origin dot")
plt.subplot(122)
plt.imshow(np.abs(np.fft.ifft2(X_recon)),cmap = 'gray')
plt.title("Lena with out noise")
plt.show()
```


![Lena_noise](https://jimmy9704.github.io/image/Lena_noise.png)


노이즈가 없는 사진에서 저역의 주파수 즉 푸리에스펙트럼에서 가운데 부분 즉 저역 주파수가 강하게 나타나는데


저역 주파수를 노이즈 제거하는 것 처럼 0으로 만들면 기존 사진에 손상이 생긴다.


따라서 저역 부분인 가운데 흰 점을 제외하고 나머지 흰 점을 검은색 즉 0으로 만들어 노이즈를 제거한다
