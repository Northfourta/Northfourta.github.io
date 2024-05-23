---
title: Linux下clion安装第三方库
date: 2022-05-19 14:03:37
tags:
- C++
- Linux
categories: 教程

---

## Ubuntu下安装eigen3

### 1. 使用命令行安装（**方法一**）

```c
sudo apt-get install libeigen3-dev
```

### 2.采用源码安装（**方法二**）

进入[eigen](https://eigen.tuxfamily.org/index.php?title=Main_Page)网址，下载对应版本源码压缩包（zip / targz），并解压。

由于eigen只由头文件构成，因此它不需要编译；使用cmake进行安装：

```c
cd eigen-3.3.9
mkdir build
cd build
cmake ..
sudo make install
// 默认安装目录在（/usr/include/eigen3）
```

clion下 CMakeLists.txt 配置文件写法：

```python
cmake_minimum_required(VERSION 3.17)
project(eigenMatrix)

set(CMAKE_CXX_STANDARD 14)

#find_package (Eigen3 3.3.9 REQUIRED)

include_directories("//usr//include//eigen3")
add_executable(eigenMatrix main.cpp)

#target_link_libraries (eigenMatrix Eigen3::Eigen)
```

### 3. eigen语法

- 定义矩阵：`Matrix<dtype, matrix_size, matrix_size> var`
- 定义向量（3x1）：`Vector3d var`等同于`Matrix<double,3,1> matrix_NN`
- 定义方阵（3x3）：`Matrix3d var`
- 随机数矩阵：`Matrix3d::Random()`
- 零矩阵：`Matrix3d::Zero()`



```c++
// example
#include <iostream>
#include <Eigen/Core>

using namespace std;
using namespace Eigen;

int main(){
    Matrix<float,2,3> matrix_23;
    
    return 0;
}
```



