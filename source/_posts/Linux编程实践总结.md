---
title: Linux下c++运行总结
date: 2021-03-13 14:03:37
tags:
- C++
- Linux
categories: 笔记

---

## 1. Linux下C++编程实践总结

### 1.1 C++程序编译

1. 编写一个文本文件：helloSLAM.cpp

```c++
#include <iostream>
using namespace std;

int main()
{
	cout << Hello, SLAM ! << endl;
	return 0;
}
```

2. 将文本文件编译成可执行程序（g++）

```python
$ g++ helloSLAM.cpp
>>> a.out
```

3. 运行编译完成后的程序

```python
$ ./a.out
>>> hello, SLAM !
```

> g++基本语法：
>
> ```c++
> g++ [选项] 准备编译的文件 [选项] [目标文件]
> ```
>
> ```python
> $ g++ helloSLAM.cpp -o helloSLAM	# 将 helloSLAM 编译成名为 helloSLAM 的可执行文件
> ```
>
> 

### 1.2 cmake

由于g++一次只能编译一个程序，当在面对一个大型工程项目时，此种编译方法无疑会带来一个巨大的工作量，cmake会更加高效。在一个cmake工程中，会使用cmake命令生成一个makefile文件，然后用make命令根据makefile文件的内容编译整个工程。

```python
# fliename : CMakeLists.txt
# 声明要求的 cmake 最低版本
cmake_minimum_required( VERSION 2.8 )

# 声明一个cmake工程
project( helloSLAM )

# 添加一个可执行程序
# 语法：add_executable( 程序名 源代码文件 )
add_executable( helloSLAM helloSLAM.cpp )
```

cmake将会按照 CMakeLists.txt 文件生成一个makefile文件，使用make命令编译工程

```python
$ cmake .   # .表示在当前目录下进行cmake
>>> (finished)
$ make

$ ./helloSLAM
>>> Hello, SLAM !
```

cmake过程中会生成一些中间文件，当发布代码时，我们不希望中间文件一同发布出去，一个个删除太麻烦。更好的做法是将这些中间文件放在一个中间目录中，编译成功后把中间目录删除即可。更常见的做法是：

```python
$ mkdir build
$ cd build
$ cmake .. # 对上一文件夹进行编译
$ make
```

### 1.3 库文件

C++工程中，并不是所有的代码都会编译成可执行文件。只有main函数的文件才会生成可执行程序。另一部分代码，我们将其打包成一个东西，供其他程序调用。这个东西称为库（Library）。

```c++
// filename : libHelloSLAM.cpp
// 这是一个库文件
#include <iostream>
using namespace std;

void printHello()
{
	cout << Hello, SLAM ! << endl;
}
```

在 CMakeLists.txt 里添加如下内容：

```c
add_library( hello libHelloSLAM.cpp) // 将这个文件编译成一个名叫 “hello” 的库
    
>>> 生成一个 libhello.a 的文件
```

> Linux中库文件分为**静态库**和**动态库**，静态库以 .a 作为后缀，共享库以 .so 结尾。

生成共享库：

```C++
add_library( hello_shared SHARED libHelloSLAM.cpp)
    
>>> 生成一个 libhello_shared.so 的文件
```

在使用库文件之前，需要定义头文件。下面编写 libhello 的库文件：

```c++
// filename : libHelloSLAM.h
#ifndef LIBHELLOSLAM_H_
#define LIBHELLOSLAM_H_
// 防止重复引用头文件而引起重定义错误

void printHello();
#endif
```

调用库：

```c++
// filename : useHello.cpp
#include "libHelloSLAM.h"

int main()
{
	printHello();
	return 0;
}
```

在 CMakeLists.txt 里添加如下内容：

```c
add_executable( useHello useHello.cpp )
target_link_libraries( useHello hello_shared )
```



## 2. ubuntu创建桌面图标

首先，进入 applications 目录下，并创建一个 clion.desktop 文件,使用 gedit 进行编辑：

```c
cd /usr/share/applications/
    
sudo touch clion.desktop
    
sudo gedit clion.desktop
```

在文件中添加如下代码：

```c++
[Desktop Entry]
Encoding=UTF-8

Name=CLion

Comment=clion-2020.3.3

Exec=/home/gq/Application/clion-2020.3.3/bin/clion.sh

Icon=/home/gq/Application/clion-2020.3.3/bin/clion.svg

Categories=Application;Development;Java;IDE

Version=2020.3.3

Type=Application

#Terminal=1
```



