### Rust & openOS：开源操作系统学习

#### 0. Introduction

- 本次活动分为两个阶段：
  - 第一阶段：线上自学Rust编程和OS基础，并进行[Rust语言编程自学](https://github.com/LearningOS/rust-based-os-comp2022/blob/main/scheduling.md#step-0-自学rust编程大约714天)、[Rust语言编程实验](https://github.com/LearningOS/rustlings)、[RISC-V处理器学习](https://github.com/LearningOS/rust-based-os-comp2022/blob/main/scheduling.md#step-1-自学risc-v系统结构大约27天)和[Rust-based OS Kernel学习&实验](https://github.com/LearningOS/rust-based-os-comp2022/blob/main/scheduling.md#step-2-基于rust语言进行操作系统内核实验--based-on-qemu-大约1431天)（2022.11.01～2022.12.15）
  - 第二阶段：线上自学并挑战[OS Kernel supporting Linux Apps实验](https://github.com/LearningOS/oscomp-kernel-training)（2022.12.16～2023.02.01） ,主要是**用Rust语言设计实现支持Linux APP的OS Kernel**，大约要支持50个左右的Linux Syscalls，能通过上百个Linux App测试用例。
- 第一阶段活动安排：
  - step0 自学rust编程（大约7~14天）
  - step1 自学risc-v系统结构（大约2~7天）
  - step2 基于Rust语言进行操作系统内核实验——based on qemu（大约14~31天）

#### 1. Step 0 自学Rust编程

前提条件： 要求有基本数据结构，算法基础，相对了解或熟悉C语言等编程.

1. 自学：[阅读书籍/课程/视频等资源汇总](https://github.com/rcore-os/rCore/wiki/study-resource-of-system-programming-in-RUST)
   - 推荐：[Rust语言圣经(Rust教程 Rust Course和配套练习)](https://course.rs/)
   - 推荐：[Rust速查表（cheatsheet）](https://cheats.rs/) 该项目不仅提供了基础的语法速查，还有执行顺序详解和编写时需要关注的注意事项。项目还包含了示例代码（EX）、书籍（BK）、标准（STD）等相关资料的扩展。
   - 推荐：[清华计算机系大一学生2022暑期课程：Rust程序设计训练（有课程视频）](https://lab.cs.tsinghua.edu.cn/rust/)
2. 自学编程
   - [Rust-lang Lab Test based on Rustlings](https://classroom.github.com/a/YTNg1dEH)（采用Github Classroom模式的Rustling小练习，点击上述链接，形成自己的练习用repo）
     - 要求：**必须完成** 。每完成几个小练习，就执行 `git add; git commit -m"update"; git push` 命令，把更新提交到GithubClassroom的CI进行自动评测。要求小练习全部通过GithubClassroom的CI自动评测。
     - [学习系列视频：Rust中文社群线上学习室--通过 Rustlings 学 Rust](https://space.bilibili.com/24917186/video)
   - （Option）[32 Rust Quizes](https://dtolnay.github.io/rust-quiz/1)
     - 要求：小练习全部通过。（**非必须完成**）
   - （Option）[exercisms.io 快速练习(88+道题目的中文详细描述)](http://llever.com/exercism-rust-zh/index.html)
     - 要求：大部分练习会做或能读懂。（**非必须完成**）
     - [exercism.io官方站点](https://exercism.io/)

-----

开始自学Rust语法

- Rust 教程 | 菜鸟教程
  - Rust Introduction
    - Rust语言的特点
      - 高性能：Rust速度惊人且内存利用率极高。由于没有运行时和垃圾回收，它能够胜任对性能要求特别高的服务，可以在嵌入式设备上运行，还能轻松和其他语言集成。
        - *疑问？没有运行时是什么意思*
      - 可靠性：Rust丰富的类型系统和所有权模型保证了内存安全和线程安全，在编译期就能够消除各种各样的错误
      - 生产力：Rust拥有出色的文档、友好的编译器和清晰的错误提示信息，还集成了一流的工具——包管理器和构建工具，智能地自动补全和类型检验的多编辑器支持，以及自动化格式代码等
    - Rust的应用，可用于开发
      - 传统命令行程序
      - Web应用
      - 网络服务器
      - 嵌入式设备



---

### Rust 语言圣经

#### 1. Rust 安装与尝试

- Cargo.toml

  - 是cargo特有的 **项目数据描述文件**。
  - 存储了项目的所有元配置信息

- Cargo.lock

  - cargo工具根据同一项目的toml文件生成的**项目依赖详细清单**

- package 配置段落

  - ```toml
    [package]
    name = "world_hello"
    version = "0.1.0"
    edition = "2021"
    ```

- 定义项目依赖

  ```toml
  [dependencies]
  rand = "0.3"
  hammer = { version = "0.5.0"}
  color = { git = "https://github.com/bjz/color-rs" }
  geometry = { path = "crates/geometry" }
  ```



##### 1.1 Rust 数据类型

- 整数型：Integer
  - 有符号、无符号，i、u
  - 8/16/32/64/128、size
  - 表述方式：
    - 十进制：98_222
    - 十六进制：0x
    - 八进制：0o
    - 二进制：0b
    - 字节：b‘A’
- 浮点型：
  - 32/64位浮点数
- 布尔型：
  - bool：true、false
- 字符型：
  - char：4字节大小，Unicode标量值
- 复合类型：
  - 元组：`()`
  - 数组：`[]`

##### 1.2 Rust注释

- 注释方式：
  - //
  - /* */
- tips:
  - 优雅的文档注释技巧：///

##### 1.3 Rust 函数

- fn <function_name> (<params>) <function body>
  e.g.

  ```rust
  fn main() {
      println!("Hello, world!");
      another_function();
  }
  
  fn another_function() {
      println!("Hello, runoob!");
  }
  ```

  - Rust 不在乎在何处定义了函数，只需要有所定义就行

- 函数参数：

  - Rust中定义函数如果需要参数，必须声明参数名称和类型：
    ```rust
    // Rust中定义函数如果需要参数，必须声明参数名称和类型
    fn another_function(x: i32, y: i32) {
        println!("x 的值为 : {}", x);
        println!("y 的值为 : {}", y);
    }
    ```

- 函数体的语句和表达式

  - Rust中可以在一个用 {} 包括的块里写一个较为复杂的表达式
    ```rust
    fn main() {
        let x = 5;
    
        let y = {
            let x = 3;
            x + 1
        };
    
        println!("x 的值为 : {}", x);
        println!("y 的值为 : {}", y);
    }
    ```

- 函数返回值

  - Rust 函数声明返回值类型的方式：在参数声明之后用 **->** 来声明函数返回值的类型（不是 **:** ）

  - 在函数体中，随时都可以以 return 关键字结束函数运行并返回一个类型合适的值。这也是最接近大多数开发者经验的做法
    ```rust
    fn add(a: i32, b: i32) -> i32 {
        return a + b;
    }
    ```

  - 但是 Rust 不支持自动返回值类型判断！如果没有明确声明函数返回值的类型，函数将被认为是"纯过程"，不允许产生返回值，return 后面不能有返回值表达式。这样做的目的是为了让公开的函数能够形成可见的公报。

  - **注意：**函数体表达式并不能等同于函数体，它不能使用 **return** **关键字。**

##### 1.4 Rust 条件语句与循环

##### 1.5 Rust 所有权

- Rust 语言为高效使用内存而设计的语法机制。
  - 所有权概念是为了让Rust在编译阶段更有效地分析内存资源的有用性以实现内存管理而诞生的概念
- 所有权规则：三条规则
  - 1）Rust中的每个值都有一个变量，称为其所有者
  - 2）一次只能有一个所有者
  - 3）当所有者不在程序运行范围时，该值将被删除
- 内存和分配
- 变量与数据交互的方式：
  - 移动：Move
  - 克隆：Clone
- 涉及函数的所有权机制
  - 变量的所有权机制
  - 函数返回值的所有权机制
- 引用与租借
  - 引用：Reference
    - 可变引用： &mut
    - 不可变引用：&
  - 垂悬引用：在Rust中是不被允许的

##### 1.5 Rust Slice 切片类型

- Slice：切片类型
  - &s[x..y]

##### 1.6 Rust 结构体

- Rust 中的结构体 Struct， 与元组 Tuple，都可以将若干个类型不一定相同的数据捆绑在一起形成整体。

  - 但结构体的每个成员和其本身都有一个名字，这样访问它成员的时候就不用记住下标了
  - 元组常用于非定义的多值传递，而结构体用于规范常用的数据结构。
  - 结构体中的每个成员叫做“字段”

- 结构体的定义：
  ```rust
  struct Site {
      domain: String,
      name: String,
      nation: String,
      found: u32
  }
  ```

- 结构体实例

- 元组结构体

- 结构体所有权

  - 输出结构体

- 结构体方法

- 结构体关联函数

- 单元结构体

##### 1.7 Rust 枚举类

- 普通枚举类
- match 语法
- Option 枚举类
- if let 语法

##### 1.8 Rust 组织管理

- 组织管理

  - 箱：Crate

    - "箱"是二进制程序文件或者库文件，存在于"包"中
    - "箱"是树状结构的，它的树根是编译器开始运行时编译的源文件所编译的程序

  - 包：Package

    - "箱"是树状结构的，它的树根是编译器开始运行时编译的源文件所编译的程序
    - 工程的实质就是一个包，包必须由一个 Cargo.toml 文件来管理，该文件描述了包的基本信息以及依赖项
    - 一个包最多包含一个库"箱"，可以包含任意数量的二进制"箱"，但是至少包含一个"箱"（不管是库还是二进制"箱"）

  - 模块：Module

    - 对于一个软件工程来说，我们往往按照所使用的编程语言的组织规范来进行组织，组织模块的主要结构往往是树
    - Java 组织功能模块的主要单位是类，而 JavaScript 组织模块的主要方式是 function
    - 这些先进的语言的组织单位可以层层包含，就像文件系统的目录结构一样
    - Rust 中的组织单位是模块（Module）

    ```rust
    mod nation {
        mod government {
            fn govern() {}
        }
        mod congress {
            fn legislate() {}
        }
        mod court {
            fn judicial() {}
        }
    }
    ```

  - PS:

    - 在文件系统中，目录结构往往以斜杠在路径字符串中表示对象的位置，Rust 中的路径分隔符是 **`::`**

    - 路径分为绝对路径和相对路径。绝对路径从 crate 关键字开始描述。相对路径从 self 或 super 关键字或一个标识符开始描述
      ```rust
      crate::nation::government::govern();
      
      nation::government::govern();
      ```

      

- 访问权限

  - Rust中两种简单的访问权：
    - 公共：public
      - pub 关键字
      - 
    - 私有：private
      - 对于私有的模块，只有在与其平级的位置或下级的位置才能访问，不能从其外部访问

- use 关键字

  - use 关键字能够将模块标识符引入当前作用域：
    ```rust
    mod nation {
        pub mod government {
            pub fn govern() {}
        }
    }
    
    use crate::nation::government::govern;
    
    fn main() {
        govern();
    }
    ```

  - 当然，有些情况下存在两个相同的名称，且同样需要导入，我们可以使用 as 关键字为标识符添加别名：
    ```rust
    mod nation {
        pub mod government {
            pub fn govern() {}
        }
        pub fn govern() {}
    }
       
    use crate::nation::government::govern;
    use crate::nation::govern as nation_govern;
    
    fn main() {
        nation_govern();
        govern();
    }
    
    // use关键字与pub关键字配合使用
    mod nation {
        pub mod government {
            pub fn govern() {}
        }
        pub use government::govern;
    }
    
    fn main() {
        nation::govern();
    }
    ```

- 引用标准库

  - Rust 官方标准库字典：https://doc.rust-lang.org/stable/std/all.html

    在学习了本章的概念之后，我们可以轻松的导入系统库来方便的开发程序了

##### 1.9 Rust 错误处理

- 错误处理

  - Rust 有一套独特的处理异常情况的机制，它并不像其它语言中的 try 机制那样简单
  - 对于可恢复错误用 Result<T, E> 类来处理，对于不可恢复错误使用 panic! 宏来处理

- 不可恢复错误

- 可恢复错误

  - 在 Rust 中通过 Result<T, E> 枚举类作返回值来进行异常表达

  - 在 Rust 标准库中可能产生异常的函数的返回值都是 Result 类型的

    ```rust
    enum Result<T, E> {
        Ok(T),
        Err(E),
    }
    ```

    e.g.
    ```rust
    use std::fs::File;
    
    fn main() {
        let f = File::open("hello.txt");
        match f {
            Ok(file) => {
                println!("File opened successfull.");
            },
            Err(err) => {
                println!("Failed to open the file.");
            }
        }
    }
    ```

  - 如果想使一个可恢复错误按不可恢复错误处理，Result 类提供了两个办法：unwrap() 和 expect(message: &str) 

  - 这段程序相当于在 Result 为 Err 时调用 panic! 宏。两者的区别在于 expect 能够向 panic! 宏发送一段指定的错误信息

- 可恢复错误的传递

- kind 方法获取错误类型，识别错误

##### 1.10 Rust 泛型与特性

- 泛型
  - 在函数中定义泛型
  - 结构体与枚举类中的泛型
- 特性：trait
  - 特性（trait）概念接近于 Java 中的接口（Interface），但两者不完全相同
  - 特性与接口相同的地方在于它们都是一种行为规范，可以用于标识哪些类有哪些方法。
  - Rust 同一个类可以实现多个特性，每个 impl 块只能实现一个
  - 默认特性：
    - 这是特性与接口的不同点：接口只能规范方法而不能定义方法，但特性可以定义方法作为默认方法
    - 因为是"默认"，所以对象既可以重新定义方法，也可以不重新定义方法使用默认的方法
  - 特性做参数
  - 特性做返回值
  - 有条件实现方法

##### 1.11 Rust 生命周期

- Rust 生命周期机制是与所有权机制同等重要的资源管理机制

- 之所以引入这个概念主要是应对复杂类型系统中资源管理的问题

- 垂悬引用在Rust中不允许

- 生命周期注释
  ```rust
  &i32        // 常规引用
  &'a i32     // 含有生命周期注释的引用
  &'a mut i32 // 可变型含有生命周期注释的引用
  ```

- 结构体中使用字符串切片引用

- 静态生命周期

- E.g 泛型、特性与生命周期协同作战
  ```rust
  use std::fmt::Display;
  
  fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
      where T: Display
  {
      println!("Announcement! {}", ann);
      if x.len() > y.len() {
          x
      } else {
          y
      }
  }
  ```

##### 1.12 Rust 文件与IO

- 封装
- 继承

##### 1.13 Rust 集合与字符串

- 向量 Vector
- 字符串 String
- 映射表 Map

##### 1.14 Rust 面向对象

- 

#### 2. Rust 基础入门

- 新的概念：
  - 所有权
  - 借用
  - 生命周期
  - 宏编程
  - 模式匹配
  - 智能指针

##### 2.1 变量绑定与解构

- 为什么要手动设置变量的可变性？
- 变量命名
  - Rust命名规范
  - 关键字
- 变量绑定
  - let a= "hello world"
  - 给这个过程起一个名字叫：变量绑定
  - 因为引入了所有权的概念
- 变量可变性
  - let mut x = 6;
  - x = 5;
- 使用下划线开头忽略未使用的变量
- *变量解构*
  - *解构式赋值*
  - 有点不是很懂解构的目的？
- 常量和变量之间的差异
  - const 与 let 变量的差异
    - 是否仅在于提高代码阅读性了？

- 变量遮蔽（shadowing）
  - Rust 允许声明相同的变量名，在后面声明的变量会遮蔽掉前面声明的

##### 2.2 基本类型

###### 2.2.1 数值类型

- 整数类型
  - 整型溢出
- 浮点数类型
  - 浮点数陷阱
  - NaN：Not a Number
- 数字运算
  - 加减乘除取模
- 位运算
- 序列：Range
  - i..j, e.g.:  1..5 代表 [1, 5) 之间的整数
  - 也可以用来生成字符类型
- 有理数和复数
  - 在第三方库： num 中
- 总结：
  - Rust 拥有相当多的数值类型
  - 类型转换必须是显示的
  - Rust的数值上可以使用方法

###### 2.2.2 字符、布尔、单元类型

- 字符类型：Char
  - 占用 4 个字节
  - Unicode值
- 布尔：bool
  - true/false
- 单元类型： Unit
  - `()`， 唯一的值也是()
  - fn main() 目前的返回值就是  ()
  - println! 宏的返回值也是 ()
  - 可以作为一个值用来占位，但是完全不占用任何内存

###### 2.2.3 语句和表达式

###### 2.2.4 函数

- 函数定义
- 函数要点
- 函数返回
  - Rust中的特殊返回类型
  - 无返回值()
- 永不返回的发散函数  `!`

##### 2.3 所有权和借用

###### 2.3.1 所有权

- 栈Stack 与 堆 Heap
  - 性能区别
  - 所有权与堆栈
- 所有权原则
  - 规则：
    - 1）Rust中每一个值都被一个变量所拥有，该变量被称为值得所有者
    - 2）一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
    - 3）当所有者离开作用域范围时，这个值将被丢弃Drop
  - 变量作用域
  - 简单介绍String类型
    - 为了应对字符串字面值的不足点，Rust提供了动态字符串类型 String
    - 该类型被分配到堆上，因此可以动态伸缩
- 变量绑定背后的数据交互
  - 转移所有权
    - 基本数据类型，存储在栈上，Rust自动拷贝
    - String 不是基本数据类型，存储在堆上，不能自动拷贝
  - 克隆：深拷贝
  - 拷贝：浅拷贝

###### 2.3.2 引用和借用

- 引用& 与解引用*
- 不可变引用
- 可变引用
  - 可变引用同时只能存在一个
  - 可变引用与不可变引用不能同时存在
  - NLL：Non-Lexical Lifetimes
- 悬垂引用：dangling references

##### 2.4 复合类型

###### 2.4.1 字符串与切片

- 字符串
- 切片Slice
  - 其他切片
- 字符串字面量是切片
- 什么是字符串？
- String 与 &str 的转换
- 字符串索引
  - 深入字符串内部
  - 字符串的不同表现形式
- 字符串切片
- 操作字符串
  - 追加Push
  - 插入Insert
  - 替换Replace
  - 删除Delete
  - 连接Concatenate
- 字符串转义
- 操作UTF-8字符串
  - 字符
  - 字节
  - 获取子串
- 字符串深度剖析
  - 在Rust中，变量在离开作用域后，就自动释放其占用的内存
  - 类似于其他系统编程语言的free函数，Rust提供了一个内存释放函数 drop
    - Rust在变量离开作用域后，自动调用 drop 函数

###### 2.4.2 元组

- 元组的定义

- 用模式匹配解构元组
- 元组的使用示例

###### 2.4.3 结构体

- 结构体语法
  - 定义结构体
  - 创建结构体实例
  - 访问结构体字段
  - 简化结构体创建
  - 结构体更新语法
- 结构体的内存排列
- 元组结构体 Tuple Struct
- 单元结构体 Unit-like Struct
- 结构体数据的所有权
- 使用 #[derive(Debug)] 来打印结构体的信息

###### 2.4.4 枚举

- 枚举：enum、enumeration

###### 2.4.5 数组



























































