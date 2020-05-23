---
layout: post
title:  "[Linux] Linux ad-hoc mode"
date:   2020-05-23
desc: "Quick test on writing code snippets in a blog post"
keywords: "ad-hoc"
categories: [Linux]
tags: [Linux]
icon: icon-html
---


# Linux ad-hoc mode


/etc/network/interfaces의 파일을 아래와 같이 수정한다


```
auto wlan0
iface wlan0 inet static
address 192.168.4.1
netmask 255.255.255.0
wireless-mode ad-hoc
wireless-channel 1
wireless-essid testadhoc
```

- wlan0: 무선랜 인터페이스 이름
- address: 사용하기 위한 주소
- node1의 경우 192.168.4.1 node2의 경우 192.168.4.2 ... `변경가능`
- wireless-channel 1: 사용하기위한 채널 `변경가능`


재부팅후 모드 변경과 ip를 확인해준다
