---
title: js高级程序设计-第四版-读书笔记
typora-root-url: ./js高级程序设计-第四版-读书笔记
tags:
  - html
  - css
  - javascript
  - 前端
categories:
  - 读书笔记
description: 《javascript高级程序设计(第四版)》笔记记录
date: 2025-07-26 15:07:03
---

# 1. 什么是JavaScript

完整的JavaScript实现包含三个部分：

- **核心（ECMAScript）**：语言的标准规范
  - Web浏览器只是ECMAScript实现可能存在的一种宿主环境（host environment）
  - 宿主环境提供ECMAScript的基准实现和与环境自身交互必需的扩展
  - 其他宿主环境：服务器端JavaScript平台Node.js
- **文档对象模型（DOM）**：是一个应用编程接口(API)，用于在HTML中使用扩展的XML
- **浏览器对象模型（BOM）**：是一个应用编程接口(API)，用于支持访问和操作浏览器的窗口



# 2. 在HTML中使用JavaScript

1. \<script\>元素8个**属性**：

   - **charset**：表示通过src属性指定的代码的字符集
   - **crosssorigin**：配置相关请求的CORS(跨源资源共享)设置，默认不使用CORS。
     - crossorigin="anonymous"：配置文件请求不必设置凭据标志
     - crossorigin="use-credentials"：设置凭据标志，意味着出站请求会包含凭据
   - **integrity**：允许比对接收到的资源和指定的加密签名以验证子资源完整性(SRI,Subresource Integrity)。
     - 可用于确保内容分发网络(CDN,Content Delivery Network)不会提供恶意内容。
   - **language**：废弃。
   - **src**：表示要执行的代码的外部文件
   - **type**：代替language，表示代码块中脚本语言的内容类型(MIME类型)
     - 按照惯例值始终是：text/javascript
     - 如果值为module，则代码会被当成ES6模块（只有这时代码中才能出现import/export关键字）
   - **async**：异步加载脚本
   - **defer**：延迟加载脚本

2. \<script\>元素**使用**：

   - 页面中嵌入行内代码
   - 引入外部JavaScript代码（建议使用，优点：可维护性、可缓存、适应XHTML）

3. 脚本加载：

   | defer                                          | async                                                        | 动态加载脚本(向DOM中动态加入script元素)                      |
   | ---------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | 延迟加载                                       | 异步加载                                                     | 相当于添加async属性                                          |
   | 按照先后顺序执行(但实际不一定)                 | 不保证按顺序执行                                             |                                                              |
   | 都在DOMContentLoaded事件之前执行(但实际不一定) | 保证在页面load事件前执行，但可能在DOMContentLoaded之前或之后 |                                                              |
   | 把延迟脚本放在页面底部是最好选择               | 异步脚本之间互不依赖很重要，以及脚本不要在加载期间修改DOM    | 对浏览器预加载器不可见，会影响它们在资源获取队列中的优先级（可通过在文档头部显式声明避免：<link rel="preload" href="template.js"> |
   | 加载完所有脚本再按顺序执行onload事件           | 加载完单个脚本立即执行对应onload事件                         |                                                              |

4. 文档模式：可以使用**DOCTYPE**切换文档模式 

   - 混杂模式、标准模式、准标准模式

5.  **\<noscript\>** 元素，使用场景：

   - 浏览器不支持脚本
   - 浏览器支持脚本，但脚本被禁用



# 3. 语言基础

- **严格模式："use strict";**

## 3.1 变量

| var                                           | let                                                          | const                                                        |
| :-------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 函数作用域                                    | 块作用域                                                     | 同let                                                        |
| 声明提升：自动提升到函数作用域顶部            | 严格来说也声明提升，但因为“暂时性死区”，实际上不能在声明之前使用变量 | 同let，声明变量的同时必须初始化，且不能再重新赋予新值        |
| 可重复声明                                    | 不可重复声明                                                 | 同let                                                        |
| 省略var操作符，定义成全局变量(严格模式下报错) | 不能省略操作符，报错                                         | 同let                                                        |
| 全局作用域中声明的变量会变成window对象的属性  | 不会变成window对象的属性                                     | 同let                                                        |
|                                               |                                                              | 声明的限制只适用于它指向的变量的引用，因此可以改变对象内部的属性（如果想让整个对象都不能修改，可以使用**Object.freeze()**，再给属性赋值时不报错但静默失败） |

## 3.2 数据类型

- 简单数据类型：**Undefined**、**Null**、**Boolean**、**Number**、**String**、**Symbol**
- 复杂数据类型：**Object**
- 类型检测：
  - **typeof**操作符：**可以用于判断原始值类型**
    - undefined(值未定义或未变量声明)、boolean(布尔值)、string(字符串)、number(数值)、object(对象或null)、function(函数)、symbol(符号)
    - null被认为是一个空对象的引用，因此typeof返回的是object
    - 对未声明的变量，只能执行typeof操作（其他操作会报错）
  - **instanceof**操作符：**用于判断值的引用类型**

1. **Boolean类型**：
   - 转型函数**Boolean()**：空字符串、0、NaN、null、undefined(转换为false)
2. **Number类型**：使用IEEE754格式表示（**浮点数计算精度缺失**）
   - 数值范围：
     - 最小数值**Number.MIN_VALUE**(一般5e-324)
     - 最大数值**Number.MAN_VALUE**(一般1.7976931348623157e+308)
     - 超出范围自动转换成**±Infinity**(判断是否为有穷值，可用 **isFinite()** 函数)
     - **Number.NEGATIVE_INFINITY**,**Number.POSITIVE_INFINITY**分别保存±Infinity
   - **NaN(**Not a Number)：用于表示本来要返回数值的操作失败了（而不是抛出错误）
     - 任何涉及NaN的操作，都返回NaN；0除以0
     - NaN与任何值都不相等，包括NaN本身
     - **isNaN()** 函数
   - 数值转换：
     - **Number()**函数：
       - null -> 0 , undefined -> NaN, 空字符串 -> 0, 非数值字符串 -> NaN
       - 对象 -> 先调用valueOf()方法，再按照规则转换，如果结果是NaN，再调用toString()方法
     - **parseInt()**函数：第二个参数代表转换所用进制(默认十进制)
     - **parseFloat()**函数：只能解析十进制
3. **String类型**：
   - 字符串一旦创建，值就不能改变
   - 转型函数 **toString()** ：可以传参，表示进制（null和undefined没有toString方法）
   - 转型函数 **String()**：在toString的基础上，也支持转化null、undefined
   - 模板字面量(``)：保留换行字符，可以跨行定义字符串
   - 模板字面量标签函数(tag function)： **fn\`${a}+${b}=${a+b}\` => fn(["","+","=",""], a, b, a+b)** 
   - 获取原始模板字面量内容，而不是转换后的字符：**String.raw**标签函数，**字符串数组的.raw属性**
4. **Symbol类型**：
   -  **Symbol()** ：也可以传入一个字符串参数作为对符号的描述（但是这个字符串参数与符号定义或标识无关）
   -  **Symbol.for(key)** ：检查全局符号注册表中是否存在对应key的符号，没有则新建，有就返回该符号实例
   -  **Symbol.keyFor()** ：查询全局注册表，返回该全局符号对应的字符串键key
   - 对象字面量只能在计算属性语法中使用符号作为属性：let o = {  [Symbol('abc')] :  'abc val'}
   -  **Object.getOwnPropertyNames()** 返回对象实例的常规属性数组， **Object.getOwnPropertySymbols()** 返回对象实例的符号属性数组，这两个方法返回值彼此互斥
   -  **Object.getOwnPropertyDescriptors()**  ：返回同时包含常规和符号属性描述符的对象
   -  **Reflect.ownKeys()**  ：返回两种类型的键
   - 常用内置符号：
     - **Symbol.asyncIterator**：表示一个方法，返回对象默认的AsyncIterator(由**for-await-of**使用)
     - **Symbol.iterator**：表示一个方法，返回对象默认的迭代器(由**for-of**语句使用)
     - **Symbol.hasInstance**：表示一个方法，该方法决定一个构造器对象是否认可一个对象是它的实例(由**instanceof**操作符使用)
     - **Symbol.isConcatSpreadable**：表示一个布尔值，true意味着对象应该用**Array.prototype.concat()**打平其数组元素
     - **Symbol.species**：表示一个函数值，该函数作为创建派生对象的构造函数(在内置类型中最常用，用于对内置类型实例方法的返回值暴露实例化派生对象的方法)
     - **Symbol.toPrimitive**：一个方法，该方法将对象转换为相应的原始数值(由**toPrimitive**抽象操作使用)
     - **Symbol.toStringTag**：一个字符串，该字符串用于创建对象的默认字符串描述(由内置方法**Object.prototype.toString()**使用)
     - **Symbol.unscopables**：一个对象，该对象所有的以及继承属性，都会从关联对象的with环境中排除(设置这个符号并让其映射对应属性的键值为true，就能阻止该属性出现在with环境绑定中)
     - **Symbol.match**：一个正则表达式方法，匹配字符串(由**String.prototype.match()**方法使用)
     - **Symbol.replace**：一个正则表达式方法，替换字符串中匹配子串(由**String.prototype.replace()**方法使用)
     - **Symbol.search**：一个正则表达式方法，返回字符串中匹配正则表达式的索引(由**String.prototype.search()**方法使用)
     - **Symbol.split**：一个正则表达式方法，在匹配正则表达式的索引位置拆分字符串(由**String.prototype.split()**方法使用)
5. **Object类型**都有的属性和方法：
   - **constructor**：用于创建当前对象的函数
   - **hasOwnproperty**：用于检查给定属性在当前实例中(而不是在实例原型中)，检查的属性名必须为字符串或符号symbol
   -  **isPrototypeOf(object)** ：检查传入的对象是否是传入对象的原型
   -  **propertyIsEnumerable(propertyName)** ：判断给定的属性是否可以使用for-in语句枚举
   -  **toLocaleString()** ：返回对象的字符串表示，该字符串反应对象所在的本地化执行环境
   -  **toString()** ：返回对象的字符串表示
   -  **valueOf()** ：返回对象对应的字符串、数值或布尔表示（一般与toString()返回值相同）

## 3.3 操作符

1. 一元操作符：**递增/递减操作符++/--**，**一元加/减+/-**
   - 可以作用于非数值，会执行与使用Number()转型函数一样的类型转换
2. 位操作符：
   - 当数值应用位操作符时，64位数值被转换为32位数值，然后执行位操作，最后在将32位结果转为64位
   - NaN和Infinity值应用操作时，会被当成0来处理
   - 可以作用于非数值，会执行与使用Number()转型函数一样的类型转换
   - 负值的**补码**存储：绝对值的二进制对应的反码结果加1
   - **按位非~**：最终效果时对数值取反并减1
   - **按位与&**，**按位或|**，**按位异或^**，**左移<<**
   - 右移：
     - **有符号右移>>**：会将数值所有32位都向右移，同时保留符号(正或负)
     - **无符号右移>>>**：会将数值所有32位都向右移。对于正数，结果同有符号右移一样；对于负数，无符号右移会给空位补0，而不管符号位是什么；
3. 布尔操作符：
   - **逻辑非!**：先将操作数转换为布尔值，然后再对其取反 (同时使用两个(!!)相当于调用转型函数Boolean())
   - **逻辑与&&**：一种短路操作符，如果第一个操作数决定了结果，那么永远不会对第二个操作数求值
   - **逻辑或||**：与逻辑与类似，具有短路特性

4. 乘性操作符：如果有不是数值的操作数，则会在后台被使用Number()转型函数转换成数值
   - **乘法操作符***
   - **除法操作符/**
   - **取模操作符%**

5. **指数操作符****：相当于**Math.pow()**函数
6. 加性操作符：
   - **加法操作符+**：
     - 如果是Infinity加-Infinity，则返回NaN
     - 如果有一个是字符串，另一个转换成字符串再进行拼接
     - 任意操作数是对象、数值或布尔值，则调用它们的toString()方法以获取字符串
     - 对于null和undefined，则调用String()函数，分别获取"undefined"和"null"

   - **减法操作符-**：
     - 如果是-Infinity减-Infinity，则返回NaN
     - 如果有任一操作数是字符串、布尔值、null或undefined，则先在后台使用Number()将其转换为数值，然后再执行数学运算
     - 如果任一操作数是对象，则调用其valueOf()方法获取表示它的数值，如果没有valueOf()方法，则调用其toString()方法，再将得到的字符串转换为数值

7. 关系操作符：**小于<**、**大于>**、**小于等于<=**、**大于等于>=**
   - 如果有任一操作数为数值，则将另一个转换为数值再进行比较
   - 如果任一操作数是对象，则调用其valueOf()方法获取结果，如果没有valueOf()方法，则调用其toString()方法，取得结果后执行比较
   - 如果有任一操作符为布尔值，则将其转换为数值再执行比较
   - 任何关系操作符在涉及比较NaN时都返回false

8. 相等操作符：
   - **等于和不等于==/!=**：
     - 在比较之前执行转换（强制类型转换）
     - 如果任一操作数是布尔值，则将其转换为数值再比较
     - 如果一个操作数是字符串，另一个是数值，则尝试将字符串转换为数值
     - 如果一个操作数是对象，另一个不是，则调用对象的valueOf()方法取得原始值，再进行比较；如果都是对象，则比较是不是同一个对象
     - null和undefined相等；null和undefined不能转换为其他类型再比较
     - 如果有任一操作数是NaN，则相等操作数返回false，不相等操作符返回true（即使两个操作数都是NaN）

   - **全等和全不等===/!==**：
     - 在比较之前不执行转换
     - null === undefined 值为false，因为它们不是相同的数据类型

9. 条件操作符：**variable = boolean_expression ? true_value : false_value;**
10. 赋值操作符(仅仅是简写语法，不会提升性能)：**乘后赋值*=**、**除后赋值/=**、**取模后赋值%=**、**加后赋值+=**、**减后赋值-=**、**左移后赋值<<=**、**右移后赋值>>=**、**无符号右移后赋值>>>=**、*指数赋值操作符(**=)*

## 3.4 语句

1. **if语句**：条件condition会自动调用Boolean()函数将表达式转换为布尔值

   ```js
   if (condition1) statement1 else if (condition2) statement2 else statement3
   ```

2. **do-while语句**：后测试循环语句，即循环体中代码执行后才会对退出条件进行求值(循环体内代码至少执行一次)

   ```js
   do{
     statement
   }while(expression)
   ```

3. **while语句**：先测试循环语句，即先检测退出条件，再执行循环体内代码(循环体内代码有可能不会执行)

   ```js
   while(expression){
     statement
   }
   ```

4. **for语句**：

   ```js
   for(initialization; expression; post-loop-expression) statement
   ```

5. **for-in语句**：迭代语句，用于**枚举对象中的非符号键符号**

   - 对象的属性是无序的，因此for-in语句不能保证返回对象属性的顺序
   - 如果迭代变量是null或undefined，则不执行循环体

   ```js
   for(property in expression) statement
   ```

6. **for-of语句**：迭代语句，用于**遍历可迭代对象的元素**(for-await-of)

   ```js
   for(property of expression) statement
   ```

7. **标签语句**：用于给语句加标签

   ```js
   label : statement
   ```

8. **breake和continue语句**：

   - break语句用于立即退出循环，**强制执行循环后的下一条语句**
   - continue语句也用于立即退出循环，但会**再次从循环顶部开始执行**

9. **with语句**：用于将代码作用域设置为特定的对象

   ```js
   with (expression) statement
   ```

10. **switch语句**：switch语句在比较每个条件的值是会使用全等操作符，因此**不会强制转换数据类型**

    ```js
    switch(expression){
        case value1:
            statement
            break;
        case value2:
            statement
            break;
        case value3:
            satement
            break;
        default:
            statement
    }
    ```



# 4. 变量、作用域与内存

## 4.1 原始值与引用值

| 原始值(primitive vlaue)                                      | 引用值(referenece value)                                 |
| ------------------------------------------------------------ | -------------------------------------------------------- |
| 最简单的数据                                                 | 多个值构成的对象                                         |
| 大小固定，保存在**栈内存**                                   | 对象，保存在**堆内存**                                   |
| 变量是**按值访问**                                           | 变量是**按引用访问**，实际上只包含指向相应对象的**指针** |
| 变量之间复制会创建该值的第二个副本                           | 变量之间复制只会复制指针                                 |
| 原始类型初始化如果使用new关键字，则会创建一个Object类型的实例 | 只有引用值可以动态添加属性                               |
|                                                              | **函数参数都是按值传递**                                 |

## 4.2 执行上下文与作用域

- **执行上下文**：**全局上下文**、**函数上下文**、**块级上下文**（eval()调用内部也存在上下文）
- 代码执行流每进入一个新上下文，都会创建一个**作用域链**，用于搜索变量和函数，以及访问它们时的顺序
- **作用域链增强**：临时添加一个上下文
  - try/catch语句的catch块
  - with语句
- 变量的执行上下文用于确定什么时候释放内存

## 4.3 垃圾回收

- JavaScript是使用垃圾回收的语言，通过自动内存管理实现内存分配和闲置资源回收
- 垃圾回收策略：
  - **标记清理**：先给当前不使用的值加上标记，再回来回收它们的内存
  - **引用计数**：记录值被引用多少次
    - JavaScript引擎不再使用，但是某些旧版本IE仍然会受影响，因为js会访问非原生js对象(IE某些旧版本BOM和DOM中的对象是c++实现的组件对象模型对象，使用引用计数实现垃圾回收)
    - 在代码中存在**循环引用**时会出现问题
- 离开作用域的值会被自动标记为可回收，然后在垃圾回收期间被删除
- 内存泄露：
  - 没有使用任何关键字声明变量
  - 定时器（回调通过闭包引用了外部变量）
  - 闭包
- 解除变量的引用不仅可以消除循环引用，而且对垃圾回收也有帮助。全局对象、全局对象的属性和循环引用都应该在不需要时解除引用(设置为null)



# 5. 基本引用类型

- 引用值(或对象)是某个特定引用类型的实例
- 引用值与传统面向对象编程语言中的类相似，但实现不同
- 函数实际上是Fuction类型的实例，也就是说函数也是对象，因此也具有能够增强自身行为的方法

## 5.1 Date

- 日期保存为1970年1月1日午夜(零时)至今所经过的毫秒数
- 日期转为毫秒数(时间戳)方法：(**会被Date构造函数隐式调用**)
   -  **Date.parse()** ：
     - 接收字符串参数
     - 转换为本地时间
     - 解析结果可能因浏览器实现而异
   -  **Date.UTC()** ：
     - 接收多个数字参数
     - 转换为GMT时间，被构造函数隐式调用时转换为本地时间
   -  **Date.now()** ：返回表示方法执行日期和时间的毫秒数
- 继承的方法：
   -  **toLocaleString()** ：返回与浏览器运行的本地环境一致的日期和时间
   -  **toString()** ：返回带时区信息的日期和时间
   -  **valueOf()** ：不返回字符串，返回日期的毫秒表示
- 日期格式化方法：(输出与toLocaleString、toString一样，会因浏览器而异，不能用于界面显示)
   - **toDateString**、**toTimeString**、**toLocaleDateString**、**toLocaleTimeString**、**toUTCString**
- 日期/时间组件方法：
   - 日期的毫秒表示： **getTime()** 、 **setTime(milliseconds)** 
   - 年： **getFullYear()** 、 **getUTCFullYear()** 、 **setFullYear(year)** 、 **setUTCFullYear(year)** 
   - 月： **getMonth()** 、 **getUTCMonth()** 、 **setMonth(month)** 、 **setUTCMonth(month)** 
   - 日： **getDate()** 、 **getUTCDate()** 、 **setDate(date)** 、 **setUTCDate(date)** 
   - 周几： **getDay()** 、 **getUTCDay()** 
   - 时： **getHours()** 、 **getUTCHours()** 、 **setHours(hours)** 、 **setUTCHours(hours)** 
   - 分： **getMinutes()** 、 **getUTCMinutes()** 、 **setMinutes(minutes)** 、 **setUTCMinutes(minutes)** 
   - 秒： **getSeconds()** 、 **getUTCSeconds()** 、 **setSeconds(seconds)** 、 **setUTCSeconds(seconds)** 
   - 毫秒： **getMilliseconds()** 、 **getUTCMilliseconds()** 、 **setMilliseconds(milliseconds)** 、 **setUTCMilliseconds(milliseconds)** 
   - 以分钟计的UTC与本地时区的偏移量： **getTimezoneOffset()** 

## 5.2 RegExp

- 创建：字面量形式、构造函数(需要二次转义)

   ```js
   // let expression = /pattern/falgs; 字面量形式创建
   // flags: g(全局模式)|i(不区分大小写)|m(多行模式)|y(粘附模式)|u(Unicode模式)|s(dotAll模式)
   let pattern1 = /\[bc\]at/i;
   let pattern2 = new RegExp("\\[bc\\]at", "i");  //构造函数创建
   ```

- 实例属性：

   -  **global/ignoreCase/unicode/sticky/multiline/dotAll** ：布尔值，表示是否设置了(g/i/u/y/m/s)标记
   - **lastIndex**：整数，表示在源字符串中下一次搜索的开始位置，始终从0开始
   - **source**：正则表达式的字面量字符串，没有开头和结尾的斜杠
   - **flags**：标记字符串

- 实例方法：

   -  **exec()** ：返回数组，第一个元素是匹配整个模式的字符串，其他是与表达式中捕获组匹配的字符串(如果没有捕获组，则只有第一个元素)
     - 返回的数组包含额外属性：**index**(字符串中匹配模式的起始位置)、**input**(要查找的字符串)
   -  **test()** ：如果输入的文本与模式匹配，返回true，否则false
   -  **toLocaleString()** 、 **toString()** ：返回正则表达式的字面量表示
   -  **valueOf()** ：返回正则表达式本身

- 构造函数属性：

   -  **input($_)** ：最后搜索的字符串
   -  **lastMatch($&)** ：最后匹配的文本
   -  **lastParen($+)** ：最后匹配的捕获组
   -  **leftContext($`)** ：input字符串中出现在lastMatch前的文本
   -  **rightContext($')** ：input字符串中出现在lastMatch后的文本
   -  **RegExp.$1~RegExp.$9** ：第1~9个捕获组的匹配项

## 5.3 原始值包装类型

- 每当用到某个原始值的方法或属性时，后台都会创建一个相应原始包装类型的对象，从而暴露出操作原始值的各种方法
- **引用类型与原始包装类型的区别**：在于对象的生命周期
  - 在通过new实例化引用类型后，得到的实例会在离开作用域时被销毁
  - 自动创建的原始值包装对象只存在于访问它的那行代码执行期间
- 不推荐显式创建原始值包装类型，因为容易让开发者疑惑，分不清到底时原始值还是引用值，但它们对于操作原始值的功能是很重要的

### 5.3.1 Boolean

- 继承的方法：
  -  **valueOf()** ：返回一个原始值true或false
  -  **toString()** ：返回字符串“true”或“false”

### 5.3.2 Number

- 继承的方法：

    -  **valueOf()** ：返回Number对象表示的原始数值
    -  **toString()** ：返回数值字符串，**可选接收一个表示基数的参数，返回相应基数形式的数值字符串**
    -  **toLocaleString()** ：返回数值字符串

- 数值格式化为字符串：（会自动向上或向下舍入）

    -  **toFixed()** ：返回包含指定小数点位数的数值字符串
    -  **toExponential()** ：返回包含指定小数点位数的科学计数法表示的数值字符串
    -  **toPrecision()** ：根据情况返回最合理的输出结果，接收一个参数，表示结果中数字的总位数(不包含指数)
      - 本质上，根据数值和精度来决定调用toFixed还是toExponential
- 构造函数静态方法：
    -  **Number.isInteger()** ：用于判断一个数值是否保存为整数
      -  **Number.isSafeInteger()** ：鉴别整数是否在Number.MIN_SAFE_INTEGER(-2e+53+1)~Number.MAX_SAFE_INTEGER(2e+53-1)安全整数范围内

### 5.3.3 String

- 继承的方法： **valueOf()** 、 **toString()** 、**toLocaleString()** 都返回对象的原始字符串值
- 字符：

    - 每个String对象都有一个**length属性**，表示字符串中字符的数量
    -  **charAt()** ：返回给定索引位置的字符
    -  **charCodeAt()** ：查看指定码元字符编码
    -  **charPointAt()** ：同charCodeAt，但是能解析既包含单码元字符也包含代理对字符的字符串(如果传入的码元索引并非代理对的开头，会返回错误的码点)
    -  **fromCharCode()** ：根据给定的UTF-16码元创建字符串中的字符
    -  **fromCodePoint()** ：同fromCharCode，但是既包含单码元字符也包含代理对字符
    -  **normalize("NFD|NFC|NFKD|NFKC")** ：针对看起来一样但编码方式不同的字符，进行规范化一致的格式
- 字符串操作方法：

    - **concat()** ：拼接字符串，更常用加号操作符(+)

        -  **slice()** ：提取子字符串
          - 第一个参数表示字符串开始位置，第二个参数表示子字符串结束位置
          - 参数是负数时：所有负值参数当成字符串长度+负值
        -  **substr()** ：提取子字符串
          - 第一个参数表示字符串开始位置，第二个参数表示**返回的子字符串数量**
          - 参数是负数时：第一个参数当成字符串长度+负值，**第二个参数转换为0**
    -  **substring()** ：提取子字符串
      - 第一个参数表示字符串开始位置，第二个参数表示子字符串结束位置
      - 参数是负数时：**将所有负值转换为0**
- 字符串位置方法：（接收可选的第二参数，表示开始搜索的位置）

    -  **indexOf()** ：从字符串开头查找字符串
    -  **lastIndexOf()** ：从字符串末尾开始查找字符串
- 字符串包含方法：

    -  **startsWith()** ：接收可选第二参数，表示开始搜索的位置
    -  **endsWith()** ：接收可选第二参数，表示当作字符串末尾的位置
    -  **includes()** ：接收可选第二参数，表示开始搜索的位置
- **trim()、trimLeft()、trimRight()** ：创建字符串的一个副本，删除前后所有空格，再返回结果
- **repeat()** ：接收一个整数参数，表示要将字符串复制多少次，然后返回拼接所有副本后的结果
- **padStart()、padEnd()** ：复制字符串，如果小于指定长度，则在相应一边填充字符

    - 第一个参数为长度，第二参数为可选填充字符串，默认空格
    - 如果长度小于或等于字符串长度，则返回原始字符串
- **可迭代和解构**
- 字符串大小写转换方法： **toLowerCase()、toUpperCase()** 、(基于特定地区实现) **toLocaleLowerCase()、toLocaleUpperCase()**  
- 字符串模式匹配方法：

     -  **match()** ：本质跟RegExp对象的exec()方法相同，返回同样的数组
     -  **search()** ：返回模式第一个匹配的位置索引，如果没找到返回-1
     -  **replace()** ：第一个参数可以是RegExp对象或字符串(不会转换为正则表达式)，第二个参数可以是字符串或函数(函数参数：与整个模式匹配的字符串、匹配项在字符串中的开始位置、整个字符串)
     -  **split()** ：根据传入的分隔符将字符串拆分成数组（第二个参数是数组大小，确保返回的数组不会超过指定大小）
- **localeCompare()** ：按照字母表顺序，比较两个字符串

## 5.4 单例内置对象

- 任何由ECMAScript实现提供、与宿主环境无关，并在ECMAScript程序开始执行时就存在的对象。开发者不用显式地实例化对象，因为它们已经实例化好了
- 当代码开始执行时，全局上下文会存在两个内置对象：**Global**、**Math**
- 内置对象还有：**Object**、**Array**、**String**

### 5.4.1 Global

- 一种兜底对象，他所针对的是不属于任何对象的属性和方法。不存在全局变量或函数，**在全局作用域中定义的变量和函数都会变成Global对象的属性**
- URI编码方法：用于编码统一资源标识符(URI)，以便传给浏览器
  -  **encodeURI()、decodeURI()** ：不会编码属于URL组件的特殊字符（编码整个URI）
  -  **encodeURIComponent()、decodeURIComponent()** ：会编码它发现的所有非标准字符（编码会追加到已有URI后面的字符串）
-   **eval()** ：
  - 一个完整的ECMAScript解释器，接收一个参数，即一个要执行的ECMAScript字符串
  - 通过eval执行的代码属于该调用所在上下文，被执行的代码与该上下文有相同的作用域链
  - 通过eval定义的任何变量和函数都不会被提升
  - 严格模式下，eval内部创建的变量和函数无法被外部访问
- **window对象**：浏览器将window对象实现为Global对象的代理。因此，所有全局作用域中声明的变量和函数都变成了window的属性

### 5.4.2 Math

- 提供一些辅助计算的方法和属性
- **Math.min()、Math.max()** ：用于确定一组数值中的最小值和最大值
- 舍入方法：

    -   **Math.ceil()** ：始终向上舍入
    -   **Math.floor()** ：始终向下舍入
    -   **Math.round()** ：执行四舍五入
    -   **Math.fround()** ：返回数值最接近的单精度(32位)浮点值表示
- **random()** ：返回一个0~1范围内的随机数，其中包含0不包含1

    - 公式：`Math.floor( Math.random() * total_number_of_choices + first_possible_value)`
    - 如果是为了加密而需要生成随机数，建议使用 **window.crypto.getRandomValues()** 
- 其他方法：

    -   **Math.abx(x)** ：返回x的绝对值

    -   **Math.pow(x, power)** ：返回x的power次幂




# 6. 集合引用类型

## 6.1 Object

- Object是一个基础类型，所有引用类型都从它继承了基本行为
- 属性名可以是**字符串**、**数值**、**Symbol符号**（数值属性会自动转换为字符串）

## 6.2 Array

- 创建的静态方法：
   -  **Array.from(arr， fn， this)** ：用于将类数组结构转换为数组实例
     - 类数组结构：任何可迭代的结构，或者有一个length属性和可索引元素的结构
     - 可选第二参数：映射函数参数，可以直接增强新数组的值，不用通过Array.from().map()那样创建一个中间数组
     - 可选第三参数：指定映射函数的this值（但在箭头函数中不适用）
   -  **Array.of()** ：将一组参数转换为数组实例
- 数组索引：
   - **length属性**：通过修改length属性，可以从数组末尾删除或添加元素
- 检测数组：
   -  **Array.isArray()** ：确定一个值是否为数组，而不用管它是在哪个全局上下文中创建
     - 在只有一个网页的情况下，使用**instanceof**操作符就足够
     - 但是网页里面有多个框架，则可能涉及两个不同的全局执行上下文，就会有两个不同版本的Array构造函数
- 迭代器方法：
   -  **keys()** ：返回数组索引的迭代器
   -  **values()** ：返回数组元素的迭代器
   -  **entries()** ：返回索引/值对的迭代器
- 复制和填充方法：(**静默忽略超出数组边界、零长度及方向相反的索引范围**) （会改变数组本身）
   -  **copyWithin(index, start, end)** ：按照指定范围浅复制数组中部分内容，然后插入到指定索引开始的位置(js引擎在插值前会完整复制范围内的值，因此复制期间不存在重写的风险)
   -  **fill(num, start, end)** ：填充数组
- 转换方法：(**如果数组中某一项是null或undefine，则会转换为空字符串**)
   - 继承的方法：
     -  **valueOf()** ：返回数组本身
     -  **toString()** ：返回每个值调用toString方法，并用逗号分隔的字符串
     -  **toLocaleString()** ：返回每个值调用toLocaleString方法，并用逗号分隔的字符串
   -  **join()** ：返回每个值调用toString方法，并用设置的参数分隔的字符串
- 栈/队列方法：
   -  **push()** ：添加参数到数组末尾，返回数组最新长度
   -  **pop()** ：删除数组最后一项，返回被删除的值
   -  **shift()** ：删除数组的第一项，返回被删除的值
   -  **unshift()** ：添加参数到数组开头，返回数组最新长度
   - 栈(后进先出LIFO，Last-In-First-Out)：push、pop方法
   - 队列(先进先出FIFO)：push、shift方法
   - 反向队列：unshift、pop方法
- 排序方法：(都返回调用它们的数组的引用)
   -  **reverse()** ：将数组元素反向排列
   -  **sort()** ：每一项调用String转型函数，然后比较字符串来决定顺序
     - 可接收一个比较函数，如：((a,b) => a\<b?1:a\>b?-1:0)/((a,b) => a-b)
- 操作方法：
   -  **concat()** ：在现有数组全部元素基础上创建一个新数组
   -  **slice(start, end)** ：创建一个包含原有数组中一个或多个元素的新数组(结束位置小于开始位置，返回空数组)
   -  **splice(开始位置, 要删除的元素数量, 要插入的元素)** ：可以进行删除、插入、替换操作，返回被删除的元素组成的数组/空数组
- 搜索和位置方法：
  - 严格相等搜索：(参数：要查找的元素、(可选)起始搜索位置； 使用全等(===)比较)
    -  **indexOf()** ：从头开始搜索，返回第一个匹配元素位置
    -  **lastIndexOf()** ：从尾开始搜索，返回第一个匹配元素位置
    -  **includes()** ：从头开始搜索，返回布尔值
  - 断言函数搜索：（断言函数参数：元素、索引、数组本身;返回布尔值）
    -  **find(fn, this)** ：返回第一个匹配元素
    -  **findIndex(fn, this)** ：返回第一个匹配元素对应的索引
- 迭代方法：(参数：fn, this； 其中fn参数： 元素，索引，数组本身)不改变调用它们的数组
  -  **filter()** ：函数返回true的项会组成数组之后返回
  -  **forEach()** ：相当于使用for循环遍历数组，没有返回值
  -  **map()** ：返回由每次函数调用结果构成的数组
  -  **every()** ：如果每一项函数都返回true，则返回true
  -  **some()** ：如果有一项函数返回true，则返回true
- 归并方法：(参数：fn, (可选)归并起点值； 其中fn参数：上一个归并值，当前项，当前项的索引值，数组本身；) (函数返回的任何值会作为下次调用同一个函数的参数; 如果没有传入归并起点值，则第一次迭代将从数组第二项开始)
  -  **reduce()** ：从头开始遍历
  -  **reduceRight()** ：从尾开始遍历

## 6.3 定型数组(typed array)

- 目的是提升与WebGL等原生库交换二进制数据的效率
- js并没有“TypeArray”类型，它所指的其实是一种特殊的包含数值类型的数组
- 包含一套不同的引用类型，用于管理数值在内存中的类型

### 6.3.1 ArrayBuffer

- ArrayBuffer是所有定型数组及视图引用的**基本单位**
- ArrayBuffer一经创建就不能再调整大小，可以通过slice()复制其全部或部分到一个新实例中

### 6.3.2 DataView

- DataView是允许读写ArrayBuffer的**视图**，专为文件I/O和网络I/O设计
- DataView对缓冲内容没有任何预设，也**不能迭代**
- 对已有的ArrayBuffer读取或写入时才能创建DataView实例

```js
const buf = new ArrayBuffer(16);
const halfDataView = new DataView(buf, 0, 8);//参数：(buf, 字节偏移量，字节长度)
```

- 8种ElementType类型：**Int8**、**Uint8**、**Int16**、**Uint16**、**Int32**、**Unit32**、**Float32**、**Float64**
- DataView对每个类型有对应的 **get(offset, 字节序)** 和 **set(offset, value)** 方法

### 6.3.3 定型数组

- 创建：**读取已有的缓冲**、**使用自有缓冲**、**填充可迭代结构**、**填充基于任意类型的定型数组**、 **\<ElementType\>.from()** 、 **\<ElementType\>.of()** 
- 构造函数和实例都有一个属性：**BYTES_PER_ELEMENT**，返回该类型数组中每个元素的大小
- 同样使用数组缓冲来存储数据，而数组缓冲无法调整大小
- 向外或向内复制数据：
  -  **set(array, index)** ：从提供的数组或定型数组中把值复制到当前定型数组中索引的位置
  -  **subarray(start, end)** ：基于从原始定型数组中复制的值返回一个新定型数组

## 6.4 Map(映射)

- Map可以使用任何js数据类型作为键，内部使用SameValueZero比较操作，基本相当于使用严格对象相等的标准来检查键的匹配性
- 基本API：
  -  **set(key, value)** ：添加键/值对，返回映射实例(所以可以把多个set操作连缀起来)
  -  **get(key)** ：查询
  -  **has(key)** ：查询
  -  **size属性** ：获取映射中键/值对的数量
  -  **delete(key)** ：删除一个键/值对
  -  **clear()** ：清除所有键/值对
- 顺序与迭代：Map实例会维护插入顺序，因此可以根据顺序进行迭代
  -  **entries()** ：返回插入顺序[key, value]迭代器，Symbol.iterator属性引用它
  -  **keys()** ：返回以插入顺序生成键的迭代器
  -  **values()** ：返回以插入顺序生成值的迭代器
  -  **forEach(callback, opt_thisArg)** 
- 与Object比较：
  - 内存占用：不同浏览器情况不同，但给定固定大小的内存，Map大约可以比Object多存储50%的键/值对
  - 插入性能：向Object和Map中插入新键/值对的消耗大致相当，不过插入Map在所有浏览器中一般会稍微快一点
  - 查找速度：从大型Object和Map中查找键/值对的性能差异极小，但如果只包含少量键/值对，则Object有时候速度更快
  - 删除性能：对大多数引擎来说，Map的delete操作都比插入和查找更快；如果涉及大量删除操作，Map是更好的选择

## 6.5 WeakMap(弱映射)

- 键只能是Object或者继承自Object的类型
- 基本API：**set**、**get**、**has**、**delete**（**不可迭代**）
- 弱键：**这些键不属于正式的引用，不会阻止垃圾回收**
- 使用场景：私有变量、保存DOM节点元数据

## 6.6 Set(集合)

- 同Map，可以使用任何数据作为键，内部使用SameValueZero比较操作
- 基本API：
  -  **add(value)** ：返回集合实例（可以把多个set操作连缀起来）
  -  **has(value)** ：查询
  -  **size属性** ：获取集合数量
  -  **delete(value)** ：返回一个布尔值，表示集合中是否存在要删除的值
  -  **clear()** ：销毁集合中所有值
- 顺序与迭代：会维护插入顺序，因此可以根据顺序进行迭代
  -  **values()** ：返回插入顺序的值的迭代器，Symbol.iterator属性引用它
  -  **keys()** ：values的别名方法
  -  **entries()** ：返回插入顺序产生包含两个元素的数组的迭代器，两个元素是集合中每个值的重复出现
  -  **forEach(callback, opt_thisArg)** 

## 6.7 WeakSet(弱集合)

- 键只能是Object或者继承自Object的类型
- 基本API：**add**、**has**、**delete**（**不可迭代**）
- 弱键：**这些键不属于正式的引用，不会阻止垃圾回收**
- 使用场景：给对象打标签



# 7. 迭代器与生成器(ES6)

## 7.1 迭代器

-  **可迭代对象(iterable)** ：实现了正式的Iterable接口，可以通过迭代器Iterator消费

-  **迭代器(iterator)** ：一个可以由任意对象实现的接口，支持连续获取对象产出的每一个值。按需创建的一次性对象，每个迭代器都会关联一个可迭代对象。

- **实现Iterable接口(可迭代协议)要求**：支持迭代的自我识别能力、创建实现Iterator接口的对象的能力

- **Symbol.iterator属性**：任何实现Iterable接口的对象都有。引用默认的迭代器，默认迭代器就像一个迭代器工厂，也就是一个函数，调用之后会产生一个实现Iterator接口的对象

- **实现了Iterable接口的内置类型**：字符串、数组、映射、集合、arguments对象、NodeList等DOM集合类型

- **接收可迭代对象的原生语言特性**：

  - for-of循环
  - 数组解构、扩展操作符
  - Array.from()
  - 创建集合、映射
  - Promise.all()、Promise.race()接收由期约组成的可迭代对象
  - yield*操作符，在生成器中使用

- 迭代器API方法：

  -  **next()** ：每次成功调用next，都会返回一个**IteratorResult**对象，包含两个属性：**done**、**value**

    - 每个迭代器都表示对可迭代对象的一次有序遍历。不同迭代器的实例相互之间没有联系，只会独立地遍历可迭代对象

    - 迭代器并不与可迭代对象某个时刻的快照绑定，而仅仅是使用游标来记录遍历可迭代对象的历程。如果可迭代对象在迭代期间被修改了，那么迭代器也会反映相应的变化

    - 迭代器维护着一个指向可迭代对象的引用，因此迭代器会阻止垃圾回收程序回收可迭代对象

    - 创建的迭代器也实现了Iterable接口，Symbol.iterator属性引用的工厂函数会返回相同的迭代器

      ```js
      let arr = ['foo', 'bar', 'baz'];
      let ite1 = arr[Symbol.iterator]();
      let ite2 = ite1[Symbol.iterator]();
      console.log(ite1 === ite2);			//true
      ```

  -  **return()** ：(可选方法)用于指定在迭代器提前关闭时执行的逻辑，必须返回一个有效的**IteratorResult**对象，可以只返回 **{done: true}** 

    - 可能情况：**for-of循环通过break、continue、return或throw提前退出**，**解构操作并未消费所有值**
    - 如果迭代器没有关闭，则还可以继续从上次离开的地方继续迭代。比如，**数组的迭代器就是不能关闭的**
    - return方法是可选的，所以并非所有迭代器都是可关闭的。要知道某个迭代器是否可关闭，可以测试这个迭代器实例的return属性是不是函数对象
    - 给一个不可关闭的迭代器增加return方法不能让它变成可关闭的，但是return()方法还是会被调用的

## 7.2 生成器

- 拥有在一个函数块内暂停和恢复代码执行的能力。

- 使用场景：**自定义迭代器**、**实现协程**

- 生成器的形式是一个函数，函数名称前加星号(*)表示它是一个生成器。只要是可以定义函数的地方就能定义生成器（**箭头函数不能用来定义生成器函数**）

- 调用生成器函数会产生一个生成器对象。生成器对象也实现了Iterator接口，因此具有next()方法，可用在任何消费可迭代对象的地方。

- 生成器对象一开始处于暂停执行(suspended)状态。**生成器函数只会在初次调用next()方法后开始执行**

- 生成器对象实现了Iterable接口，它们默认的迭代器是自引用的

  ```js
  function *generatorFn() {}
  const g = generatorFn();
  console.log(g === g[Symbol.iterator]());		//true
  ```

- **yield**：

  - **yield关键字只能在生成器函数内部使用**
  - yield关键字可以让生成器停止和开始执行。生成器函数在遇到yield之前会正常执行，遇见之后执行会停止，函数作用域的状态会被保留。停止执行的生成器函数只能通过在生成器对象上调用next()方法来恢复执行
  - 通过**yield**关键词退出的生成器函数会处在**done:false**状态；通过**return**关键字退出的生成器函数会处于**done:true**状态
  - 生成器函数内部的执行流程会针对每个生成器对象区分作用域，因此在一个生成器对象上调用next()不会影响其他生成器
  - **上一次让生成器暂停的yield关键字会接收到传给next()方法的第一个值**
  - 使用星号增强yield行为，让它能够迭代一个可迭代对象，如：yield* [1, 2, 3];  **yield*的值是关联迭代器返回done:true时的value属性**

- **使用yield*实现递归算法**：

  ```js
  function *nTimes(n){
      if(n > 0){
          yield* nTimes(n-1);
          yield n-1;
      }
  }
  for (const x of nTimes(3))}{
      console.log(x);
  }
  ```

- **生成器作为默认迭代器**：生成器函数和默认迭代器被调用之后都产生迭代器，所以生成器格外适合作为默认迭代器。

  ```js
  class Foo{
      constructor(){
          this.value = [1, 2, 3];
      }
      *[Symbol.iterator](){
          yield* this.value;
      }
  }
  ```

- 生成器对象方法：

  -  **next()** ：同迭代器
  -  **return()** ：**会强制生成器进入关闭状态。提供给return()方法的值，就是终止迭代器对象的值**
    - 与迭代器不同，**所有生成器对象都有return()方法**，只要通过它进入关闭状态，就无法恢复了。后续调用next()会显示done:true状态，而提供的任何返回值都不会被存储或传播；
    - for-of循环等内置语言结构会忽略状态为done:true的IteratorObject内部返回的值
  -  **throw()** ：会在暂停的时候将一个提供的错误注入到生成器对象中。
    - 如果错误未被处理，生成器就会关闭；如果生成器函数内部处理了这个错误，那么生成器就不会被关闭，而且还可以恢复执行。**错误处理会跳过对应的yield**。
    - 如果生成器对象还没有开始执行，那么调用throw()抛出的错误不会在函数内部被捕获，因为这相当于在函数块外部抛出了错误



# 8. 对象、类与面向对象编程

## 8.1 理解对象

- 属性类型：

  1. **数据属性** ：包含一个保存数据值的位置。
     - 特性：
       1. **[[Configurable]]**：表示属性是否可以通过delete删除并重新定义，是否可以修改它的特性，以及是否可以把它改为访问器属性（默认true）
       2. **[[Enumerable]]**：表示属性是否可以通过for-in循环返回（默认true）
       3. **[[Writable]]**：表示属性的值是否可以修改（默认true）
       4. **[[Value]]**：包含属性实际的值（默认undefined）
     - 一个属性被定义为不可配置([[Configurable]]为false)之后，就不能再变回可配置的了
  2. **访问器属性** ：不包含数据值。包含一个获取函数和一个设置函数（两个函数非必需）
     - 特性：
       1. **[[Configurable]]**：表示属性是否可以通过delete删除并重新定义，是否可以修改它的特性，以及是否可以把它改为数据属性（默认true）
       2. **[[Enumerable]]**：同上
       3. **[[Get]]**：获取函数，再读取属性时调用（默认undefined）
       4. **[[Set]]**：设置函数，再写入属性时调用（默认undefined）
     - 访问器属性不能直接定义，必须使用Object.defineProperty()

- 属性方法：

  1.  **Object.defineProperty(对象， 属性名称， 描述符对象)** ：定义单个属性特性
  2.  **Object.defineProperties(对象， 描述符对象)** ：通过多个描述符一次性定义多个属性
  3.  **Object.getOwnPropertyDescriptor(对象， 属性名)** ：取得指定属性的属性描述符对象
  4.  **Object.getOwnPropertyDescriptors(对象)** ：在每个自有属性上调用Object.getOwnPropertyDescriptor()并在一个新对象中返回它们

- 对象方法：

  1.  **Object.assign()** ：合并对象，返回修改后的目标对象（浅复制）
     - 将每个源对象中可枚举和自有属性复制到目标对象。以字符串和符号为键的属性会被复制。
     - 对每个符合条件的属性，会使用源对象上的[[Get]]取得属性的值，然后使用目标对象上的[[Set]]设置属性的值。
     - 从源对象访问器属性取得的值，比如获取函数，会作为一个静态值赋给目标对象，因此，不能在两个对象间转移获取函数和设置函数
     - 如果赋值期间出错，则操作会中止并退出，同时抛出错误。它没有“回滚”概念，因此它是一个尽力而为，可能只会完成部分复制的方法。
  2.  **Object.is()** ：与===很像，但是考虑到了边界情形。

- ES6增强的对象语法：属性值简写、可计算属性、简写方法名

- **对象解构**：

  - 对象解构就是使用与对象匹配的结构来实现对象属性赋值

  - 解构在内部使用函数ToObject()（不能在运行时环境中直接访问）把源数据结构转换为对象。这意味着在对象解构的上下文中，原始值会被当成对象。这也意味着，null和undefined不能被解构，否则会抛出错误。

  - 解构并不要求变量必须在解构表达式中声明。不过，如果是给事先声明的变量赋值，则赋值表达式必须包含在一对括号中。

  - 在外层属性没有定义的情况下不能使用嵌套解构，无论源对象还是目标对象。

  - 涉及多个属性的解构赋值是一个输出无关的顺序化操作。如果一个解构表达式涉及多个赋值，开始的赋值成功后而后面的赋值出错，则整个解构赋值只会完成一部分。

  - 在函数参数列表中也可以进行解构赋值。对参数的解构赋值不会影响argument对象，但可以在函数签名中声明在函数体内使用的局部变量。

    

## 8.2 创建对象

### 8.2.1 工厂模式

- 用于抽象创建特定对象的过程
- **优点**：解决创建多个类似对象的问题
- **缺点**：没有解决对象标识问题（即新创建的对象是什么类型）

```js
function createPerson(name, age, job){
    let o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function(){
        console.log(this.name);
    }
    return o;
}
let person1 = createPerson("xiaohong", 29, "Software Engineer");
let person2 = createPerson("lihua", 27, "Doctor");
```

### 8.2.2 构造函数模式

- 自定义构造函数，以函数的形式为自己的对象类型定义属性和方法

- **优点**：可以确保实例被标识为特定类型(如： person instaceof Person)
- **缺点**：定义的方法会在每个实例上都创建一遍（相同逻辑的函数重复定义）

```js
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function(){
        console.log(this.name);
    }
}
let person1 = new Person("xiaohong", 29, "Software Engineer");
let person2 = new Person("lihua", 27, "Doctor");
console.log(person1 instanceof Object);		//true
console.log(person1 instanceof Person);		//true
console.log(person2 instanceof Object);		//true
console.log(person2 instanceof Person);		//true
```

- 和工厂模式函数区别：没有显式创建对象、属性和方法直接赋值给this、没有return
- 使用new操作符调用构造函数会执行操作：
  1. 在内存中创建一个新对象
  2. 这个新对象内部的[[Prototype]]特性被赋值为构造函数的prototype属性
  3. 构造函数内部的this被赋值为这个新对象（即this指向新对象）
  4. 执行构造函数内部的代码（给新对象添加属性）
  5. 如果构造函数返回非空对象，则返回该对象；否则，返回刚创建的新对象。
- 构造函数与普通函数的唯一区别：调用方式不同。除此之外，构造函数也是函数。任何函数只要使用new操作符调用就是构造函数，而不使用new操作符调用的函数就是普通函数。
- 在调用一个函数而没有明确设置this的情况下（即没有作为对象的方法调用或没有使用call/apply调用），this始终指向Global对象（在浏览器中就是window对象）

### 8.2.3 原型模式

- **优点**：原型对象上定义的属性和方法可以被对象实例共享
- **缺点**：弱化了向构造函数传递初始化参数的能力，会导致所有实例默认都取得相同的属性值

```js
function Person() {}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function(){
     console.log(this.name);
}
let person1 = new Person();
let person2 = new Person();
console.log(person1.sayName == person2.sayName);		//true
```

- **原型**：

  ![原型](./Screenshot_20250916_131738.jpg)

  - 每个函数都会创建一个prototype属性，这个属性是一个对象，包含应该由特定引用类型的实例共享的属性和方法。这个对象就是通过调用构造函数创建的对象的原型。
  - 只要创建一个函数，就会按照特定的规则为这个函数创建一个**prototype**属性(指向原型对象)
  - 所有原型对象自动获得一个名为**constructor**的属性，指向与之关联的构造函数
  - 每次调用构造函数创建一个新实例，这个实例内部[[Prototype]]指针就会被赋值为构造函数的原型对象。
  - 脚本中没有访问这个[[Prototype]]特性的标准方式，但Firefox、Safari和Chrome会在每个对象上暴露**\_\_proto\_\_**属性，通过这个属性可以访问对象的原型
  - **实例与构造函数原型之间有直接的联系，但实例与构造函数之间没有**
  - 正常原型链会终止于Object对象的原型对象，**Object原型的原型是null**
  - 同一构造函数创建的两个实例，共享同一个原型对象

- 原型相关方法：

  -  **instanceof**：用于检查实例的原型链中是否包含指定构造函数的原型
  -  **isPrototypeOf()** ：在传入参数的[[Prototype]]指向调用它的对象时返回true（如：Person.prototype.isPrototypeOf(person1) ）
  -  **Object.getPrototypeOf()** ：返回参数的内部特性[[Prototype]]的值（如：Object.getPrototypeOf(person1) == Person.prototype ）
  -  **Object.setPrototypeOf()** ：向实例的私有特性[[Prototype]]写入新制，重写对象的原型继承关系（可能会严重影响代码性能）
  -  **Object.create()** ：创建一个新对象，同时为其指定原型

- 判断属性方法：

  -  **Object.hasOwnProperty()** ：在属性存在于调用它的对象实例上时返回true（可以用来判断属性是在实例上还是在原型对象上）
  -  **in操作符** ：在可以通过对象访问指定属性时返回true，无论属性在实例还是在原型上。
    - 判断属性在原型上：!object.hasOwnProperty(name) && (name in object)

- 枚举属性方法：

  -  **for-in** ：可以通过对象访问且可以被枚举的属性都会返回，包括实例属性和原型属性
  -  **Object.keys()** ：返回对象上所有可枚举的实例属性
  -  **Object.getOwnPropertyNames()** ：返回对象上所有实例属性，无论是否可枚举
  -  **Object.getOwnPropertySymbols()** ：与getOwnPropertyNames类似，只针对符号
  - **枚举顺序**：
    - for-in、Object.keys()：枚举顺序不确定，取决于JavaScript引擎，可能因浏览器而异；
    - Object.getOwnPropertyNames()、Object.getOwnPropertySymbols()、Object.assign()：枚举顺序确定，先以升序枚举数值键，然后以插入顺序枚举字符串和符号键。

- 对象迭代：（浅复制，非字符串属性会被转换为字符串输出，符号属性会被忽略）

  -  **Object.values()** ：返回对象值的数组

  -  **Object.entries()** ：返回键/值对数组

    

## 8.3 继承

- 接口继承：只继承方法签名；
- 实现继承：继承实际的方法
- 实现继承是ECMAScript唯一支持的继承方式，主要通过原型链实现

### 8.3.1 原型链

- 子类型原型是父类型的实例。
- **缺点**：原型中包含的引用值会在所有实例间共享；子类型在实例化时不能给父类型的构造函数传参；

```js
function SuperType(){
    this.property = true;
}
SuperType.prototype.getSuperValue = function(){
    return this.property;
}
function SubType(){
    this.subproperty = false;
}
//继承SuperType
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function(){
    return this.subproperty;
}
let instance = new SubType();
```

![原型链](./Screenshot_20250916_161305.jpg)

### 8.3.2 盗用构造函数(对象伪装/经典继承)

- 在子类构造函数中调用父类构造函数，使用apply和call方法以新创建的对象为上下文执行构造函数
- **优点**：可以在子类构造函数中向父类构造函数传参
- **缺点**：在构造函数中定义的方法不能重用；子类不能访问父类原型上定义的方法

```js
function SuperType(){
    this.colors = ['red', 'blue', 'green'];
}
function SubType(){
    //继承SuperType
    SuperType.call(this);
}
let instance1 = new SubType();
let instance2 = new SubType();
```

### 8.3.3 组合继承(伪经典继承)

- 综合了原型链和盗用构造函数，使用原型链继承原型上的属性和方法，通过盗用构造函数继承实例属性
- **优点**：保留了instanceof和isPrototypeOf()方法识别合成对象的能力
- **缺点**：效率问题，父类构造函数始终会被调用两次：一次在创建子类原型时，一次在子类构造函数中调用；导致SuperType的实例属性变成SubType的原型属性

```
function SuperType(name){
	this.name = name;
    this.colors = ['red', 'blue', 'green'];
}
SuperType.prototype.sayName = function(){
    console.log(this.name);
}
function SubType(name, age){
    //继承属性
    SuperType.call(this, name);
    this.age = age;
}
//继承方法
SubType.prototype = new SuperType();
SubType.prototype.sayAge = function(){
    console.log(this.age);
}
```

### 8.3.4 原型式继承

- 即使不自定义类型也可以通过原型实现对象之间的信息共享（无需明确定义构造函数而实现继承，本质上是对给定对象执行浅复制）
- **优点**：适合不需要单独创建构造函数，但仍需要在对象间共享信息的场合。
- **缺点**：和原型链一样，引用值会在相关对象间共享
- 关键：

```js
function object(o){
    function F(){}
    F.prototype = o;
    return new F();
}
```

-  **Object.create(作为新对象原型的对象， 给新对象定义额外属性的对象(可选))** 方法将原型式继承的概念规范化了

```js
let person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};
let anotherPerson = object(person);
let yetAnotherPerson = Object.create(person);
```

### 8.3.5 寄生式继承

- 先基于一个对象创建一个新对象，然后再增强这个新对象，最后返回新对象
- **优点**：适合主要关注对象，而不在乎类型和构造函数的场景
- **缺点**：与构造函数模式类似，通过寄生式继承给对象添加函数会导致函数难以重用

```js
function createAnother(original){
    let clone = object(orginal);	//通过调用函数创建一个新对象
    clone.sayHi = function (){		//以某种方式增强这个对象
        console.log("hi");
    };
    return clone;
}

let person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};
let anotherPerson = createAnother(person);
```

### 8.3.6 寄生式组合继承

- 通过盗用构造函数继承属性，但使用混合式原型链继承方法。
- 使用寄生式继承来继承父类原型，然后将返回的新对象复制给子类型原型。
- **优点**：解决了组合继承中，父类构造函数始终会被调用两次的效率问题

```js
function inheritPrototype(subType, superType){
    let prototype = object(superType.prototype);	//创建对象
    prototype.constuctor = subType;					//增强对象
    subType.prototype = prototype;					//赋值对象
}
function SuperType(name){
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}
SuperType.prototype.sayName = function(){
    console.log(this.name);
}
function SubType(name, age){
    //继承属性
    SuperType.call(this, name);
    this.age = age;
}
//继承方法
inheritPrototype(SubType, SuperType);
SubType.prototype.sayAge = function(){
    console.log(this.age);
}
```



## 8.4 类

- 背后使用仍然是原型和构造函数的概念。
- 类表达式在被求值之前不能引用；虽然函数定义声明可以提升，但类定义不能；
- 函数受函数作用域限制，类受块作用域限制；
- 类是JavaScript的一等公民，因此可以像其他对象或函数引用一样把类作为参数传递；
- 类为特殊函数，typeof返回function
- 类也有prototype属性指向原型，同时原型也有constructor指向类
- 类也能使用instanceof
- 类的构成：
  - **类构造函数constructor**：
    - 方法名constructor会告诉解释器在使用new操作符创建类的新实例时，应该调用这个函数。
    - 构造函数定义不是必需的，不定义构造函数相当于将构造函数定义为空函数。
    - 与构造函数的区别：调用类构造函数必须使用new操作符。而普通构造函数如果不使用new调用，就会以全局的this作为内部对象。
    - 实例化后，类构造函数会成为普通的实例方法，但仍然要使用new才能调用
  - **实例成员**：在constructor中定义
  - **原型方法**：在类块中定义的方法
  - **访问器**：在类块中定义获取和设置访问器
  - **静态类方法(static关键字)**：通常用于执行不特定于实例的操作，也不要求存在类的实例
  - **非函数原型和类成员**：虽然类定义并不显式支持在原型或类上添加成员数据，但在类定义外部，可以手动添加
  - **迭代器与生成器方法**：类定义语法支持在原型和类本身上定义生成器方法（可以通过添加一个默认迭代器，把类实例变成可迭代对象）
- **继承(extends关键字)**：（也可以继承普通构造函数）
  - **super关键字**：
    - 只能在派生类中使用，而且仅限于类构造函数、实例方法和静态方法内部
    - 不能单独引用super关键字，要么用它调用构造函数，要么用它引用静态方法
    - 调用super()会调用父类构造函数，并将返回的实例赋值给this
    - 如果没有定义类构造函数，在实例化派生类时会调用super(),而且会传入所有传给派生类的参数
    - 在类构造函数中，不能在super()之前引用this
    - 如果在派生类中显式定义了构造函数，则要么必须在其中调用super()，要么必须在其中返回一个对象
  - 抽象基类：可供其他类继承，但本身不会被实例化
    - 可以通过new.target实现抽象基类
    - **new.target**：保存通过new关键字调用的类或函数
  - 继承内置类型：ES6类为继承内置引用类型提供了顺畅的机制，开发者可以方便地扩展内置类型
  - 类混入：把不同类的行为集中到一个类。ES6没有显式支持多类继承，可以通过现有特性模拟；



# 9. 代理与反射

## 9.1 代理基础

- 提供了拦截并向基本操作嵌入额外行为的能力

- 代理使用**Proxy**构造函数创建的，接收两个参数：目标对象、处理程序对象；返回代理对象

- Proxy.prototype是undefined

- 使用代理的主要目的是可以定义**捕获器(trap)**。捕获器就是在处理程序对象中定义的“基本操作的拦截器”。

- 只有在代理对象上执行操作才会触发捕获器，在目标对象上执行操作仍然会产生正常的行为

- 处理程序对象中所有可以捕获的方法都有对应的**反射(Reflect)**API方法。这些方法与捕获器拦截的方法具有相同的名称和函数签名，而且也具有与被拦截方法相同的行为

- **捕获器不变式**：捕获处理程序的行为必须遵循“捕获器不变式”(trap invariant)

- **可撤销代理**：通过调用**Proxy.revocable()**方法，创建支持撤销代理对象与目标对象关联的代理。撤销代理操作不可逆，撤销后再调用代理会抛出TypeError

  ```js
  const {proxy, revoke} = Proxy.revocable(target, handler);
  revoke();	//撤销函数调用
  console.log(proxy.foo);		//TypeError
  ```

- 反射API的**状态标记**：很多反射方法返回称作“状态标记”的布尔值，表示意图执行的操作是否成功
  - 提供状态标记的反射方法：
    - Reflect.defineProperty()
    - Reflect.preventExtensions()
    - Reflect.setPrototypeOf()
    - Reflect.set()
    - Reflect.deleteProperty()
- 代理的问题与不足：
  - this值：如果目标对象依赖于对象标识，可能导致问题
  - 有些ECMAScript内置类型可能会依赖代理无法控制的机制，导致在代理上调用某些方法会出错（Date类型方法的执行依赖this值上的内部槽位[[NumberDate]]，代理对象上不存在这个内部槽位，而且这个内部槽位的值也不能通过普通的get()和set()操作访问到，于是代理拦截后本该转发给目标对象的方法会抛出TypeError）

## 9.2 代理捕获器与反射方法

1.  **get(target, property, receiver)** ：在获取属性值的操作中被调用；
   - 返回值无限制
2.  **set(target, property, value, receiver)** ：在设置属性值的操作中被调用；
   - 返回true表示成功，返回false表示失败，严格模式下会抛出TypeError
3.  **has(target, property)** ：在in操作符中被调用；
   - 必须返回布尔值，表示属性是否存在。返回非布尔值会被转型为布尔值
4.  **defineProperty(target, property, descriptor)** ：在Object.defineProperty()中被调用；
   - 必须返回布尔值，表示属性是否成功定义。返回非布尔值会被转型为布尔值
5.  **getOwnPropertyDescriptor(target, property)** ：在Object.getOwnPropertyDescriptor()中被调用；
   - 必须返回对象，或者在属性不存在时返回undefined
6.  **deleteProperty(target, property)** ：在delete操作符中被调用
   - 必须返回布尔值，表示删除属性是否成功。返回非布尔值会被转型为布尔值
7.  **ownKeys(target)** ：在Object.keys()中被调用；
   - 必须返回包含字符串或符号的可枚举对象
8.  **getPrototypeOf(target)** ：在Object.getPrototypeOf()中被调用；
   - 必须返回对象或null
9.  **setPrototypeOf(target, prototype)** ：在Object.setPrototypeOf()中被调用；
   - 必须返回布尔值，表示原型赋值是否成功。返回非布尔值会被转型为布尔值
10.  **isExtensible(target)** ：在Object.isExtensible()中被调用；
    - 必须返回布尔值，表示target是否可扩展。返回非布尔值会被转型为布尔值
11.  **preventExtensions(target)** ：在Object.preventExtensions()中被调用；
    - 必须返回布尔值，表示target是否已经不可扩展。返回非布尔值会被转型为布尔值
12.  **apply(target, thisArg, ...argumentsList)** ：在调用函数时中被调用
    - 返回值无限制
13.  **constuct(target, argumentsList, newTarget)** ：在new操作符中被调用
    - 必须返回一个对象

## 9.3 代理模式

- 使用代理可以在代码中实现一些有用的编程模式：
  1. 跟踪属性访问
  2. 隐藏属性
  3. 属性验证
  4. 函数与构造函数参数验证
  5. 数据绑定与可观察对象



# 10. 函数

- **函数**实际上是**对象**，每个函数都是Function类型的实例；函数名是指向函数对象的指针；
- **函数名**在ECMAScript中就是**变量**，所以函数可以用在任何可以使用变量的地方；可以把函数作为参数传给另一个函数，也可以在一个函数中返回另一个参数；

## 10.1 函数定义

1. **函数声明**：要求写出函数名称

   ```js
   function sum(num1, num2){
       return num1+num2;
   }
   ```

   - JavaScript引擎在任何代码执行之前，会先读取函数声明，并在执行上下文中生成函数定义，即**函数声明提升**

2. **函数表达式**：不要求写出函数名称

   ```js
   let sum = function(num1, num2){
       return num1 + num2;
   }
   ```

   - 要等到代码执行到这一行，才会在执行上下文中生成函数定义，**没有函数声明提升**
   - **匿名函数(兰姆达函数)**：function关键字后面没有标识符

3. **箭头函数**：

   ```js
   let sum = (num1, num2)=>{
       return num1 + num2;
   }
   ```

   - **不能**使用**arguments**、**super**和**new.target**，也不能用作**构造函数**，没有**prototype**属性。

4. **使用Function构造函数**(不推荐)：

   ```js
   let sum = new Function("num1", "num2", "return num1 + num2");
   ```

   - 代码会被解释两次，一次将它当作常规ECMAScript代码，第二次是解释传给构造函数的字符串，影响性能。

## 10.2 函数参数

- ECMAScript函数既不关心传入的参数个数，也不关心参数的数据类型；函数的参数只是为了方便才写出来的，并不是必须写出来；**不能重载**
- 所有参数都是**按值传递**；如果把对象作为参数传递，那么传递的值就是这个对象的引用；
- 参数在内部表现为一个数组。**arguments**对象是一个**类数组对象**(但不是Array的实例)，可以访问**arguments.length属性**
- 非严格模式下，arguments对象的值始终会与对应的命名参数同步。但并不意味着它们访问同一个内存地址，它们在内存还是分开的，只是会保持同步
- 严格模式下，arguments对象的值与对应的命名参数不会同步，且在函数中尝试重写arguments对象会导致语法错误
- **默认参数**：
  - 使用默认参数时，arguments对象不反映参数的默认值，只反映传给函数的参数
  - 默认参数并值并不限于原始值或对象类型，也可以使用调用函数返回的值，计算默认值的函数只有在调用函数但未传相应参数时才会被调用
  - 参数是按顺序初始化的，且遵循“暂时性死区”规则，所以后定义默认值的参数可以引用先定义的参数，但是前面定义的参数不能引用后面定义的
  - 参数也存在于自己的作用域中，它们不能引用函数体的作用域
- **扩展操作符**：可以用于调用函数时传参，也可以用于定义函数参数(因为收集参数的结果可变，所以只能作为最后一个参数)

## 10.3 函数内部

- **arguments**：一个类数组对象，包含调用函数时传入的所有参数
  - **length**属性
  - **callee**属性(严格模式下访问报错)：一个指向arguments对象所在函数的指针
- **this**：标准函数中，this引用的是把函数当成**方法调用的上下文对象**；箭头函数中，this引用的是**定义箭头函数的上下文**
  - 严格模式下，调用函数如果没有指定上下文对象，this值不会指向window。除非使用apply()或call()把函数指定给一个对象，否则this值为undefined
- **new.target**：检测函数是否是使用new关键字调用的
  - 函数正常调用的值为undefined；使用new关键字调用的将**引用被调用的构造函数**

## 10.4 函数属性和方法

- **name**属性(只读)：
  - 所有函数对象都会暴露一个name属性，其中包含关于函数的信息
  - 保存的一个函数标识符，或者说是一个字符串化的变量名；
  - 函数没有名称显示空字符串；
  - 使用Function构造函数创建的，标识成"anonymous"；
  - 获取函数、设置函数、使用bind()实例化，标识符前面加上前缀：get、set、bound
- **caller**(在函数内部使用,严格模式下不能进行赋值，会报错)：引用调用当前函数的函数
- **length**：保存函数定义的命名参数的个数
- **prototype**(不可枚举，使用for-in不会返回这个属性)：保存引用类型所有实例方法的地方，如toString()等
- **apply(函数内this值， 参数数组(Array实例或arguments对象))**：以指定this值来调用函数
- **call(函数内this值， ...逐个函数参数)**：同上，只是传参形式不同
- **bind(函数内this值)**：会创建一个新的函数实例，其this值会被绑定到传给bind()的对象，如：**let objectSayColor =  sayColor.bind(o);**

## 10.5 递归

- 严格模式下运行的代码不能访问arguments.callee，可以使用命名函数表达式实现递归

```js
const factorial = (function f(num){
    if(num <= 1){
        return 1;
    }else{
        return num * f(num-1);
    }
})
```

## 10.6 尾调用优化

- 一项内存管理优化机制，以节省栈空间，非常适合“尾调用”，即外部函数的返回值是一个内部函数的返回值

- 关键：如果函数的逻辑允许基于尾调用将其销毁，则引擎就会那么做

- 条件：

  - 代码在严格模式下执行；
  - 外部函数的返回值是对尾调用函数的调用；
  - 尾调用函数返回后不需要执行额外的逻辑；
  - 尾调用函数不是引用外部函数作用域中自由变量的闭包；

- 没有优化前，每多调用一次嵌套函数，就会多增加一个栈帧；而优化之后，无论调用多少次嵌套函数，都只有一个栈帧；

- 这个优化在递归场景下是最明显的，因为递归代码最容易在栈内存中迅速产生大量栈帧

  ```js
  “use strict";	//递归计算斐波那契数列
  functon fib(n){
      return fibImpl(0, 1, n);
  }
  function fibImpl(a, b, n){
      if(n == 0){
          return a;
      }
      return fibImpl(b, a+b, n-1);
  }
  ```

## 10.7 闭包

- 引用了另一个函数作用域中变量的函数，通常是在嵌套函数中实现。
- 函数执行时，每个执行上下文中都会有一个包含其中变量的对象。全局上下文中的叫变量对象，它会在代码执行期间始终存在。而函数局部上下文中的叫活动对象，只在函数执行期间存在。在定义函数时，就会为它创建作用域链，预装载全局变量对象，并保存在内部[[Scope]]中。在调用这个函数时，会创建相应的执行上下文，然后通过复制函数的[[Scope]]来创建其作用域链。接着会创建函数的活动对象函数的活动对象(用作变量对象)并将其推入作用域链的前端。作用域链其实是一个包含指针的列表，每个指针分别指向一个变量对象，但物理上并不会包含相应的对象。
- 外部函数的活动对象并不能在它执行完毕后销毁，因为匿名函数的作用域链中仍然有对它的引用。在外部函数执行完毕后，其执行上下文作用域链会销毁，但它的活动对象仍然会保留在内存中，直到匿名函数被销毁后才会被销毁。
- 因为闭包会保留它们包含函数的作用域，所以比其他函数更占用内存，过度使用闭包可能导致内存过度占用
- 闭包在被函数返回之后，其作用域会一直保存在内存中，直到闭包被销毁
- **闭包中的this**：
  - 内部函数使用**非箭头函数定义**：this对象会在**运行时绑定到执行函数的上下文**；如果在全局函数中调用，则this非严格模式下为window，严格模式下为undefined；如果作为某个对象的方法调用，则this等于这个对象；（每个函数在被调用时都会自动创建this和arguments，因此内部函数永远不可能直接访问外部函数的着两个变量，可以通过把它们保存到闭包可以访问的变量中来进行访问，如：let that = this;）
  - 内部使用**箭头函数定义**：箭头函数没有自己的this，所以它的this值为**外部函数的this**；

## 10.8 立即调用的函数表达式

- (IIFE,Immediately Invoke Function Expression)，立即调用的匿名函数，类似函数声明，但由于被包含在括号中，所以会被解释为函数表达式（以前常用于模拟块级作用域）
- 如果不在包含作用域中将返回值赋值给一个变量，则其包含的所有变量都会被销毁

## 10.9 私有变量

- 任何定义在函数或块中的变量，都可以认为是私有的，因为在这个函数或块的外部无法访问其中的变量。

- 虽然JavaScript中没有私有对象属性的概念，但可以使用闭包实现公共方法，访问位于包含作用域中的变量

- 私有变量包括函数参数、局部变量、以及函数内部定义的其他函数。

- **特权方法**：能够访问函数私有变量（及私有函数）的公有方法

  1. **在构造函数中实现**：（缺点：每个实例都会重新创建一遍新方法）

     ```js
     function MyObject(){
         let privateVariable = 10;
         function privateFunction(){
             return false;
         }
         //特权方法
         this.publicMethod = function(){
             privateVariable++;
             return privateFunction();
         }
     }
     ```

  2. **使用私有作用域定义私有变量和函数（静态）**（私有变量和函数是由实例共享的，可以利用原型更好地重用代码，只是每个实例没有了自己的私有变量）：这个模式定义的构造函数没有使用函数声明，使用的是函数表达式。函数声明会创建内部函数，在这里不是必需的。这里声明MyObject没有使用任何关键字，因为不使用关键字声明的变量会创建在全局作用域中，所以MyObject变成了全局变量，可以在这个私有作用域外部被访问。注意在严格模式下给未声明的变量赋值会导致错误。

     ```js
     (function(){
         let privateVariable = 10;
         function privateFunction(){
             return false;
         }
         //构造函数
         MyObject = function(){};
         //公有和特权方法
         MyObject.prototype.publicMethod = function(){
             privateVariable++;
             return privateFunction();
         }
     })();
     ```

  3. **模块模式**：在单例对象(只有一个实例的对象)基础上加以扩展，使其通过作用域链来关联私有变量和特权方法

     ```js
     let singleton = function(){
         let privateVariable = 10;
         function privateFunction(){
             return false;
         }
         //特权/公有方法和属性
         return {
             publicProperty: true,
             publicMethod(){
                 privateVariable++;
             	return privateFunction();
             }
         }
     }
     ```

  4. **模块增强模式**：在返回对象之前先对其进行增强。适合单例对象需要是某个特定类型的实例，但又必须给它添加额外属性或方法的场景。

     ```js
     let singleton = function(){
         let privateVariable = 10;
         function privateFunction(){
             return false;
         }
         //创建对象
         let object = new CustomType();
         //添加特权/公有方法和属性
         object.publicProperty = true;
         object.publicMethod = function(){
             privateVariable++;
            	return privateFunction();
         };
         //返回对象
         return object;
     }
     ```



# 11. 期约与异步函数

## 11.1 异步编程

- **同步行为**对应内存中顺序执行的处理器指令；**异步行为**类似于系统中断，即当前进程外部的实体可以触发代码执行。

- 以往的异步编程模式-嵌套异步回调（异步返值又依赖另一个异步返回值，回调地狱）

  ```js
  function double(value, success, failure){
      setTimeout(()=>{
          try{
              if (typeof value !== 'number'){
                  throw 'Must provide number as first argument';
              }
              success(2*value);
          }catch(e){
              failure(e);
          }
      }, 1000);
  }
  const successCallback = (x) =>{
      double(x, (y) => console.log(`Success: ${y}`));
  };
  const failureCallback = (e) => console.log(`Failure: ${e}`);
  double(3, successCallback, failureCallback);	//Success: 12 (大约100毫秒之后)
  ```

## 11.2 期约Promise

- 期约是对尚不存在结果的一个替身，主要功能是为异步代码提供了清晰的抽象，可以用期约表示异步执行的代码块，也可以用期约表示异步计算的值

- 通过new操作符来实例化，创建新期约时需要传入**执行器(executor)函数**作为参数。

- 期约3种状态：**待定**(pending)、**兑现/解决**(fulfilled/resolved)、**拒绝**(rejected)

- 只要从待定转换为兑现或拒绝，期约的状态就不在改变，而且，也不能保证期约必然会脱离待定状态。

- 期约异步特性：它们是同步对象（在同步执行模式中使用），但也是异步执行模式的媒介

- **执行器函数**主要职责：

  1. 初始化期约的异步行为（执行器函数是同步执行的）
  2. 控制状态的最终转换（通过调用它的两个函数参数实现：**resolve()**、**reject()**）

- 方法：

  - **Promise.resolve()**静态方法：实例化一个解决的期约
    - 如果传入的参数本身是一个期约，那它的行为就类似于一个空包装（幂等方法）
    - 这个幂等性会保留传入期约的状态
  - **Promise.reject()**：实例化一个拒绝的期约并抛出一个**异步错误**(这个错误不能通过try/catch捕获，只能通过拒绝处理程序捕获)
    - 如果传入一个期约对象，则这个期约会成为它返回的拒绝期约的理由
  - **Promise.all()**：将多个期约实例组合成一个期约，创建的期约会在一组期约全部解决之后再解决
    - 接收可一个可迭代对象，返回一个新期约
    - 如果至少有一个包含的期约待定，则合成的期约也会待定。如果有一个包含的期约拒绝，则合成的期约也会拒绝
    - 如果所有期约都成功解决，则合成期约的解决值就是所有包含期约解决值的数组，按照迭代器顺序。
    - 如果有期约拒绝，则第一个拒绝的期约会将自己的理由作为合成期约的拒绝理由，之后再拒绝的期约不会影响最终期约的拒绝理由。合成的期约会静默处理所有包含期约的拒绝操作。
  - **Promise.race()**：将多个期约实例组合成一个期约，返回一个包装期约，是一组集合中最先解决或拒绝的期约的镜像
    - 接收一个可迭代对象，返回一个新期约
    - 如果有期约拒绝，只要他是第一个落定的，就会成为合成期约的拒绝理由，之后再拒绝的期约不会影响最终期约的拒绝理由。合成的期约会静默处理所有包含期约的拒绝操作。
  - 在ECMAScript暴露的异步结构中，任何对象都有一个then()方法，这个方法被认为实现了Thenable接口。
  - **Promise.prototype.then(onResolved处理程序, onRejected处理程序)**：为期约实例添加处理程序的主要方法
    - 传的任何非函数类型的参数都会被静默忽略
    - 返回一个新的期约实例，基于onResolved处理程序的返回值来构建，返回值会通过Promise.resolve()包装来生成新期约，如果没有提供这个处理程序，则包装上一个期约解决之后的值
    - 抛出异常会返回拒绝的期约
    - onResolved处理程序返回的值也会被Promise.resolve()包装
  - **Promise.prototype.catch(onRejected处理程序)**：给期约添加拒绝处理程序（语法糖，相当于调用Promise.prototype.then(null, onRejected)）
    - 返回一个新期约实例
  - **Promise.prototype.finally(onFinally处理程序)**：给期约添加onFinally处理程序，在期约转换为解决或拒绝状态时都会执行。
    - onFinally处理程序没有办法直到期约状态是解决还是拒绝，所以这个方法主要用于添加清理代码
    - 返回一个新期约实例，大多数情况下表现为父期约的传递
    - 如果返回的是一个待定的期约，或者onFinally处理程序抛出了错误（显示抛出错误或返回了一个拒绝期约），则会返回相应的期约（待定或拒绝）；
    - 返回期待期约时，只要期约一解决，新期约仍然会原样后传初始的期约
  - 

- **”非重入”(non-reentrancy)特性**：当期约进入落定状态时，与该状态相关的处理程序仅仅会被排期，而非立即执行。跟在添加这个处理程序的代码之后的同步代码一定会在处理程序之前先执行。（非重入适用于onResolved/onRejected处理程序、catch()处理程序和finally()处理程序）

- 如果给期约添加了多个处理程序，当期约状态变化时，相关处理程序会按照添加它们的顺序依次执行，无论是then()、catch()还是finally()

- 扩展：ES6不支持取消期约和进度通知，一个主要原因就是这样会导致期约连锁和期约合成过度复杂化

  - **期约取消**：可以在现有实现基础上提供一种临时性封装，以实现取消期约的功能

    ```html
    <button id="start">Start</button>
    <button id="cancel">Cancel</button>
    <script>
        class CancelToken {
            constructor(cancelFn){
                this.promise = new Promise((resolve, reject) =>{
                    cancelFn(()=>{
                        setTimeout(console.log, 0, "delay cancelled");
                        resolve();
                    });
                });
            }
        }
        const startButton = document.querySelector('#start');
        const cancelButton = document.querySelector('#cancel');
        function cancellableDelayedResolve(delay){
            setTimeout(console.log, 0, "set delay");
            return new Promise((resolve, reject) =>{
                const id = setTimeout(()=>{
                    setTimeout(console.log, 0, "dealyed resolve");
                    resolve();
                }, delay);
                const cancelToken = new CancelToken((cancelCallback) => cancelButton.addEventListener("click", cancelCallback));
                cancelToken.promise.then(()=>clearTimeout(id));
            })
        }
         startButton.addEventListener("click", cancellableDelayedResolve(1000));
    </script>
    ```

  - **期约进度通知**：ES6期约并不支持进度追踪，但是可以通过扩展来实现

    ```js
    class TrackablePromise extends Promise{
        constructor(executor){
            const notifyHandlers = [];
            super((resolve, reject)=>{
                return executor(resolve, reject, (status)=>{
                    notifyHandlers.map((handler)=> handler(status));
                });
            });
        }
        notify(notifyHandler){
            this.notifyHandlers.push(notifyHandler);
            return this;
        }
    }
    let p = new TrackablePromise((resolve, reject, notify)=>{
        function countdown(x){
            if(x>0){
                notify(`${20*x}% remaining`);
                setTimeout(()=> countdown(x-1), 1000);
            }else{
                resolve();
            }
        }
        countdown(5);
    });
    p.notify((x)=> setTimeout(console.log, 0, 'progress:', x));
    p.then(()=> setTimeout(console.log, 0, 'completed'));
    ```

## 11.3 异步函数async/await

- 旨在解决利用异步结构组织代码的问题，是将期约应用于JavaScript函数的结果，可以暂停执行而不阻塞主线程
- **async**：用于**声明异步函数**，可以让函数具有异步特征，但总体上其代码仍然是同步求值的
  - 异步函数如果使用return关键字返回了值（没有return则返回undefined），这个值会被Promise.resolve()包装成一个期约对象
  - 异步函数的返回值期待（但并不要求）一个实现Thenable接口的对象，但常规的值也可以
  - 在异步函数中抛出错误会返回拒绝的期约，但是拒绝期约的错误不会被异步函数捕获，会抛出未捕获错误
- **await**：暂停异步函数代码的执行，等待期约解决
  - 必须在异步函数中使用，且异步函数的特质不会扩展到嵌套函数，因此await关键字只能直接出现在异步函数的定义中
  - 它会让出JavaScript运行时的执行线程，这个行为与生成器函数中的yield关键字一样
  - 期待（但并不要求）一个实现Thenable接口的对象，但常规的值也可以
  - 单独的Promise.reject()不会被异步函数捕获，而会抛出未捕获错误；对拒绝的期约使用await则会释放错误值（将拒绝期约返回）
  - async/await中真正起作用的是await，async不过是个标识符
  - JavaScript运行时在碰到await关键字时，会记录在哪里暂停执行，等到await右边的值可用了，JavaScript运行时会向消息队列中推送一个任务，这个任务会恢复异步函数的执行
- 期约与异步函数的功能有相当程度的重叠，但它们在内存中的表示差别很大。JavaScript引擎在创建期约时尽可能保留完整的调用栈，会带来额外的消耗。



# 12. BOM

## 12.1 window对象

- 表示浏览器的实例，在浏览器中既是**ECMAScript中的Global对象**，也是**浏览器窗口的JavaScript接口**
- 因为window对象的属性在全局作用域中有效，所以很多浏览器API及相关构造函数都以window对象属性的形式暴露出来
- 通过var声明的所有全局变量和函数都会变成window对象的属性和方法
- 属性和方法：
  1. **top**：始终指向最上层（最外层）**窗口**，及浏览器窗口本身
  2. **parent**：始终指向当前窗口的父窗口，如果当前窗口是最上层窗口，则parent等于top(都等于window)
  3. **self**：始终指向window
  4. **screenLeft/screenTop**：表示**窗口**相对于屏幕左侧和顶部的**位置**，返回值的单位是css像素
  5. **moveTo(x,y)/moveBy(相对x，相对y)**：移动窗口
  6. **deviecePixelRatio**：表示物理像素与逻辑像素之间的缩放系数（DPI表示单位像素密度，window.deviecePixelRatio实际上与DPI是对应的）
  7. **outerWidth/outerHeight**：返回浏览器**窗口**自身的**大小**（不管在最外层window还是在窗格frame中使用）
  8. **innerWidth/innerHeight**：返回浏览器窗口中页面视图的大小（不包含浏览器边框和工具栏）
     - **document.documentElement.clientWidth/clientHeight**：返回页面视口的宽度和高度
     - **document.compatMode**=="CSS1Compat"：检查页面是否处于标准模式
     - 移动设备上，innerWidth/innerHeight返回视口的大小，也就是屏幕上页面可视区域的大小；Mobile Internet Explorer中，document.documentElement.clientWidth/clientHeight提供同样的信息。页面缩放时，值也会相应变化。
     - 其他移动浏览器中，document.documentElement.clientWidth/clientHeight返回的布局视口大小，即渲染页面的实际大小；Mobile Internet Explorer中，document.body.clientWidth/clientHeight保存的布局视口信息。页面缩放时，值也会相应变化。
     - 布局视口是相对于可见视口的概念，可见视口只能显示整个页面的一小部分。
  9. **resizeTo(w,h)/resizeBy(相对w，相对h)**：调整窗口大小
  10. **pageXoffset/scrollx/pageYoffset/scrollY**：度量文档相对于**视口**滚动距离
  11. **scroll(x,y)/scrollTo(x,y)/scrollBy(相对x，相对y)**：滚动页面
      - 参数也都接收一个ScrollToOptions字典，除了提供偏移值left、top，还可以通过behavior属性告诉浏览器是否平滑滚动
  12. **open(要加载的URL，目标窗口，特性字符串，表示新窗口在浏览器历史记录中是否替代当前加载页面的布尔值)**：可以用于导航到指定URL，也可以用于打开新浏览器窗口
      - 第二个参数可以是已经存在的窗口或窗格(frame)的名字，也可以是一个特殊的窗口名(如\_self,\_parent,\_top,\_blank)
      - 第二个参数如果不是已有窗口，则会打开一个新窗口或标签页
      - 特性字符串，用于指定新窗口的配置；如果打开的不是新窗口，则忽略第三个参数
      - 方法返回一个对新建窗口的引用，这个对象与普通window对象没有区别。
        - 引用对象有**close()**方法可以关闭新打开的窗口，但是只能用于windwo.open()创建的弹出窗口。
        - 弹出窗口可以调用**top.close()**来关闭自己。关闭窗口以后，窗口的引用虽然还在，但只能用于检查其**closed属性**
        - 新创建窗口的window对象有个**opener属性**，指向打开它的窗口。这个属性只在弹出窗口的最上层window(top)对象有定义，是指向调用window.open()打开它的窗口或窗格的指针。
      - 在浏览器扩展或其他程序屏蔽弹窗时，window.open()通常会抛出错误。因此要准确检测弹窗是否被屏蔽，除了检测window.open的返回值是否为null，还要把它用try/catch包装起来
  13. **setTimeout()/setInterval()定时器**：
      - setTimeout用于指定在一定时间后执行某些代码；setInterval用于指定每隔一段时间执行某些代码；
      - 参数：要执行的代码、在执行回调函数前等待的时间(毫秒)
      - setTimeout返回唯一标识符ID，配合**clearTimout(id)**可取消；setInterval也返回一个循环定时ID，配合**clearInteval(id)**可取消循环定时
      - JavaScript维护了一个任务队列，其中的任务会按照添加到队列的先后顺序执行。setTimeout第二个参数只告诉JavaScript引擎在指定的毫秒数过后把任务添加到这个队列。如果队列是空的，则会立即执行代码；如果队列不是空的，则代码必须等待前面的任务执行完才能执行。
      - 所有超时执行的代码（函数）都会在全局作用域中的一个匿名函数中运行，因此函数中的this值在非严格模式下始终指向window，在严格模式下是undefined。如果给setTimeout提供了一个箭头函数，那么this会保留为定义它时所在的词汇作用域。
  14. **alert()/confirm()/prompt(要显示给用户的文本，文本框的默认值)**：让浏览器调用系统对话框向用户显示消息
      - **同步的模态对话框**，即在它们显示的时候，代码会停止执行
      - 这些对话框与浏览器中显示的网页无关，而且也不包含HTML，它们的外观由操作系统或者浏览器决定，无法使用CSS设置。
      - 警告框(alert)通常用于向用户显示一些它们无法控制的消息；确认框(confirm)通常用于让用户确认执行某个操作；提示框(prompt)用于提示用户输入消息；
      - confirm返回值：true=>单击了OK按钮；false=>单击了Cancel按钮或者X图标关闭确认框
      - prompt返回值：文本框值=>单击了OK按钮；null=>单击了Cancel按钮或者X图标关闭确认框
      - **find()/print()**：**异步显示对话框**，即控制权会立即返回给脚本；对应用户在浏览器菜单上选择”查找“/“打印“显示的对话框；不会返回任何有关用户在对话框中执行了什么操作的信息，因此很难加以利用；因为对话框时异步的，所以浏览器的对话框计数器不会涉及它们，而且用户选择禁用对话框对它们也没有影响。

## 12.2 location对象

- 提供了当前窗口中加载文档的信息，以及通常的导航功能。

- 既是window的属性，也是document的属性：window.location和document.location指向同一个对象

- 不仅保存着当前加载文档的信息，也保存着把URL解析为离散片段后能够通过属性访问的信息

  ![location对象属性](./Screenshot_20250920_174943.jpg)

- **URLSearchParams**：提供了一组标准API方法，通过它们可以检查和修改查询字符串。

  - 给URLSearchParams构造函数传入一个查询字符串(location.search)，就可以创建一个实例
  - 实例方法：**has()、get()、set()、delete()、toString()**等
  - 大多数支持URLSearchParams的浏览器也支持将URLSearchParams的实例用作可迭代对象

- 操作地址：

  - **assign(URL)**：立即启动导航到新URL的操作，同时在浏览器历史记录中增加一条记录；设置location.href或window.location执行一样的操作；
    - 同步修改当前加载的页面的location属性有：**hash、search、hostname、pathname、port**。除了hash之外，只要修改location的一个属性，就会导致页面重新加载URL
  - **replace(URL)**：重新加载URL，加载后不会增加历史记录（因此，用户不能回到前一页）
  - **reload()**：重新加载当前显示页面。
    - 不传参数，页面可能会以最有效的方法重新加载；如果页面自上次请求以来一直没有修改过，浏览器可能会从缓存中加载页面。如果想强制从服务器重新加载，可以给reload传true。
    - 脚本中位于reload()调用之后的代码可能执行也可能不执行，这取决于网络延迟和系统资源等因素。因此，最好把reload()作为最后一行代码。

## 12.3 navigator对象

- 客户端标识浏览器的标准，navigator对象的属性通常用于确定浏览器的类型

  ![navigator对象属性方法](./Screenshot_20250920_181033.jpg)

  ![navigator对象属性方法](./Screenshot_20250920_181107.jpg)

- **检测插件**：检测浏览器是否安装了某个插件，可以通过**plugins**数组来确定

  - 数组中的每一项包括：name(插件名)、description(插件介绍)、filename(插件文件名)、length(由当前插件处理的MIME类型数量)
  - plugins数组中的每个插件对象还有一个MimeType对象，可以通过中括号访问。每个MimeType对象有四个属性：description描述MIME类型，enabledPlugin是指向插件对象的指针，suffixes是该MIME类型对对应扩展名的逗号分隔的字符串，type是完整的MIME类型字符串。
  - plugins有个**refresh()**方法，用于刷新plugins属性以反映新安装的插件。这个方法接收一个布尔值参数，表示刷新时是否重新加载页面。如果传入true，则所有包含插件的页面都会重新加载，否则，只有plugins会更新，但页面不会重新加载。

- 注册处理程序：**registerProtocolHandler()**方法可以把一个网站注册为处理某种特定类型信息应用程序

## 12.4 screen对象

- 保存的纯粹是客户端能力，也就是浏览器窗口外面的客户端显示器的信息

  ![screen对象属性](./Screenshot_20250920_182507.jpg)

## 12.5 history对象

- 表示当前窗口首次使用以来用户的导航历史记录，这个对象不会暴露用户访问过的URL，但可以通过它在不知道实际URL的情况下前进和后退；history对象通常被用于创建”后退“和”前进“按钮，以及确定页面是不是用户历史记录中的第一条记录。
- 导航：
  - **go()**：可以在用户历史记录中沿任何方向导航，可以前进也可以后退，简写方法：**back()**、**forward()**
  - **length**：表示历史记录中有多少个条目
- 历史状态管理：
  - **hashchange事件**：在页面URL的散列变化时被触发
  - 状态管理API：改变浏览器URL而不会加载新页面
    - **pushState(state对象， 新状态标题， 相对URL)**：方法执行后，状态信息会被推到历史记录中，浏览器地址也会改变以反映新的相对URL
    - **popstate事件**：事件对象有一个state属性，其中包含通过pushState()第一个参数传入的state对象。
    - **state属性**：保存当前的状态对象
    - **replaceState(state对象， 新状态标题)**：更新状态。更新状态不会创建新历史，只会覆盖当前状态
    - 使用HTML5状态管理时，要确保通过pushState()创建的每个”假“URL背后都对应着服务器上一个真实的物理URL。否则，单击”刷新“按钮会导致404报错。所有单页面应用程序(SPA,Single Page Application)框架都必须通过服务器或客户端的某些配置解决这个问题。



# 13. 客户端检测

在选择客户端检测方法时，首选是使用能力检测，特殊能力检测要放在次要位置，作为决定代码逻辑的参考。用户代理检测是最后一个选择，因为它过于依赖用户代理字符串。

## 13.1 能力检测(特性检测)

- 在JavaScript运行时中使用一套简单的检测逻辑，测试浏览器是否支持某种特性
- 不要求事先知道特定浏览器信息，只需检测自己关心的能力是否存在即可
- 关键：
  - 应该先检测最常用的方式，测试最常用的方案可以优化代码执行，在多数情况下都可以避免无谓检测
  - 必须检测切实需要的特性，某个能力存在并不代表别的能力也存在
- 最有效的场景是检测能力是否存在的同时，验证其是否能够展现出预期的行为（尽量使用**typeof**操作符）
- 如果应用程序需要使用特定的浏览器能力，那么最好集中检测所有能力，而不是等到用的时候再重复检测
- 可以根据对浏览器特性的检测并与已知特性对比，确认用户使用的是什么浏览器。这样可以获得比用户代码嗅探更准确的结果。
- 使用能力检测而非用户代理检测的**优点**在于，伪造用户代理字符串很简单，而伪造能够欺骗能力检测的浏览器特性却很难
- **局限**：通过检测一种或一组能力，并不总能确定使用的是哪种浏览器，不能精确地反映特定的浏览器或版本。
- 能力检测最适合用于决定下一步该怎么办，而不一定能够作为辨识浏览器的标志

## 13.2 用户代理检测

- 通过浏览器的用户代理字符串确定使用的是什么浏览器。用户代理字符串包含在每个HTTP请求的头部，在Javascript中可以通过**navigator.userAgent**访问
- 在客户端，用户代理检测被认为是不可靠的，只应该在没有其他选项时再考虑
- 想要知道自己代码运行在什么浏览器上，大部分开发者会分析window.navigator.userAgent返回的字符串（也可以使用第三方**用户代理解析程**序）
- 通过检测用户代理来识别浏览器并不是完美的方式，毕竟这个字符串是可以造假的。
- 通过用户代理字符串，可以推断出的环境信息：**浏览器、浏览器版本、浏览器渲染引擎、设备类型(桌面/移动)、设备生产商、设备型号、操作系统、操作系统版本**

## 13.3 软件与硬件检测

- 浏览器也提供了一些软件和硬件相关信息。这些信息通过navigator和screen对象暴露出来。利用这些API，可以获取关于操作系统、浏览器、硬件、设备位置、电池状态等方面的准确信息。不过浏览器支持还不够好，远未达到标准化的程度（**建议在使用之前先检测它们是否存在**）
- 识别浏览器与操作系统：navigator和screen对象提供了关于**页面所在软件环境的信息**
  - **navigator.oscpu**：字符串，通常对应用户代理字符串中操作系统/系统架构相关信息
  - **navigator.vendor**：字符串，通常包含浏览器开发商信息
  - **navigator.platform**：字符串，通常表示浏览器所在的操作系统
  - **screen.colorDepth/pixelDepth**：显示器每像素颜色的位深
  - **screen.orientation**：返回一个screenOrientation对象，包含Screen Orientation API定义的屏幕信息(angel、type属性主要用于确定设备旋转后浏览器的朝向变化)
- 浏览器元数据：navigator对象暴露了一些API，可以提供**浏览器和操作系统的状态信息**
  - **Geolocation API**：navigator.geolocation属性暴露了Geolocation API，可以让浏览器脚本感知当前设备的**地理位置**
    - 只在安全执行环境（通过HTTPS获取的脚本）中可用
    - getCurrentPosition()方法：获取浏览器当前的位置
  - **Connection State和NetworkInformation API**：
    - 浏览器会跟踪**网络连接状态**并以两种方式暴露这些信息：连接事件(online、offline)和.onLine属性
    - navigator对象还暴露了NetworkInformation API，可以通过navigator.connection属性使用，提供了一些只读属性，并为连接属性变化事件处理程序定义了一个事件对象
  - **Battery API**：浏览器可以访问设备**电池及充电状态**的信息。
    - navigator.getBattery()方法：返回一个期约实例，解决为一个BatteryManager对象。
- **硬件**：浏览器硬件检测的能力相当有限，不过，navigator对象还是通过一些属性提供了基本信息
  - **navigator.hardwareConcurrency**：返回浏览器支持的逻辑处理器核心数量，表示浏览器可以并行执行的最大工作线程数量，不一定是实际CPU核心数
  - **navigator.oscpu**：返回设备大致的系统内存大小
  - **navigator.oscpu**：返回触摸屏支持的最大关联触电数量



# 14. DOM

- 文档对象模型(DOM,Document Object Model)是HTML和XML文档的编程接口。它与浏览器中的HTML网页相关，并在JavaScript中提供了DOM API。

## 14.1 节点层级

- 任何HTML或XML文档都可以用DOM表示为一个由节点构成的层级结构。节点分很多类型，每种类型对应着文档中不同的信息和标记，也都有自己不同的特性、数据和方法，而且与其他类型有某种关系。这些关系构成了层级，让标记可以表示为一个以特定节点为根的树形结构。
- 每个文档只能有一个文档元素。在HTML页面中，文档元素始终是\<html\>元素；在XML文档中，则没有这样预定义的元素，任何元素都可能成为文档元素。
- HTML中的每段标记都可以表示为树形结构中的一个节点。
- **节点类型**：
  1. **Node**类型：
     - DOM Level1描述了名为Node的接口，这个接口是所有DOM节点必须实现的。Node接口在JavaScript中被实现为Node类型。在JavaScript中，**所有节点类型都继承Node类型**，因此所有类型共享相同的基本属性和方法。
     - 属性和方法：
       - **nodeType**：表示该节点的类型(12个数值常量,12种节点类型,但是浏览器并不支持所有节点类型)
       - **nodeName**和**nodeValue**：保存着有关节点的信息，值取决于节点类型（使用前最好先检测节点类型）
       - **childNodes**：包含一个NodeList实例，其中的每个节点都是同一列表中其他节点的同胞节点
         - NodeList是一个类数组对象(不是Array实例)，存储可以按位置存取的有序节点（可以通过中括号、item()访问其中元素，拥有length属性）
         - NodeList是一个对DOM结构的查询，因此DOM结构的变化会自动地在NodeList种反映出来（NodeList是**实时的活动对象**，而不是第一次访问时所获得内容的快照）
       - **parentNode**：指向其DOM树中的父元素
       - **previousSibling**和**nextSibling**：在childNodes列表的节点中导航（如果childNodes只有一个节点，则previousSibling和nextSibling都为null）
       - **firstChild**和**lastChild**：分别指向childNodes中第一个和最后一个子节点
       - **ownerDocument**：指向代表整个文档的文档节点
       - **hasChildNodes()**：判断是否有子节点
       - **appendChild(新添加的节点)**：在childNodes列表末尾添加节点，返回新添加的节点
         - 如果把文档中已经存在的节点传给appendChild()，则这个节点会从之前的位置被转移到新位置
       - **insertBefore(要插入的节点,参照节点)**：把节点放到childNodes中的特定位置，要插入的节点变成参照节点的前一个同胞节点并返回
         - 如果参照节点是null，则childNodes()和appendChild()效果相同
       - **replaceChild(要插入的节点,要替换的节点)**：要替换的节点会被返回并从文档树中完全移除，要插入的节点会取而代之
         - 所有关系指针都会从被替换的节点复制过来
       - **removeChild(要移除的节点)**：返回被移除的节点
       - **cloneNode(是否深复制)**：返回与调用它的节点一摸一样的节点
         - 传入true时，复制节点及其整个子DOM树；传入false时，只复制调用该方法的节点
         - 复制返回的节点属于文档所有，但尚未指定父节点，所以可称为孤儿节点
         - 不会复制添加到DOM节点的JavaScript属性，如事件处理程序；**只复制HTML属性**
       - **normalize()**：处理文档子树中的文本节点。
         - 检测这个节点的所有后代，对于不包含文本的文本节点(删除)或文本节点之间互为同胞关系的节点(合并为一个文本节点)
  2. **Document**类型
  3. **Element**类型
  4. **Text**类型
  5. **Comment**类型
  6. **CDATASection**类型
  7. **DocumentType**类型
  8. **DocumentFragment**类型
  9. **Attr**类型

## 14.2 DOM编程

## 14.3 MutationObserver接口



# 15. DOM扩展

## 15.1 Selectors API

## 15.2 元素遍历

## 15.3 HTML5

## 15.4 专有扩展



# 16. DOM2和DOM3

## 16.1 DOM的演进

## 16.2 样式

## 16.3 遍历

## 16.4 范围

