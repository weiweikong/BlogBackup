title: "[Gazebo][ROS] Gazebo with ROS control package"
date: 2016-05-29 08:19:19
tags: ROS
---

## Reference
1. ROS by Example -rrbot例子
2. Gazebo自带例子
2.1 Gazebo与ROS控制接口的源码
https://github.com/ros-simulation/gazebo_ros_pkgs
2.2 Gazebo与ROS控制接口例程的源码
https://github.com/ros-simulation/gazebo_ros_demos


## 1. Gazebo与ROS Control

###1.1 最少必要知识

http://gazebosim.org/tutorials/?tut=ros_control
- `gazebo_ros_control`只支持URDF格式的Model（2016.05.28）
- 在`ROS Jade`版本下，需要下载`gazebo_ros_control`和`ros control`的源码来确保编译过程有效。（2016.05.28）
- URDF格式以及对`xacro`的文件能够看懂，特别是里面的函数调用
- 
###1.2 PID控制的调试方法

- 使用`rqt_gui`里面的`dyanmic configuration`

###1.3 对Joint进行控制的例子

- Step 1: 明确对哪一个Joint进行控制并配置属性
    - 这里选择`joint1`
    - 注意其属性配置为`continuous`
```
  <joint name="joint1" type="continuous">
    <parent link="link1"/>
    <child link="link2"/>
    <origin xyz="0 ${width} ${height1 - axel_offset}" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
    <dynamics damping="0.7"/>
  </joint>
```

- Step 2: 在URDF中添加Gazebo中的Plugin
    - 添加`.so`文件
    - 注意Namespace的命名统一
```
<gazebo>
  <plugin name="gazebo_ros_control" filename="libgazebo_ros_control.so">
    <robotNamespace>/MYROBOT</robotNamespace>
    <robotSimType>gazebo_ros_control/DefaultRobotHWSim</robotSimType>
  </plugin>
</gazebo>
```

- Step 3: 设置Transmission的内容
    - 明确转换关系
```
  <transmission name="tran1">
    <type>transmission_interface/SimpleTransmission</type>
    <joint name="joint1">
      <hardwareInterface>EffortJointInterface</hardwareInterface>
    </joint>
    <actuator name="motor1">
      <hardwareInterface>EffortJointInterface</hardwareInterface>
      <mechanicalReduction>1</mechanicalReduction>
    </actuator>
  </transmission>
```

- Step 4: `yaml`文件的配置
    - 允许系统输出Joint State
    - 明确控制关节Joint 1，控制类型为Position，给定PID初始参数
```
rrbot:
  # Publish all joint states -----------------------------------
  joint_state_controller:
    type: joint_state_controller/JointStateController
    publish_rate: 50 
 
  # Position Controllers ---------------------------------------
  joint1_position_controller:
    type: effort_controllers/JointPositionController
    joint: joint1
    pid: {p: 100.0, i: 0.01, d: 10.0}
```

- Step 5: `launch`文件的修改
    - 主要是读取`yaml`文件内容
    - 通过`controller_manager`这个package，来生成`position controller`和`state controller`。
    - 通过`robot_state_publisher`这个package，来监视state的状态
```
<launch> 
  <!-- Load joint controller configurations from YAML file to parameter server -->
  <rosparam file="$(find rrbot_control)/config/rrbot_control.yaml" command="load"/>
 
  <!-- load the controllers -->
  <node name="controller_spawner" pkg="controller_manager" type="spawner" respawn="false"
    output="screen" ns="/rrbot" args="joint1_position_controller joint2_position_controller joint_state_controller"/>
 
  <!-- convert joint states to TF transforms for rviz, etc -->
  <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher"
    respawn="false" output="screen">
    <remap from="/joint_states" to="/rrbot/joint_states" />
  </node> 
</launch>

```
- Step 6: 通过命令行测试关节运动情况
```
rostopic pub -1 /rrbot/joint1_position_controller/command std_msgs/Float64 "data: 1.5"
```

### 1.4 robot_state_publisher的主要功能

- 发布机器人的`tf`信息
- 主要针对URDF格式

## 2. Gazebo with Sensor

TODO
## 3. URDF与SDF

- <xmind>
- <Mastering ROS for Robotics - Chp.2 Working with 3D Robot Modeling in ROS>
- SDF转换到URDF
https://github.com/andreasBihlmaier/pysdf


