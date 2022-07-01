# rust 入门

参考资料：[Rust 程序设计语言](https://kaisery.github.io/trpl-zh-cn/)

## 在虚拟机上面安装Rust

我才用的环境是虚拟机搭载的Ubuntu20.04
运行：

curl https://sh.rustup.rs -sSf | sh

source ~/.cargo/env
> Welcome to Rust!

rustc --version
>rustc 1.64.0-nightly (ddcbba036 2022-06-29)

cargo --version
>cargo 1.64.0-nightly (dbff32b27 2022-06-24)
环境安装完毕！
 
 ## Hello, World!
 
 $ mkdir study_rust
 $ cd study_rust
 
 ### 建立工程
 $ cargo new hello
 
 $ cd hello
 会发现一个配置文件cargo.toml,而代码在src内main.rs中
 ```rs
fn main() {
    println!("Hello, world!");
}
```
使用cargo run运行
> Hello,world!
这里的println!指令是一个打印的宏（macro），而非函数
- 编译 cargo bulid
- 语法检查是cargo build

## 变量类型
 
$ mkdir learn_variable
 
### 变量的定义
 
$ vim src/main.rs
变量定义的格式 let name:type=X;
let可以自适应判断变量类型，所以也可以写成let name = X;
type有 i32,i64,u8,u16,u32,u64和f32,f64,bool在rust中char是规定64位这个和C/C++是不一样的。
```rs
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```
这里会有报错cannot assign twice to immutable variable
如果想assign twice 可以对开始的一个语句进行替换 let mut x=5;，当没有加入mut关键字的时候是不可变变量

### 隐藏性
这个在C语言orC++里面是不一样的,(但是这样安全么?)
 ```rs
 fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    let x =1;
    println!("The value of x is: {}", x);
}
```
后面的变量会把前面的隐藏掉

### 常量

$ const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;

常量在整个程序生命周期中都有效，此属性使得常量可以作为多处代码使用的全局范围的值，例如一个游戏中所有玩家可以获取的最高分或者光速。

### 复合类型

元组

元组长度固定：一旦声明，其长度不会增大或缩小。

```rs
 fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

```rs
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}
```
> The value of y is: 6.4

这叫做解构（destructuring）,因为它将一个元组拆成了三个部分。最后,程序打印出了 y 的值，也就是 6.4。,我们也可以使用点号（.）后跟值的索引来直接访问它们
```rs
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```
数组

数组中的每个元素的类型必须相同。Rust 中的数组与一些其他语言(C/C++)中的数组不同，Rust中的数组长度是固定的。也就是说长度也是它的一大特性
格式如下：
```rs
let a = [1, 2, 3, 4, 5];
let a: [i32; 5] = [1, 2, 3, 4, 5];
```
还可以通过在方括号中指定初始值加分号再加元素个数的方式来创建一个每个元素都为相同值的数组：
```rs
let a = [3; 5];
```
这种写法与 let a = [3, 3, 3, 3, 3]; 效果相同，但更简洁。

### 小程序

guessing_game,文件在rust-exercise中
