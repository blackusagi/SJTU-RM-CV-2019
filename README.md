# 上海交通大学 RoboMaster 2019赛季 视觉代码
本代码是上海交通大学RoboMaster2019赛季步兵车辆的视觉部分，分为三个模块:**装甲板识别**，**能量机关**，以及**封装的设备驱动和配置文件**。可以提取能量机关以外的模块并修改main函数直接作为哨兵识别代码。 

本代码统一使用640×480大小的图像进行处理

作者：自瞄（唐欣阳，卫志坤），能量机关（孙加桐，罗嘉鸣）。

运行效果：自瞄帧率120（摄像头最大帧率）,识别距离根据环境不同大约8米左右(5mm焦距镜头)。

## 一、代码运行环境

| 硬件设备                                             | 操作系统                                     | 运行库                                                       | ToolChain                                                  |
| ---------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| IntelNUC<br />MindVision工业相机×１<br />USB转TTL×１ | Ubuntu18.04<br />Ubuntu16.04<br />Ｗindows10 | OpenCV3.4.5<br />OpenCV_contrib3.4.5<br />Eigen3<br />MindVision相机驱动 | Ubuntu18/16 : cmake3+gcc7+g++7 <br />Win10 : cmake3+VS2019 |

实际装载在步兵和哨兵上的运行环境为Ubuntu18.04。  

相机驱动下载地址：[相机驱动](https://www.mindvision.com.cn)

OpenCV下载地址：[OpenCV](https://github.com/opencv)

OpenCV安装教程 : [linux](https://docs.opencv.org/3.4.5/d7/d9f/tutorial_linux_install.html)  [Windows](https://docs.opencv.org/3.4.5/d3/d52/tutorial_windows_install.html)

Eigen下载方法：
* Ubuntu16/18: ```sudo apt install libeigen3-dev```
* Windows10 : [Eigen下载地址](http://eigen.tuxfamily.org/)

## 二、程序编译运行方式
### Ubuntu16/18（在项目文件夹下）
```shell
mkdir build
cd build
cmake ..
make -j8
sudo ./run
```

### Windows10
打开cmake-gui，选择项目文件夹和build文件夹，生成VS工程。在VS中编译项目。
### 命令行参数
```./run --help```可以查看所有命令行参数及其作用。

## 三、文件目录结构
```
.
├── armor                       // 存放自瞄主要算法代码
│   ├── include                 // 自瞄头文件
│   └── src                     // 自瞄源码
├── CMakeLists.txt              // cmake工程文件
├── energy                      // 存放能量机关主要算法代码
│   ├── include                 // 能量机关头文件
│   └── src                     // 能量机关源码
├── main.cpp                    // 主函数
├── others                      // 存放摄像头、串口、配置文件等
│   ├── include                 // others头文件
│   ├── libmvsdk.dylib          // mac相机驱动链接库
│   ├── libMVSDK.so             // linux相机驱动链接库
│   ├── MVCAMSDK_X64.dll        // win10相机驱动链接库
│   ├── MV-UB31-Group0.config   // 相机配置文件
│   └── src                     // others源码
└── tools                       // 存放分类器训练代码及参数，自启动脚步等
    ├── auto-pull.sh            // 自动代码更新脚本
    ├── create-startup.sh       // 自启动文件创建脚本
    ├── monitor.bat             // win10进程守护脚本
    ├── monitor.sh              // linux进程守护脚本
    ├── para                    // 分类器参数
    └── TrainCNN                // 分类器训练源码
```
## 四、程序运行基本流程

　　　　　　　　　　　　↗  大能量机关 ↘

各项初始化→读取当前状态  → 小能量机关  → 回到读取状态

　　↓　　　　　　　　　↘　　自瞄　　↗

数据接收线程

## 五、识别方式

### 1.自瞄装甲板识别方式

​    首先对图像进行通道拆分以及二值化操作，再进行开闭运算，通过边缘提取和条件限制得出可能为灯条的部分。再对所有可能的灯条进行两两匹配，根据形状大小特性进行筛选，得出可能为装甲板的候选区。然后把所有候选区交给分类器判断，得出真实的装甲板及其数字id。最后根据优先级选取最终击打目标以及后续处理。

### 2.能量机关识别方式

​    首先对图像进行二值化操作，然后进行一定腐蚀和膨胀，通过边缘提取和条件限制得出待击打叶片（锤子形）。在待击打叶片范围内进一步用类似方法寻找目标装甲板和流动条，在二者连线上寻找中心的“R”。根据目标装甲板坐标和中心坐标计算极坐标系下的目标角度，进而预测待击打点的坐标（小符为装甲板本身，大符需要旋转）。最后将待击打点坐标和图像中心的差值转换为yaw和pitch轴角度，增加一环PID后发送给云台主控板。

## 六、代码命名规范

函数名：使用首字母小写的驼峰命名法

类型名：使用首字母大写的驼峰命名法

变量名：使用下划线分割命名法

