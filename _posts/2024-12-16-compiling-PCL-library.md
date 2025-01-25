---
title: 编译安装PCL库
date: 2024-12-16 12:00:00 +0800
categories: [Leaning Notes, PointCloudLibrary]
tags: [Compile, Install]
---

# 编译安装PCL库

## 获取PCL源码

从Github或者镜像站点克隆PCL库的源码：

```bash
# 克隆 PCL 仓库
git clone https://gitee.com/mirrors/pcl.git
```

## 编译PCL源码

进入克隆后的项目文件夹，进行配置和编译。

```bash
# 进入项目文件夹
cd pcl

# 切换到指定版本(如1.14.0)
git checkout pcl-1.14.0

# 创建构建目录
mkdir release
cd release

# 配置 cmake 构建
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_GPU=ON -DBUILD_apps=ON -DBUILD_examples=ON ..

# 编译
make -j$(nproc)  # 使用所有CPU核心进行并行编译
```

说明：关于cmake命令详解，见笔记文档

`cmake -D` 用于定义编译文件`CMakeLists.txt`中变量的值，例如：

- -DCMAKE_BUILD_TYPE=Release：指定编译版本为优化后的发布版本
- -DCMAKE_INSTALL_PREFIX=/usr/local：指定安装路径为/usr/local
- -DBUILD_GPU=ON：启用GPU加速（编译GPU库）
- -DBUILD_apps=ON：编译自带的工具集
- -DBUILD_examples=ON：编译示例程序

## 安装编译后的程序

执行以下命令来安装编译后的程序：

```bash
# 安装编译后的程序
make install
```

## 测试安装是否成功

安装PCL工具包，方便在终端使用PCL程序：

```bash
# 安装 pcl-tools
apt-get install pcl-tools
```

在PCL目录下的`test`文件夹中，有`pcd`文件用于验证：

```bash
# 测试安装是否成功
pcl_viewer ../test/pcl_logo.pcd
```

如果正常显示PCL Logo图案，则安装成功。