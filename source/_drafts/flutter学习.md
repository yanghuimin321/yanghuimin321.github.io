---
title: flutter学习
typora-root-url: ./flutter学习
date: 2026-11-03 16:19:48
tags:
categories:
description:
---

# dart学习

1. 通过cli方式(dart create cli)生成的初始dart应用文件：
  - .dart_tool/ 
  - bin/    :存放可执行代码的地方，cli.dart也是应用程序的入口点
  - lib/
  - test/
  - analysis_options.yaml
  - pubspec.lock
  - pubspec.yaml  : 管理Dart项目的元数据、依赖项（例如http包）和资源
2. 命令行命令：
  - dart create [参数] : 生成基本Dart项目，会设置所需的必要文件和目录
  - dart create -t package [参数] : 
  - dart run : 运行默认应用程序
  - dart bin/xxx.dart [参数] : 运行应用程序
  - dart pub get : 获取新添加依赖项，使其可供项目使用（类似npm install；dart pub是dart/flutter生态的包管理工具）
3. 每个 Dart 程序都是从它的main函数开始执行的
4. 强类型语言，有类型校验
5. 变量：
   1. Dart不会为非可空类型设置初始值，可空变量默认初始化为null。非空变量虽然在声明变量时不必初始化变量，但是在使用之前需要为其赋值/初始化。不能在可空类型的表达式上访问属性和方法。
   2. 顶级变量和类变量是延迟初始化的，它们会在第一次被使用时再初始化
   3. late : 声明一个非空变量，但不在声明时初始化。 延迟初始化一个变量。 
   4. final和const：final开始可以不赋值，只能赋值一次；final不仅有const的编译时常量的特性，最重要的是它是运行时常量，并且final是惰性初始化， 即在运行时第一次使用前才初始化
   5. _ : 通配符变量。声明了一个不作为可用变量的局部变量或参数，本质上是一个占位符（不适用于顶层声明或类的成员）
6. 
