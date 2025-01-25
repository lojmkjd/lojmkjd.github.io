---
title: 基于VMware+Ubuntu 20.04的PCL环境搭建
date: 2024-12-16 12:00:00 +0800
categories: [Leaning Notes, PointCloudLibrary]
tags: [Environment, Ubuntu]
---

# 基于VMware+Ubuntu 20.04的PCL环境搭建

在Ubuntu中进行环境搭建时，有两种方法：源码构建和二进制包安装（即预编译安装和包管理器安装）。源码构建可以开启一些特殊功能，比如GPU功能。

**注意：** 在**虚拟机**中无法使用GPU，因此在**后期**开发时，推荐使用Windows开发环境，或者安装双系统。当然，也可以使用Windows上的Docker开发，这个方法同样可以移植到标准的Ubuntu系统中。

## 安装依赖环境

首先，需要安装一些必要的依赖工具和库，这些是编译和运行PCL所必需的。

### 更新软件包列表

```
sudo apt-get update
```

### 安装必要的依赖包

1. 安装git，便于进行项目管理和拉取库文件；安装pkg-config，提高安装和编译的效率：

   ```
   sudo apt-get install git pkg-config
   ```

2. 安装编译器、更新linux的C标准库、安装测试单元库：

   ```
   sudo apt-get install build-essential linux-libc-dev libgtest-dev
   ```

3. 安装Cmake工具：

   ```
   sudo apt-get install cmake
   ```

### 安装官方列表中的依赖：

1. 使用apt-cache命令查看需要安装的依赖树：

   ```
   sudo apt-cache show libpcl-dev
   ```

   对应官网文档，发现需要安装的库为（带有xxx+dfsg-5ubuntu1的为我们需要自己编译的内容）：

   - libboost-all-dev
   - libeigen3-dev
   - libflann-dev
   - libvtk7-qt-dev
   - libqull-dev
   - libopenni-dev 、libopenni2-dev

2. 安装boost库：

   ```
   sudo apt-get install libboost-all-dev
   ```

3. 安装eigen库：

   ```
   sudo apt-get install libeigen3-dev
   ```

4. 安装flann库：

   ```
   sudo apt-get install libflann-dev
   ```

   注意：

   - libflann1.9是flann库的运行时版本，提供flann的程序代码；-dev结尾提供的是flann库的头文件，实际运用时，依赖其运行时库。
   - `libflann-dev` 只是一个开发包，它依赖于某个运行时版本（例如 `libflann1.9`），但如果系统已经安装了不同版本的运行时库，可能会导致版本不匹配的问题。
   - 在检查依赖时，若在depends中存在该库的运行时库，建议一起安装。

5. 安装vtk7库：

   ```
   sudo apt-get install libvtk7-qt-dev
   ```

6. 安装Qhull库：

   ```
   sudo apt-get install libqhull* 
   ```

   注意：libqhull依赖包含两个运行时库，并且l以ibqhull开头的全部库已全被依赖，所以使用libqhull*。

7. 安装libopenni库：

   ```
   sudo apt-get install libopenni-dev libopenni2-dev
   ```

### 安装额外的实用库（选装）

1. 安装硬件交互库，用于处理usb接口的传感器：

   ```
   sudo apt-get install libusb-1.0-0-dev libusb-dev libudev-dev
   ```

2. 安装并行计算库，提高大规模点云数据处理或并行计算速度：

   ```
   sudo apt-get install mpi-default-dev
   ```

3. 安装网络抓包库，用于抓取无线传感器的数据流和分析：

   ```
   sudo apt-get install libpcap-dev
   ```

4. 安装可视化组件，用于实现与图形界面、3D 渲染和可视化相关的功能：

   ```
   sudo apt-get install freeglut3-dev
   ```

5. 安装X11，用于实现可视化和用户交互功能，也可用于远程操控：

   ```
   sudo apt-get install libxmu-dev libxi-dev
   ```

6. 安装mono，使允许在Linux 上运行和开发 .NET 应用程序

   ```
   sudo apt-get install mono-complete
   ```

## 编译并安装PCL源码

参考文章 [编译安装PCL库](https://lojmkjd.github.io/posts/compiling-PCL-library/) ，选择适合版本的PCL库进行编译和安装。

## 构建安装脚本

将上述步骤封装到一个Shell脚本`install_pcl_dependences.sh`中，以便自动化安装。

```sh
#!/bin/

# 更新软件包列表
echo "更新软件包列表..."
sudo apt-get update -y

# 安装基本依赖工具
echo "安装基本依赖工具..."
sudo apt-get install -y git pkg-config build-essential linux-libc-dev libgtest-dev cmake cmake-gui

# 安装 PCL 依赖库
echo "安装 PCL 依赖库..."

# 安装Boost库
sudo apt-get install -y libboost-all-dev

# 安装Eigen库
sudo apt-get install -y libeigen3-dev

# 安装Flann库
sudo apt-get install -y libflann1.9 libflann-dev

# 安装VTK7库
sudo apt-get install -y libvtk7.1p vtk7 libvtk7-dev libvtk7.1p-qt libvtk7-qt-dev

# 安装Qhull库
sudo apt-get install -y libqhull*

# 安装OpenNI库
sudo apt-get install -y libopenni-dev libopenni2-dev

# 可选：安装额外的实用库
echo "安装额外的实用库..."

# 安装硬件交互库
sudo apt-get install -y libusb-1.0-0-dev libusb-dev libudev-dev

# 安装并行计算库（OpenMPI）
sudo apt-get install -y mpi-default-dev openmpi-bin openmpi-common

# 安装网络抓包库
sudo apt-get install -y libpcap-dev

# 安装可视化工具
sudo apt-get install -y freeglut3-dev

# 安装X11开发库
sudo apt-get install -y libxmu-dev libxi-dev

# 安装Mono (可选)
sudo apt-get install -y mono-complete

# 克隆PCL源代码
echo "克隆 PCL 源代码..."
mkdir ~/installation
cd ~/installation
git clone https://gitee.com/mirrors/pcl.git

# 切换到指定版本 v1.14.0
cd pcl
git checkout pcl-1.14.0

# 创建构建目录
echo "创建构建目录..."
mkdir release
cd release

# 配置CMake构建选项
echo "配置CMake构建..."
cmake -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DBUILD_GPU=ON \
      -DBUILD_apps=ON \
      -DBUILD_examples=ON ..

# 编译 PCL
echo "编译 PCL..."
make -j$(nproc)  # 使用所有CPU核心进行并行编译

# 安装编译后的程序
echo "安装编译后的 PCL..."
sudo make install

# 安装 pcl-tools（可选，提供PCL命令行工具）
echo "安装 pcl-tools..."
sudo apt-get install pcl-tools

# 测试 PCL 安装是否成功
echo "测试 PCL 安装是否成功..."
pcl_viewer ~/installation/pcl/test/pcl_logo.pcd

# 完成
echo "PCL 安装完成！"
```

#### 使用步骤

在工作文件夹中打开终端，运行以下命令来执行脚本：

```
sh ./install_pcl_dependences.sh
```

#### 注意事项

如果在Linux中运行Shell脚本时遇到如下错误：

```
-: ./xxx.sh: /bin/^M: bad interpreter: No such file or directory
```

该错误通常是由于Windows系统中编辑的文件包含不兼容的换行符导致的。Windows使用CRLF（回车+换行）作为行结束符，而Linux使用LF（换行）作为行结束符。

解决方法：

1. 使用`sed`命令删除回车符：

   ```
   sed -i 's/\r$//' xxx.sh
   ```

2. 在Linux中通过`touch`创建新脚本，并在Linux编辑器中修改：

   ```
   touch xxx.sh
   ```
