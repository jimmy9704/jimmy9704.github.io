---
layout: post
title:  "[Processing 3.0] Drawing Character"
date:   2020-05-05
desc: "Quick test on writing code snippets in a blog post"
keywords: "Drawing Character"
categories: [Processing 3.0]
tags: [C]
icon: icon-html
---


# Drawing Character

Processing 3.0을 이용하여 쿠루루 캐릭터를 그리는 코드이다


```c
void setup(){
  size(400,600);
  float  w = width/2;
  float  h = height/2;
  background(189,189,189);

  kururu(w,h);
}
```

```c

void kururu(float x, float y){
  circle(x-95,y,80);
  circle(x+95,y,80);
  fill(42,0,100);
  circle(x-60,y,110);
  circle(x+60,y,110);

  fill(255,228,0);
  circle(x,y,200);//face

  fill(255,130,36);
  arc(x, y-50, 170, 100, PI, 2*PI);//hat

  fill(176,176,176,100);
  noStroke();
  ellipse(x, y-10, 120, 150);
  stroke(0);
    for(int i = 0; i<50; i +=12){
    line(x+50, y-30+i, x-50, y-30+i);
  }
  for(int i = 30; i>0; i -=10){
    line(x+i, y+60-i, x-i, y+60-i);
  }//gradation


  fill(255,255,255);
  circle(x-55,y-10,75);
  circle(x+55,y-10,75);
  float x_,y_;
  float r = 0;
  fill(0,0,0);
  for(int start = 0; start<10; start++){
    r = 0;
    for(int degree = start; degree<650; degree +=10){
      float radian = degree *3.141592/180;
      x_ = x+53 + r * sin(radian);
      y_ = y-10 + r * cos(radian);
      circle(x_,y_,3);
      r += 0.4;
    }
  }//right eye

  fill(255,255,255);
  arc(x, y+90, 80, 20, 9.5, 12.5);
  arc(x, y+90, 80, 20, 12.5, 15.8);
  arc(x, y+82, 25, 15, 12.5, 15.8);
  line(x, y+80, x, y+89);
  line(x-8, y+80, x-8, y+88);
  line(x+8, y+80, x+8, y+88);//mouth

  fill(255,228,0);
  arc(x, y-65, 30, 30, 9.5, 12.5);
  arc(x+5, y-65, 20, 20, 12.5, 15.8);
  arc(x, y-65, 10, 10, 9.5, 12.5);
  arc(x-10, y-65, 10, 5, 12.5, 15.8);

  line(x+87, y-52, x-87, y-52);
}

```
