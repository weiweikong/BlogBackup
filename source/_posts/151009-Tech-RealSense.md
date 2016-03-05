title: Tech.001.Intel RealSense R200 Testing
date: 2015-10-09 22:34:07
tags: Hardware, RealSense
---
## 1. 物理属性
![image](http://7xlos6.com1.z0.glb.clouddn.com/151009.1.RealSense.jpg)

- 35g
- 长度
102x9.5x3.8mm

## 2. 基本原理
- 参考Intel官方网站（https://software.intel.com/en-us/articles/realsense-r200-camera）， 其基本结构如下图所示
![image](https://software.intel.com/sites/default/files/managed/d0/5d/R200.png)
- 组成
    - The R200 actually has 3 cameras providing **RGB (color) and stereoscopic IR** to produce depth.
    - The inside range is approximately 0.5-3.5 meters and an outside range up to 10 meters.
 
<!--more-->

## 3. 测试视频
- 从室内到室外，测试各个传感部件基本情况

<embed src="http://player.youku.com/player.php/sid/XMTM1NTk0ODUyOA==/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>



## 4. Windows环境测试
- 下载驱动和SDK（1.2G）
- 执行Intel RealSense SDK Sample Browser，参考相关例子，即可执行测试文件。

## 5. Linux环境
- 针对R200，目前官方给出的节点为
https://github.com/PercATI/RealSense_ROS
但目前，运行过程中，针对z轴数据有报错，且图像效果与Windows环境下略有差别。