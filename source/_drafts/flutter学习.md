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
9. 函数传参：位置参数（必选()/可选[]），命名参数（可选{}/必选required）
10. Record是一种独立的数据类型：位置字段，命名字段（作为参数被传时实际是一个参数）
11. 闭包：
   1.  全局变量特点:    全局变量常驻内存、全局变量污染全局
   2.  局部变量的特点：  不常驻内存会被垃圾机制回收、不会污染全局  
   3.  闭包: 函数嵌套函数, 内部函数会调用外部函数的变量或参数, 变量或参数不会被系统回收(不会释放内存), 实现：常驻内存但不污染全局
12. 面向对象编程(OOP)的三个基本特征是：封装、继承、多态
   1.  Dart所有的东西都是对象，所有的对象都继承自Object类。
   2.  Dart是一门使用类和单继承的面向对象语言，所有的对象都是类的实例，并且所有的类都是Object的子类
   3.  自定义类：默认构造函数，命名构造函数（构造函数前的**初始化列表**，如：Rect([int height=2, int width=10]):this.height=height,this.width=width{}）
   4.  私有方法/属性： 使用_(代表这个成员只在当前库可见，所以需要通过import的方式来实现私有) 
   5.  getter/setter
   6.  static：静态成员(类级别的变量/函数)，静态方法不能访问非静态成员，非静态方法可以访问静态成员
   7.  继承extends：
       1.  子类会继承父类里面可见的属性和方法 但是不会继承构造函数（通过super调用父类构造函数）、static静态成员、非同一个库的_成员
       2.  子类能重写父类的方法、getter和setter（@override）
       3.  子类通过 super.父类方法名 调用父类方法
   8.  抽象类abstract：
       1.  抽象类不能被实例化，只有继承它的子类可以
       2.  抽象类中没有方法体的方法称为抽象方法，子类继承抽象类必须实现抽象方法，才能称为可实例化的具体类（除非子类也是抽象类）
       3.  
   9.  多态：（抽象类经常用来实现多态）
       1.  将子类型指针赋值给父类型变量，同一个函数调用会有不同执行效果（子类的实例赋值给父类的引用）
       2.  父类定义一个方法不去实现，让继承他的子类去实现，每个子类有不同的表现
   10. 实现implements：
       1.  extends继承父类已有的实现，可以直接使用父类的属性和方法；只能extends一个类；
       2.  implements实现接口，需要自己实现接口要求的所有实例成员，不能直接使用被实现类的方法实现；可以implements多个接口；普通类抽象类都可以作为接口被实现；
   11. 
13.  
