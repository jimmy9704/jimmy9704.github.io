---
layout: post
title:  "[Linux] picamera"
date:   2020-07-12
desc: "Quick test on writing code snippets in a blog post"
keywords: "picamera"
categories: [Linux]
tags: [Linux]
icon: icon-html
---


# picamera


라즈베리파이의 카메라를 vlc를 이용하여 원격 스트리밍 한다.


```
$ raspivid -o - -t 0 -n -w 800 -h 480 -fps 24 | cvlc -vvv stream:///dev/stdin --sout '#standard{access=http,mux = ts,dst=:8090}' :demux=h264
```

- -o - : -의 경우 스트리밍으로 출력하기 위해 파일명을 지정하지 않는것
- -t 0 :무제한 시간동안의 스트리밍
- -w, -h : 가로 세로 크기
- -fps : 프레임 1초당 이미지 변화수
- | clvc :파이프를 이용하여 앞의 결과를 vlc플레이어로 전달
-  --sout '#standard{access=http,mux = ts,dst=:8090} : 8090포트를 이용하여 영상출력


vlc의 네트워크 캐쉬를 줄여서 영상지연을 줄일 수 있지만 네트워크 품질의 영향을 많이 받는다
