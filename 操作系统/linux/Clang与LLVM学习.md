# Clang与LLVM学习

## 一、简介

Clang 是一个 C、C++、Objective-C 和 Objective-C++ 编程语言的编译器前端，采用底层虚拟机（LLVM）作为后端。

LLVM ，全称为Low Level Virtual Machine，是以 BSD 许可来开发的开源的编译器框架系统，基于 C++ 编写而成，利用虚拟技术来优化以任意程序语言编写的程序的编译时间、链接时间、运行时间以及空闲时间，最早以 C/C++ 为实现对象，对开发者保持开放，并兼容已有脚本。

![Clang LLVM](C:\Users\ADMIN\Desktop\Clang与LLVM学习.assets\Clang-LLVM.jpg)

## 二、安装Clang 10

本次安装是在Ubuntu20.04上进行演示。

在Ubuntu 20.04下直接装不行，`sudo apt install clang` 会产生依赖问题。之所以产生依赖问题是因为没有安装源。在[LLVM官网](https://apt.llvm.org/)中，按照指示添加源至/etc/apt/sources.list（添加前最好备份一份）。

```shell
# i386 not available
deb http://apt.llvm.org/focal/ llvm-toolchain-focal main
deb-src http://apt.llvm.org/focal/ llvm-toolchain-focal main
# 11
deb http://apt.llvm.org/focal/ llvm-toolchain-focal-11 main
deb-src http://apt.llvm.org/focal/ llvm-toolchain-focal-11 main
# 12
deb http://apt.llvm.org/focal/ llvm-toolchain-focal-12 main
deb-src http://apt.llvm.org/focal/ llvm-toolchain-focal-12 main
```

添加源后，进行更新。

```shell
sudo apt-get update
```

更新完成之后就可以安装了。

```shell
# 先添加key
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -

# 安装clang最新的稳定版本
apt-get install clang-10 lldb-10 lld-10
```

三、使用

