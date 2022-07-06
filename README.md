# 2022 Daily schedule of OS Tranining Camp

- daily study：每日学习心得（笔记）
- rust-exercise-play 写的一些rust小程序
- my rustlings:rust的练习和代码

目前处于第一阶段，第一环节学习Rust语言


## 时间表

*七月*

| Mon               | Tues              | Wed                          | Thur                         | Fri                          | Sat               | Sun               |
| ----------------- | ----------------- | ---------------------------- | ---------------------------- | ---------------------------- | ----------------- | ----------------- |
|                              |                        |                           |                   | 1 <br> ([D1](#day-1-202271)) | 2 <br> ([D2](#day-2-202272)) | 3 <br> ([D3](#day-3-202273)) |
|4 <br> ([D4](#day-4-202274)) | 5 <br> ([D5](#day-5-202275)) | 6 <br> ([D6](#day-6-202276)) | 7 <br> ([D7](#day-7-202277)) | 8 <br> ([D8](#day-8-202278))       | 9 <br> ([D9](#day-9-202279))            | 10 <br> ([D10](#day-10-2022710))         | 
|11  <br>  ([D11](#day-11-2022711))             | 12      <br>    ([D12](#day-12-2022712))       | 13    <br>    ([D13](#day-13-2022713))             | 14         <br>    ([D14](#day-14-2020711))        | 15        <br>    ([D15](#day-15-2022715))                    | 16    <br>     ([D16](#day-16-2022716))                       | 17    <br>      ([D17](#day-17-2022717))                       |
|18    <br>    ([D18](#day-18-2020718))            | 19   <br>     ([D19](#day-19-2022719))            | 20   <br>    ([D20](#day-20-2022720))            | 21       <br>    ([D21](#day-21-2022721))         | 22     <br>    ([D22](#day-22-2022722))                         | 23     <br>    ([D23](#day-23-2022723))                         | 24    <br>    ([D24](#day-24-2022724))                        | 
|25      <br>    ([D25](#day-25-2022725))             | 26         <br>    ([D26](#day-26-2022726))           | 27         <br>    ([D27](#day-27-2022727))           | 28       <br>    ([D28](#day-28-2022728))           | 29         <br>    ([D29](#day-29-2022729))                    | 30        <br>    ([D30](#day-30-2022730))                     | 31     <br>    ([D31](#day-31-2022731))                           |

------

## Day 1 2022/7/1

### OS Tranining Camp

前几天投的简历在今天早上就收到了回复。之前了解过rust语言的核心特点是安全性，即是指Safety,而非Security。本身对rCore实验就是比较感兴趣的，能将rust语言应用到操作系统的编写当然是一件要去参与的事情。

看了课程安排，需要学习的有RISC-V架构，Rust语言，操作系统课程，这对于我来说无疑是极好的，虽说本人是本科学材料学的，但是还是对计算机充满了热爱。

### Study Rust

今天就开始学习了Rust，看了[Rust 程序设计语言](https://kaisery.github.io/trpl-zh-cn/)，（因为并不能看懂英文书，当然这在之后的时间里要加强对英语的学习与掌握）

通过观看推荐网课：

- [Rust编程视频教程（基础）](https://www.bilibili.com/video/BV1xJ411B79h?p=1&vd_source=1247df552269232c9b6af3ff5c9f0868)

- [Rust编程视频教程（进阶）](https://www.bilibili.com/video/BV1FJ411Y71o?spm_id_from=333.999.0.0)

- 具体内容参考今日笔记文档[Day1_rust](daily_study/Day1_rust.md)

## Day 2 2022/7/2

### OS Learning

今天学习了操作系统的基本概念，大概了解了操作系统的需求和我们为什么要学习操作系统，后半段学习了操作系统的历史。十分风趣幽默，知识也是易于接受，了解到了我们要学习的东西，即kernel。
操作系统的核心特征就是并发性，这也是我们需要首要实现和解决的点，为后续实现功能打好基础，之后动手搭建环境，用的是github classroom，完成简单的程序员简单操作hello,world!。

### Rust Learning

继续了rust基础阶段的学习，认识了

- 所有权(引用借用，slice类型)
- 结构体相关知识
- 枚举和模式匹配
- 使用包
- 集合
- 错误处理
- 泛型、trait 和生命周期

- 具体内容参考今日笔记文档[Day2](daily_study/day2_os-learning.md)

编程练习题 ： 

[Small exercises to get you used to reading and writing Rust code!](os2022/my-rustlings/README.md)

完成了：

- variables
- if
- function
- primitive_types
- structs
- strings
- enums
- tests
- modules
- macros
- move_semantics

和附带的quiz

## Day 3 2022/7/3

### 阅读rCore 指导书

阅读了今年适用于2022训练营的新指导书[rCore-Tutorial-Book 第三版](https://learningos.github.io/rust-based-os-comp2022/)

完成了lab0-test1，环境搭建，通过观看往届第一个实验是去除宏指令完成裸机运行输出Hello,World!看来今年只能通过观看代码解决了！
### Rust Learning

继续了学习Rust,学习的内容如下：
- 生命周期
- 函数式编程: 闭包、迭代器
- 深入类型
- 智能指针
- 循环引用与自引用
- 错误处理

做完了所有[rustling-exercise](os2022/my-rustlings/README.md),发现越往后做越难，一边hint，一边看圣经，一边debug，一边做。学了约等于没学，做起题来还是得从头看，尤其是错速处理部分。

### OS Learning

观看了第二次课程，获益匪浅

-参考第三天总结[day3](os2022/daily_study/day3-learning.md)

训练营强度还是很大的

## Day 4 2022/7/4

### Rust

虽说题目做的差不多完了，但是学习Rust的脚步还没有停止，学习Rust的数据结构。Rust最特殊的一个地方(所有权，无需free，很安全)，Rust圣经里面关于这个的讲解太到位了!还有就是智能指针
这都是要加深理解的地方。

### OS Learning

完成了第三次课的学习，在第二次课讲解实验环境和shell还有不同模式下不同功能下的OS，了解了这门课的学习目的，和要实现的东西。第三课要点在slide上面都有体现,主要为CSR寄存器和状态转换

### 阅读rCore 指导书

开始尝试去阅读第二章ch2的批处理系统相关内容，尝试去读懂代码了解内核精髓！并思考课后题

-参考第四天总结[day4](os2022/daily_study/day4-learning.md)

## Day 5 2022/7/5

### Rust

继续学习Rust语法，并尝试做第二部分习题，开始看源码

### OS learning

开始学习第四节课，开始了解任务（进程）相关知识

### Risc-v

继续学习计算机组成软硬件接口第二章知识


### 晚上开会

开幕式顺利召开

-参考第三天总结[day5](os2022/daily_study/day5-learning.md)
