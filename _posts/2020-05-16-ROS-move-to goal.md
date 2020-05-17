---
layout: post
title:  "[ROS] Control two tb3 by using SLAM navigation, move_base node"
date:   2020-05-16
desc: "Quick test on writing code snippets in a blog post"
keywords: "Drawing Character"
categories: [Ros]
tags: [ROS]
icon: icon-html
---


# Control two tb3 by using SLAM navigation, move_base node


rospy 라이브러리를 이용하여 터틀봇 2대를 control하는 프로그램이다.


tb3_0의 좌표를 전달받아 tb3_1이 움직일 좌표로 지정해준다.


tb3_0는 SLAM navigation을 통해 사용자가 지정해준 좌표로 ridar를 이용해  자율주행한다.


tb3_1또한 SLAM navigation을 이용하여 이동한다.



```py
import rospy
from nav_msgs.msg import Odometry
from geometry_msgs.msg import Pose
from move_base_msgs.msg import MoveBaseActionGoal
import time

x1=0.0
y1=0.0
z1=0.0
x2=0.0
y2=0.0
z2=0.0
w2=0.0
```
활용할 라이브러리들을 import 해주고 좌표를 저장할 전역변수들을 지정해준다


```py
def newOdom(msg):
  global x1
  global y1
  global z1
  global x2
  global y2
  global z2
  global w2

  x1=msg.pose.pose.position.x
  y1=msg.pose.pose.position.y
  z1=msg.pose.pose.position.z
  x2=msg.pose.pose.orientation.x
  y2=msg.pose.pose.orientation.y
  z2=msg.pose.pose.orientation.z
  w2=msg.pose.pose.orientation.w
```
newOdom함수는 tb3_0의 Odom노드에서 좌표를 받아 변수에 저장한다


```py
rospy.init_node("pose_to_goal")

sub=rospy.Subscriber("/tb3_0/odom",Odometry,newOdom)
pub=rospy.Publisher("/tb3_1/move_base/goal", MoveBaseActionGoal,queue_size=3)

target=MoveBaseActionGoal()


while not rospy.is_shutdown():
  target.goal.target_pose.header.frame_id="map"
  target.goal.target_pose.pose.position.x=x1
  target.goal.target_pose.pose.position.y=y1
  target.goal.target_pose.pose.position.z=z1
  target.goal.target_pose.pose.orientation.x=x2
  target.goal.target_pose.pose.orientation.y=y2
  target.goal.target_pose.pose.orientation.z=z2
  target.goal.target_pose.pose.orientation.w=w2
  pub.publish(target)

```

tb3_1의 move_base노드의 goal을 이용하여 tb3_1의 목적지를 설정해준다.


target은 MoveBaseActionGoal객체를 생성하고 좌표를 저장한다.


터틀봇이 종료되지 않을동안 MoveBaseActionGoal객체에 좌표를 저장하여


pub.publish(target)을 이용하여 map에 목적지를 전달한다.
