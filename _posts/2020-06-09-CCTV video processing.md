---
layout: post
title:  "[Python] CCTV video processing"
date:   2020-06-09
desc: "Quick test on writing code snippets in a blog post"
keywords: "CCTV video processing"
categories: [Python]
tags: [PYTHON]
icon: icon-html
---


# Ex) CCTV_video_processing

EE370: Software lab, Kyung Hee University.
Jong-Han Kim (jonghank@khu.ac.kr)




```py
import os
from google.colab import drive
drive.mount('/content/drive')
os.chdir('/content/drive/My Drive/Colab Notebooks/ee370')
import cv2
import numpy as np
import matplotlib.pyplot as plt


cap = cv2.VideoCapture('vtest.avi')
```

test비디오를 읽어온다


https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_video/py_bg_subtraction/py_bg_subtraction.html


위 사이트의 영상을 사용하였다



<iframe width="815" height="611" src="https://www.youtube.com/embed/GOSRE9WPxWc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```py
out = cv2.VideoWriter('output.avi', cv2.VideoWriter_fourcc(*'DIVX'), fps, (width, height))
fgbg = cv2.bgsegm.createBackgroundSubtractorMOG()

while(1):

  ret, frame = cap.read()

  if ret:
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
    fgmask = fgbg.apply(hsv)
    mask1 = fgmask

    lower_red = np.array([0,150,50])
    upper_red = np.array([10,255,255])

    mask2 = cv2.inRange(hsv, lower_red, upper_red)

    lower_red = np.array([170,150,50])
    upper_red = np.array([180,255,255])

    mask3 = cv2.inRange(hsv, lower_red, upper_red)

    mask = mask1 * (mask2 + mask3)
    res = cv2.bitwise_and(frame,frame, mask= mask)

    out.write(res)
  else:
    break

cap.release()
out.release()
```
`fgbg = cv2.bgsegm.createBackgroundSubtractorMOG()` MOG를 이용하여 배경과 움직이는 객체를 분리한다


hsv색상의 경우 0부터 10까지 , 170부터 180까지로 빨간색이 표현되기 때문에 mask2와 mask3로 합쳐서 필터링 하였다.


```py
def set_frame(t,frame):
  global track_window_1
  global track_window_2
  for i in range(t):
    ret ,frame = cap.read()
    if ret == True:
      hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
      dst = cv2.calcBackProject([hsv],[0],roi_hist,[0,180],1)

      # apply meanshift to get the new location
      ret, track_window_1 = cv2.meanShift(dst, track_window_1, term_crit)
      ret, track_window_2 = cv2.meanShift(dst, track_window_2, term_crit)

      # Draw it on image
      x_1,y_1,w_1,h_1 = track_window_1
      x_2,y_2,w_2,h_2 = track_window_2

      img1 = cv2.rectangle(frame, (x_1,y_1), (x_1+125,y_1+90), 255,2)
      img2 = cv2.rectangle(img1, (x_2,y_2), (x_2+125,y_2+90), 255,2)

      out.write(img2)
    else:
      break
```
t초동안의 frame을 읽고 빨간색 객체를 추적하고 프레임을 씌우는 함수이다.


https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_video/py_meanshift/py_meanshift.html



위 사이트의 예제 코드를 활용하였다.


<iframe width="815" height="611" src="https://www.youtube.com/embed/La3j5AKiPds" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```py
cap = cv2.VideoCapture('output.avi')
out = cv2.VideoWriter('frame.avi', cv2.VideoWriter_fourcc(*'DIVX'), fps, (width, height))

for i in range(10):
  ret,frame = cap.read()
  out.write(frame)  #처음 사람이 출현하는 순간까지 스킵

# take first frame of the video
ret,frame = cap.read()

# setup initial location of window
track_window_1 = (600,300,125,90)
track_window_2 = (600,300,125,90)

# set up the ROI for tracking
roi = frame[300:300+90, 600:600+125]
hsv_roi =  cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
mask = cv2.inRange(hsv_roi, np.array((0., 60.,32.)), np.array((180.,255.,255.)))
roi_hist = cv2.calcHist([hsv_roi],[0],mask,[180],[0,180])
cv2.normalize(roi_hist,roi_hist,0,255,cv2.NORM_MINMAX)

# Setup the termination criteria, either 10 iteration or move by atleast 1 pt
term_crit = ( cv2.TERM_CRITERIA_EPS | cv2.TERM_CRITERIA_COUNT, 10, 1 )

set_frame(350,frame) #42초동안 tracing

for i in range(70):
  ret,frame = cap.read()
  out.write(frame)  #2번째 사람이 출현하는 순간까지 스킵

track_window_2 = (600,500,125,90)
set_frame(190,frame)

track_window_1 = (50,150,125,90)
set_frame(55,frame)

track_window_1 = track_window_2
set_frame(25,frame)

track_window_1 = (500,130,125,90) #겹치는 부분 처리
set_frame(500,frame)

cap.release()
out.release()
```
객체가 화면에서 사라진후 다시 나타났을때 객체를 인식할 알고리즘을 구현하지못하였다.


따라서 객체가 다시 나타났을떄의 시간과 위치를 hard코딩하여 프레임의 초기 위치를 변경해주었다.
