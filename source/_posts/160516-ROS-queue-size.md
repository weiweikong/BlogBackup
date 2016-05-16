title: "[ROS] Choosing a queue_size - queue_size的大小的设置"
date: 2016-05-16 11:59:34
tags: ROS
---

## 一、函数定义
```c++
ros::Subscriber subscribe(const std::string& topic, uint32_t queue_size, <callback, which may involve multiple arguments>, const ros::TransportHints& transport_hints = ros::TransportHints());
```
- 主要讨论`queue_size`的影响。
- This is the incoming message queue size roscpp will use for your callback. 系统设置message queue的大小。
- If messages are arriving too fast and you are unable to keep up, roscpp will start throwing away messages. 一般而言，如果设置`queue_size=10`，即如果收到的数据大于10，则将开始抛弃最初收到的第一个数据。


## 二、设置多大合适
### 1. 设置为`0`
- A value of 0 here means an infinite queue, which can be dangerous. `queue_size`大小会影响内存的使用。
### 2. 设置为`1,2,3`
-  适用于10Hz的更新情况
- 设置为1，意味着系统总是使用最新发布的数据，only care about the latest measurement.

### 3. 设置大于`10`
- 系统更需要按顺序执行，例如digital_IO信号。


## Reference
http://wiki.ros.org/rospy/Overview/Publishers%20and%20Subscribers#Choosing_a_good_queue_size
