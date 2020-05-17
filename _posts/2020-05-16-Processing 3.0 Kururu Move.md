---
layout: post
title:  "[Processing 3.0] Moving Character"
date:   2020-05-16
desc: "Quick test on writing code snippets in a blog post"
keywords: "Moving Character"
categories: [Processing3.0]
tags: [Processing3.0]
icon: icon-html
---


# Moving Character

Processing 3.0을 이용하여 지난 포스트에서 그린 쿠루루 캐릭터가 움직이며 커지는 게임
캐릭터는 class로 만들어 관리하기 쉽게 하였다.


![kururu_move](https://jimmy9704.github.io/image/kururu_move.mp4)



```java
int num = 10; //num of character
kururu[] k = new kururu[num];
```
화면에 표시할 쿠루루 캐릭터의 숫자를 배열로 동적할당하여 관리하기 쉽게 한다.(초기값 = 10)


```java
void setup(){
  size(600,600);
  for(int i = 0; i<num; i++){
    k[i] = new kururu(random(600),random(600),0.3,random(15,30),random(15,30));
  }
  background(189,189,189);
}
```
쿠루루의 초기 위치,속도는 랜덤하게 주어집니다.


```java
void draw(){
  background(189,189,189);
  for(int i = 0; i<num; i++){
    k[i].update();  k[i].display();  k[i] = compare(k[i]);  k[i] = set(k[i]);
  }
}
```
draw함수로 캐릭터를 1/60만 초마다 새로 그려준다


```java
float distance(kururu a,kururu b){
  float x = pow((a.getx()-b.getx()),2);
  float y = pow((a.gety()-b.gety()),2);
  float dis = pow(x+y,0.5);
  return dis;
}//cal distance of two kururues
```
distance함수는 두 캐릭터간의 좌표를 계산하는 코드이다.


```java
kururu compare(kururu a){
  int count = 0;
  for(int i = 0; i<num; i++){
    if (distance(a,k[i])<a.gets()*100){
      k[i] = new kururu();
      count ++;
    }
  }
  if (count >= 2){
    a = new kururu(a.getx(),a.gety(),a.gets()*1.5,10,10);
  }
  return a;
}//compare kururues distance and delete character
```
Compare함수는 쿠루루가 다른 쿠루루들과의 위치를 비교하여 자신 이외에 거리가 가까운 쿠루루가 있는지 판단하여 쿠루루가 합쳐지게 합니다.


합쳐질때   `a = new kururu(a.getx(),a.gety(),a.gets()*1.5,10,10);`


코드를 이용하여 크기를 1.5 커지게합니다.


 `k[i] = new kururu();` 코드는 부딪힌 다른 캐릭터를 지워줍니다


```java
kururu  set(kururu a){
  if (a.getx()<0  || a.gety()<0  ||  a.getx()>600  ||  a.gety()>600){
    a = new kururu(random(600),random(600),a.gets(),10,10);
  }
  return a;
}//if kururu out of range set kururu at random coordiate
```
Set함수는 쿠루루가 지정된 size(background)밖으로 나갈시 다시 size안으로 위치를 재설정 합니다.



이어질 코드는 kururu캐릭터의 class입니다
```java
class kururu {
  float x,y,s,vx,vy;
  kururu() {};
  kururu(float x_,float y_, float s_,float vx_,float vy_){
    x = x_;  y = y_;  s = s_;  vx = vx_;  vy = vy_;
  };

  void update(){
    float ran = random(1);
    if (ran < 0.25){
      x += vx;
      y += vy;
    }
    else if  (ran < 0.5){
      x -= vx;
      y += vy;
    }
    else if  (ran < 0.75){
      x += vx;
      y -= vy;
    }  
    else{
      x -= vx;
      y -= vy;
    }
  }//set kururu's direction by random (percentage 1/4)

  float getx(){return x;}
  float gety(){return y;}
  float gets(){return s;}
```
쿠루루는 x좌표,y좌표,사이즈,속력을 변수로 전달받아 초기화합니다.


Update함수는 0.25확률로 쿠루루의 이동 방향을 결정합니다.


Get함수는 각각 x좌표, y좌표, 사이즈를 리턴합니다.


```java
void display(){  //drow kururu
   fill(255,255,255);
   circle(x-95*s,y,80*s);
   circle(x+95*s,y,80*s);

   fill(42,0,100);

   circle(x-60*s,y,110*s);
   circle(x+60*s,y,110*s);

   fill(255,228,0);
   circle(x,y,200*s);//face

   fill(255,130,36);
   arc(x, y-50*s, 170*s, 100*s, PI, 2*PI);//hat

   fill(176,176,176,100);
   noStroke();
   ellipse(x, y-10*s, 120*s, 150*s);
   stroke(0);
     for(int i = 0; i<50; i +=12){
     line(x+50*s, y-(30-i)*s, x-50*s, y-(30-i)*s);
   }
   for(int i = 30; i>0; i -=10){
     line(x+i*s, y+(60-i)*s, x-i*s, y+(60-i)*s);
   }//gradation

   fill(255,255,255);
   circle(x-55*s,y-10*s,75*s);
   circle(x+55*s,y-10*s,75*s);
   float x_,y_;
   float r = 0;
   fill(0,0,0);
   for(int start = 0; start<10; start++){
     r = 0;
     for(int degree = start; degree<650; degree +=10){
       float radian = degree *3.141592/180;
       x_ = x-53*s + r * sin(radian);
       y_ = y-10*s + r * cos(radian);
       circle(x_,y_,3*s);
       r += 0.4*s;
     }
   }//left eye
   r = 0;
   for(int start = 0; start<10; start++){
     r = 0;
     for(int degree = start; degree<650; degree +=10){
       float radian = degree *3.141592/180;
       x_ = x+53*s + r * sin(radian);
       y_ = y-10*s + r * cos(radian);
       circle(x_,y_,3*s);
       r += 0.4*s;
     }
   }//right eye

   fill(255,255,255);
      arc(x, y+90*s, 80*s, 20*s, 9.5, 12.5);
      arc(x, y+90*s, 80*s, 20*s, 12.5, 15.8);
      arc(x, y+82*s, 25*s, 15*s, 12.5, 15.8);
      line(x, y+80*s, x, y+89*s);
      line(x-8*s, y+80*s, x-8*s, y+88*s);
      line(x+8*s, y+80*s, x+8*s, y+88*s);//mouth

      fill(255,228,0);
      arc(x, y-65*s, 30*s, 30*s, 9.5, 12.5);
      arc(x+5*s, y-65*s, 20*s, 20*s, 12.5, 15.8);
      arc(x, y-65*s, 10*s, 10*s, 9.5, 12.5);
      arc(x-10*s, y-65*s, 10*s, 5*s, 12.5, 15.8);

      line(x+87*s, y-52*s, x-87*s, y-52*s);
    }
  }
```
Display 함수는 쿠루루 캐릭터의 형태를 결정합니다.


Display함수는 지난 포스트에서 그린 캐릭터를 class로 만들어 활용하였습니다
