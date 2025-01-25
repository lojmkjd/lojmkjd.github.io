---
title: 基于Docker Desktop构建PCL环境
date: 2025-1-24 12:00:00 +0800
categories: [Leaning Notes, PointCloudLibrary]
tags: [Environment, Docker, Windows10]
---

# 基于Docker Desktop构建PCL环境

## 安装Docker Desktop

参考本篇文章，在Windows10系统上打开虚拟化和安装Docker Desktop，完成基础的条件构建。

## 构建PCL环境

### 方式1：从官网拉取镜像

#### 查询镜像信息

官方镜像地址：[PointCloudLibrary/env - Docker Image | Docker Hub](https://hub.docker.com/r/pointcloudlibrary/env?uuid=CE4C60D4-C451-4E37-B93F-B818995339C8)

从官方 [Dockerfile 文件](https://github.com/PointCloudLibrary/pcl/blob/master/.dev/docker/env/Dockerfile) 可知，该镜像特别优化了与两家公司（Ensenso 和 Intel RealSense）硬件的兼容性。

如果你使用的是这两家公司的设备，推荐使用官方镜像。

#### 拉取镜像

```powershell
docker pull pointcloudlibrary/env:20.04
```

`docker pull`具体用法参考 [Docker官方CLI文档](https://docs.docker.com/reference/cli/docker/) 。

### 方式2：从Dockerfile构建镜像

参考官方  [Dockerfile 文件](https://github.com/PointCloudLibrary/pcl/blob/master/.dev/docker/env/Dockerfile)  和 [另一位的Dockerfile文件](https://github.com/Alraemon/PointCloudLibrary-Docker/blob/main/Dockerfile) ，编写了自定义的 [Dockerfile文件](https://github.com/lojmkjd/PointCloudLibrary-Docker/blob/main/Dockerfile) ，该文件的具体使用说明参考[lojmkjd/PointCloudLibrary-Docker](https://github.com/lojmkjd/PointCloudLibrary-Docker/tree/main) 。

## 编译并安装PCL库

参考文章 [编译安装PCL库](https://lojmkjd.github.io/posts/compiling-PCL-library/) ，选择适合版本的PCL库进行编译和安装。
