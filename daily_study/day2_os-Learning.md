# 操作系统
 
 参考书籍:[Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/)
 
## Three Easy Pieces

- 虚拟化
- 并发性
- 持久性

操作系统并没有明确的定义，从广泛的定义来说，微信也可以作为一个操作系统

## 操作系统结构

- 中断及系统调用 ，异常控制流
- 内存管理 ， 地址空间
- 处理机的调度 ， CPU的并行
- 同步互斥
- 文件系统
- I/O 子系统

解决的核心问题是 How to make  the program easy?

因为操作系统是为系统应用和客户应用服务的，需要去coding和running，OS kernel 运行在内核态，特权级下的软件和系统应用

## 操作系统中软件的分类

- shell
- GUI(图形用户接口)
- kernel(操作系统的内部)，也是我们这门课研究的核心

## 操作系统内核的抽象

我们可以抽象出: 进程 文件 地址空间 CPU 磁盘 内存。

在这里我们需要把CPU划分成不同时间段来共享使用，从管理的角度，我们才会把CPU抽象出一个概念。磁盘到内存，内存到磁盘。在这期间我们要去建立一个文件，去完成操作系统的功能

## 操作系统内核的特征

- 并发: 同时存在过河运行程序
- 共享: 互斥共享各种资源
- 虚拟: 建立一个假定虚拟环境，让program认为“独占”一个程序
- 异步: 这个跟之后提到的异步可能不一样，服务时间的不确定性，这个是因为大量的打断导致的随机性

我们的开发需求，高效和易用，强大的操作系统服务和简单的接口。

## 操作系统内核的历史

课件中有

# Rust study

## 函数

不传参，无返回值

```rs
fn other_fun()
{
  println!("This is a fuction");
 }
```

传参，无返回值

```rs
fn other_fun1(a:i32 , b : u32)
{
println!("a={} , b = {}",a,b);
}
```

函数得传参类型可以推导么？答案是不可以！

传参，有返回值

```rs
fn other_fun2(a:i32 , b:u32 ) -> i32 
{
let result = a + b;
return result;
//还有一种写法为:result
//末尾无分号
}
```

表达式会计算一些值，语句只是执行一些操作，但是不返回值得指令。

## 控制流

### if

if-else语句

```rs
fn main()
{
 let y=1;
 if y == 1 {
 println!("y = 1")
 } else {
      println!("y != 1");
      }
}
```

if-else if -else if也可以嵌套，这里面和C++一样

### loop

### while

### for

## 所有权

通过所有权机制来管理内存，编译器在编译就会根据所有权规则对内存的使用进行检查

### 堆和栈


### 作用域

释放之后就会无效

### clone

 > let s2 = s1.clone()

copy trait
常用类型有：整型，浮点型，布尔，字符型，元组
 
 ```rs
 //copy
 let a=1;
 let b=a;
 //没有报错
 //栈上的数据是进行拷贝的
 ```
 
