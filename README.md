> 下面简要介绍一下从拿到MID-360到跑通一个Mapping算法获取`.pcd`文件的大致流程。可能会有错漏，请多多包涵。

## 1. 安装并连接LiDAR到电脑

这部分请参考[Mid-360 产品用户手册](https://terra-1-g.djicdn.com/851d20f7b9f64838a34cd02351370894/livox%20mid%20360%20%E7%94%A8%E6%88%B7%E4%BD%BF%E7%94%A8%E6%89%8B%E5%86%8C240222/Livox_Mid-360_User_Manual_CHS.pdf)并结合本文内容配置，电脑系统使用Ubuntu 22.04 LTS。

- 请找到MID-360专用的一分三数据线，为LiDAR供电以及数据传输。你需要提供DC 9-27V的电源并将线缆连接到电脑的RJ45网口上。（本文使用LiDAR与电脑直接相连的配置，若有将雷达通过交换机接入车内局域网的需求，则可能需要更改雷达的IP以适配网段，请自行查阅手册。）
- 在电脑中找到与LiDAR连接的Interface，并且将其IP设置为192.168.1.1，子网掩码为24（255.255.255.0），网关也为192.168.1.1。目前MID-360的IP为192.168.1.3，重启这个Interface即可连接到LiDAR。

*由于当前LiDAR不在我手中，此部分依照记忆编写，若遇到问题可以在飞书或微信联系我。

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
   cd ./MID-360_materials/Livox-SDK2/
   mkdir build
   cd build
   cmake .. && make -j
   sudo make install
   ```


# 页面施工中......
