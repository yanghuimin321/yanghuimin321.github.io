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
5. 每个变量都指向一个对象(类的实例)，因此通常可以使用构造函数来初始化变量
6. 变量：
   1. Dart不会为非可空类型设置初始值，可空变量默认初始化为null。非空变量虽然在声明变量时不必初始化变量，但是在使用之前需要为其赋值/初始化。不能在可空类型的表达式上访问属性和方法。
   2. 顶级变量和类变量是延迟初始化的，它们会在第一次被使用时再初始化
   3. late : 声明一个非空变量，但不在声明时初始化。 延迟初始化一个变量。 
   4. final和const：final开始可以不赋值，只能赋值一次；final不仅有const的编译时常量的特性，最重要的是它是运行时常量，并且final是惰性初始化， 即在运行时第一次使用前才初始化
   5. _ : 通配符变量。声明了一个不作为可用变量的局部变量或参数，本质上是一个占位符（不适用于顶层声明或类的成员）
7. 数据类型：
   1. Numbers：int、double（都属于num类型的子类型，都可以使用abs,ceil等方法）（可以使用_来作为数字分隔符，提高可读性，如.000_000_000_01）
   2. Strings：String
   3. Booleans：bool
   4. Records(value1, value2)：匿名、不可变、聚合、固定大小、异构性、带有类型
   5. Collections: List/Set/Map
      1. 注册空Set时需要指定类型如<String>{}，不然单{}默认是Map
      2. Set是没有顺序且不能重复的集合，所以不能通过索引去获取值
   6. Functions
8. 类型：Runes(characters)(可以表现字符串的Unicode码位)、Symbols、Object、Enum、Future/Stream、Iterable、Never、dynamic、void
9. 闭包：
   1.  全局变量特点:    全局变量常驻内存、全局变量污染全局
   2.  局部变量的特点：  不常驻内存会被垃圾机制回收、不会污染全局  
   3.  闭包: 函数嵌套函数, 内部函数会调用外部函数的变量或参数, 变量或参数不会被系统回收(不会释放内存), 实现：常驻内存但不污染全局
10. 
