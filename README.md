> 下面简要介绍一下从拿到MID-360到跑通一个Mapping算法获取`.pcd`文件的大致流程。可能会有错漏，请多多包涵。

***请注意本仓库中需要电脑/上位机使用Ubuntu 22并安装了ROS2（Humble）。**

## 1. 安装并连接LiDAR到电脑

这部分请参考[Mid-360 产品用户手册](https://terra-1-g.djicdn.com/851d20f7b9f64838a34cd02351370894/livox%20mid%20360%20%E7%94%A8%E6%88%B7%E4%BD%BF%E7%94%A8%E6%89%8B%E5%86%8C240222/Livox_Mid-360_User_Manual_CHS.pdf)并结合本文内容配置。

- 请找到MID-360专用的一分三数据线，为LiDAR供电以及数据传输。你需要提供DC 9-27V的电源并将线缆连接到电脑的RJ45网口上。（本文使用LiDAR与电脑直接相连的配置，若有将雷达通过交换机接入车内局域网的需求，则可能需要更改雷达的IP以适配网段，请自行查阅手册。）
- 在电脑中找到与LiDAR连接的Interface，并且将其IP设置为192.168.1.50，子网掩码为24（255.255.255.0），网关为192.168.1.1。**目前MID-360的IP为192.168.1.3**，重启这个Interface即可连接到LiDAR。

***由于当前LiDAR不在我手中，此部分依照记忆编写，若遇到问题可以在飞书或微信联系我。**

## 2. 安装LiDAR驱动（Livox-SDK2）

这部分请参考[Livox-SDK2_README](https://github.com/Livox-SDK/Livox-SDK2/blob/6a940156dd7151c3ab6a52442d86bc83613bd11b/README.md)并结合本文内容配置。

1. 请确保系统中安装了`cmake`（通常Ubuntu22中已包含）。

   ```bash
   sudo apt install cmake
   ```

2. 将本步骤以及后续所需的所有文件下载到本地。

   ```bash
   git clone https://github.com/Lya-M1RA/MID-360_materials.git --recursive
   ```

   注意务必添加`--recursive`参数，这样才能将仓库中包含的所有submodule下载下来。

3. 接着开始编译并安装Livox-SDK2。

   ```bash
   cd <repo_root_folder>/Livox_SDK2/
   mkdir build
   cd build
   cmake .. && make -j
   sudo make install
   ```

至此，MID-360的驱动已安装完成。

## 3. 安装LiDAR ROS2驱动（Livox-ROS-Driver2）

这部分请参考[Livox-ROS-Driver2_README](https://github.com/Livox-SDK/livox_ros_driver2/blob/b6ff7d1c8b210a96b74e919823771e1e32249758/README.md)并结合本文内容配置。

1. 根据LiDAR的网络参数修改驱动配置文件。

   ```bash
   cd <repo_root_folder>/Livox_ROS_Driver2/src/Livox_ROS_Driver2/config
   vim MID360_config.json
   ```

   修改Host IP与LiDAR IP，请将文件中几个有注释的位置根据实际连接修改为对应的IP。

   ```json
   {
     "lidar_summary_info" : {
       "lidar_type": 8
     },
     "MID360": {
       "lidar_net_info" : {
         "cmd_data_port": 56100,
         "push_msg_port": 56200,
         "point_data_port": 56300,
         "imu_data_port": 56400,
         "log_data_port": 56500
       },
       "host_net_info" : {
         "cmd_data_ip" : "192.168.1.5",	# host ip
         "cmd_data_port": 56101,
         "push_msg_ip": "192.168.1.5",		# host ip
         "push_msg_port": 56201,
         "point_data_ip": "192.168.1.5",	# host ip
         "point_data_port": 56301,
         "imu_data_ip" : "192.168.1.5",	# host ip
         "imu_data_port": 56401,
         "log_data_ip" : "",
         "log_data_port": 56501
       }
     },
     "lidar_configs" : [
       {
         "ip" : "192.168.1.12",			# lidar ip
         "pcl_data_type" : 1,
         "pattern_mode" : 0,
         "extrinsic_parameter" : {
           "roll": 0.0,
           "pitch": 0.0,
           "yaw": 0.0,
           "x": 0,
           "y": 0,
           "z": 0
         }
       }
     ]
   }
   ```

2. 编译Livox ROS驱动。

   ```bash
   cd <repo_root_folder>/Livox_ROS_Driver2/src/Livox_ROS_Driver2/
   source /opt/ros/humble/setup.sh
   ./build.sh humble
   ```

3. 测试MID-360通讯。

   ```bash
   source <repo_root_folder>/Livox_ROS_Driver2/install/setup.bash
   ros2 launch livox_ros_driver2 rviz_MID360_launch.py
   ```

   你应当能在RViz界面中看到MID-360的实时点云了。

## 4. 使用Fast-LIO

1. 编译Fast-LIO。

   ```bash
   source <repo_root_folder>/Livox_ROS_Driver2/install/setup.bash
   cd <repo_root_folder>/FAST_LIO_ROS2/
   colcon build --symlink-install
   ```

2. 使用Fast-LIO采集地图。

   ```bash
   source <repo_root_folder>/FAST_LIO_ROS2/install/setup.bash
   # 运行LiDAR ROS驱动
   ros2 launch livox_ros_driver2 msg_MID360_launch.py
   # 运行Fast-LIO
   ros2 launch fast_lio mapping.launch.py 
   ```

在关闭程序后，你应当能在`<repo_root_folder>/FAST_LIO_ROS2/src/Livox_ROS_Driver2/PCD/`中看到刚才保存的`.pcd`地图。
