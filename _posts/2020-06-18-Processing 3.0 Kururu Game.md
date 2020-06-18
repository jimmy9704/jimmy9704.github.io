---
layout: post
title:  "[Processing 3.0] Kururu likes curry game"
date:   2020-06-18
desc: "Quick test on writing code snippets in a blog post"
keywords: "Kururu likes curry game"
categories: [Processing3.0]
tags: [Processing3.0]
icon: icon-html
---


# Kururu likes curry game

Processing 3.0을 이용하여 만든 쿠루루 게임

<구현부>


```java
int num = 5; //num of character
kururu[] k = new kururu[num];
my_kururu[] r = new my_kururu[1];
score[] s= new score[num];
int sc = 0;
int count = 0;


void setup(){
  size(600,600);
  for(int i = 0; i<num; i++){
    k[i] = new kururu(random(400),random(400),0.3,random(15,25),random(15,25));
    s[i] = new score(random(450),random(450));
  }
  r[0] =  new my_kururu(500,500,0.3,30,30);
  background(189,189,189);
  for(int i = 0; i<num; i++){
    k[i].update();  k[i].display();  k[i] = set(k[i]);
  }
   r[0].update(); r[0].display();
}

void draw(){
  background(189,189,189);
  fill(0,0,0);
  textSize(25);
  text("score: ",490,30);
  text(sc,570,30);
  r[0].update();  r[0].display();
  for (int i = 0; i<num; i++){
    count += compare(k[i],r[0]);
    int X = comp_score(s[i],r[0]);
    if(X == 1){
      s[i] = new score(random(600),random(600));
      sc += X;
    }
  }
  if(count == 0){
    for (int i = 0; i<num; i++){
      k[i].update();  k[i].display();  k[i] = set(k[i]);
      s[i].display();
    }
  }//play game
  else{
    background(189,189,189);
    textSize(50);
    fill(0,0,0);
    text("game over",150,200);
    text("score: ",150,300);
    text(sc,350,300);
    text("Push 'a' key to restart",30,400);
    for(int i = 0; i<num; i++){
      k[i] = new kururu(random(400),random(400),0.3,random(15,25),random(15,25));
      s[i] = new score(random(450),random(450));
    }//game over
    r[0] =  new my_kururu(500,500,0.3,random(15,30),random(15,30));
    if(keyPressed){
      if(key == 'a'){
        count = 0;
        sc = 0;
      }//push a to restart
    }
  }
}

int comp_score(score a, my_kururu b){
  float x = pow((a.getx()-b.getx()),2);
  float y = pow((a.gety()-b.gety()),2);
  float dis = pow(x+y,0.5);
  if(dis <40)  return 1;
  return 0;
}//cal score

float distance(kururu a,my_kururu b){
  float x = pow((a.getx()-b.getx()),2);
  float y = pow((a.gety()-b.gety()),2);
  float dis = pow(x+y,0.5);
  return dis;
}//cal distance

int compare(kururu a,my_kururu b){
  if (distance(a,b)<a.gets()*190) return 1;
 return 0;
}//compare kururues distance

kururu  set(kururu a){
  if (a.getx()<0  || a.gety()<0  ||  a.getx()>600  ||  a.gety()>600){
    int ran = int(random(2));
    if(ran == 0) a = new kururu(random(r[0].getx()-100),random(r[0].gety()-100),0.3*(sc/5+1),random(15,25),random(15,25));
    else a = new kururu(random(r[0].getx()+100,600),random(r[0].gety()+100,600),0.3*(sc/5+1),random(15,25),random(15,25));
  }
  return a;
}//if kururu out of range set kururu at random coordiate
```


<character 부모 class>


display함수의 설명은 Kururu draw 포스터에 있습니다.


```java
class character {
  float x,y,s,vx,vy;
  int c;//color
  character() {};

  float getx(){return x;}
  float gety(){return y;}
  float gets(){return s;}

  void display(){  //drow kururu
    fill(255,255,255);
    circle(x-95*s,y,80*s);
    circle(x+95*s,y,80*s);

    fill(42,0,100);

    circle(x-60*s,y,110*s);
    circle(x+60*s,y,110*s);

    fill(255,c,0);
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

    fill(255,c,0);
    arc(x, y-65*s, 30*s, 30*s, 9.5, 12.5);
    arc(x+5*s, y-65*s, 20*s, 20*s, 12.5, 15.8);
    arc(x, y-65*s, 10*s, 10*s, 9.5, 12.5);
    arc(x-10*s, y-65*s, 10*s, 5*s, 12.5, 15.8);

    line(x+87*s, y-52*s, x-87*s, y-52*s);
  }
}
```


<빨간 쿠루루 (자식) class>

```java
class kururu extends character{
  kururu() {};
  kururu(float x_,float y_, float s_,float vx_,float vy_){
    x = x_;  y = y_;  s = s_;  vx = vx_;  vy = vy_; c = 0;
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
}

```


<노란 쿠루루 (자식) class>


```java
class my_kururu extends character{
  my_kururu() {};
  my_kururu(float x_,float y_, float s_,float vx_,float vy_){
    x = x_;  y = y_;  s = s_;  vx = vx_;  vy = vy_; c = 228;
  };
  void update(){
    if (keyPressed){
      if (key == CODED){
        if (keyCode == UP) y -= vy;
        else if (keyCode == DOWN) y += vy;
        else if (keyCode == LEFT) x -= vx;
        else if (keyCode == RIGHT) x += vx;
      }
    }
  }//set kururu's direction by key  
}
```


<score class>


```java
class score{
  float x,y,s,vx,vy;
  score() {};
  score(float x_,float y_){
    x = x_;  y = y_;
  };

 void display(){
    fill(255,255,255);
    arc(x, y, 30, 20, 12.5, 15.8);
    arc(x, y, 20, 20, 9.5, 12.5);
    fill(153,112,0);
    arc(x, y, 20, 20, 9.5, 11.5);

 }
 float getx(){return x;}
 float gety(){return y;}
}
```
