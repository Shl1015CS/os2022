# rCore

## 本章节目的
- 通过提前加载应用程序到内存，减少应用程序切换开销
- 通过协作机制支持程序主动放弃处理器，提高系统执行效率
- 通过抢占机制支持程序被动放弃处理器，提高不同程序对处理器资源使用的公平性，也进一步提高了应用对 I/O 事件的响应效率

### 引言

- 协作式操作系统
- 抢占式操作系统

### 多道程序放置

- 在第二章中应用的加载和执行进度控制都交给 batch 子模块，而在第三章中我们将应用的加载这部分功能分离出来在 loader 子模块中实现，应用的执行和切换功能则交给 task 子模块。

```rs
 1 # user/build.py
 2
 3 import os
 4
 5 base_address = 0x80400000
 6 step = 0x20000
 7 linker = 'src/linker.ld'
 8
 9 app_id = 0
10 apps = os.listdir('src/bin')
11 apps.sort()
12 for app in apps:
13     app = app[:app.find('.')]
14     lines = []
15     lines_before = []
16     with open(linker, 'r') as f:
17         for line in f.readlines():
18             lines_before.append(line)
19             line = line.replace(hex(base_address), hex(base_address+step*app_id))
20             lines.append(line)
21     with open(linker, 'w+') as f:
22         f.writelines(lines)
23     os.system('cargo build --bin %s --release' % app)
24     print('[build.py] application %s start with address %s' %(app, hex(base_address+step*app_id)))
25     with open(linker, 'w+') as f:
26         f.writelines(lines_before)
27     app_id = app_id + 1
     
 ```
 
 它的思路很简单，在遍历 app 的大循环里面只做了这样几件事情：
 
 - 第 16~22 行，找到 src/linker.ld 中的 BASE_ADDRESS = 0x80400000; 这一行，并将后面的地址替换为和当前应用对应的一个地址；
 - 第 23 行，使用 cargo build 构建当前的应用，注意我们可以使用 --bin 参数来只构建某一个应用；
 - 第 25~26 行，将 src/linker.ld 还原。

### 多道程序加载

本章中，所有的应用在内核初始化的时候就一并被加载到内存中。为了避免覆盖，它们自然需要被加载到不同的物理地址。这是通过调用 loader 子模块的 load_apps 函数实现的：

```rs
 1 // os/src/loader.rs
 2
 3 pub fn load_apps() {
 4     extern "C" { fn _num_app(); }
 5     let num_app_ptr = _num_app as usize as *const usize;
 6     let num_app = get_num_app();
 7     let app_start = unsafe {
 8         core::slice::from_raw_parts(num_app_ptr.add(1), num_app + 1)
 9     };
10     // clear i-cache first
11     unsafe { asm!("fence.i" :::: "volatile"); }
12     // load apps
13     for i in 0..num_app {
14         let base_i = get_base_i(i);
15         // clear region
16         (base_i..base_i + APP_SIZE_LIMIT).for_each(|addr| unsafe {
17             (addr as *mut u8).write_volatile(0)
18         });
19         // load app from data section to memory
20         let src = unsafe {
21             core::slice::from_raw_parts(
22                 app_start[i] as *const u8,
23                 app_start[i + 1] - app_start[i]
24             )
25         };
26         let dst = unsafe {
27             core::slice::from_raw_parts_mut(base_i as *mut u8, src.len())
28         };
29         dst.copy_from_slice(src);
30     }
31 }
```
可以看出，第  个应用被加载到以物理地址 base_i 开头的一段物理内存上，而 base_i 的计算方式如下：

```rs
1 // os/src/loader.rs
2
3 fn get_base_i(app_id: usize) -> usize {
4     APP_BASE_ADDRESS + app_id * APP_SIZE_LIMIT
5 }
```
我们可以在 config 子模块中找到这两个常数。从这一章开始， config 子模块用来存放内核中所有的常数。看到 APP_BASE_ADDRESS 被设置为 0x80400000 ，而 APP_SIZE_LIMIT 和上一章一样被设置为 0x20000 ，也就是每个应用二进制镜像的大小限制。因此，应用的内存布局就很明朗了——就是从 APP_BASE_ADDRESS 开始依次为每个应用预留一段空间。这样，我们就说清楚了多个应用是如何被构建和加载的。


### 执行应用程序
 
 
## 任务上下文

![image](https://user-images.githubusercontent.com/91481507/178379676-92af73b7-2b83-419f-b00a-0934a36c08ea.png)

![image](https://user-images.githubusercontent.com/91481507/178379773-60a7af3a-6eaf-4682-a971-dfcf715edf74.png)

Trap 控制流在调用 __switch 之前就需要明确知道即将切换到哪一条目前正处于暂停状态的 Trap 控制流，因此 __switch 有两个参数，第一个参数代表它自己，第二个参数则代表即将切换到的那条 Trap 控制流。这里我们用上面提到过的 current_task_cx_ptr 和 next_task_cx_ptr 作为代表。在上图中我们假设某次 __switch 调用要从 Trap 控制流 A 切换到 B，一共可以分为四个阶段，在每个阶段中我们都给出了 A 和 B 内核栈上的内容。

- 阶段 [1]：在 Trap 控制流 A 调用 __switch 之前，A 的内核栈上只有 Trap 上下文和 Trap 处理函数的调用栈信息，而 B 是之前被切换出去的；
- 阶段 [2]：A 在 A 任务上下文空间在里面保存 CPU 当前的寄存器快照；
- 阶段 [3]：这一步极为关键，读取 next_task_cx_ptr 指向的 B 任务上下文，根据 B 任务上下文保存的内容来恢复 ra 寄存器、s0~s11 寄存器以及 sp 寄存器。只有这一步做完后， __switch 才能做到一个函数跨两条控制流执行，即 通过换栈也就实现了控制流的切换 。
- 阶段 [4]：上一步寄存器恢复完成后，可以看到通过恢复 sp 寄存器换到了任务 B 的内核栈上，进而实现了控制流的切换。这就是为什么 __switch 能做到一个函数跨两条控制流执行。此后，当 CPU 执行 ret 汇编伪指令完成 __switch 函数返回后，任务 B 可以从调用 __switch 的位置继续向下执行。

switch实现如下：

```rs
 1# os/src/task/switch.S
 2
 3.altmacro
 4.macro SAVE_SN n
 5    sd s\n, (\n+2)*8(a0)
 6.endm
 7.macro LOAD_SN n
 8    ld s\n, (\n+2)*8(a1)
 9.endm
10    .section .text
11    .globl __switch
12__switch:
13    # 阶段 [1]
14    # __switch(
15    #     current_task_cx_ptr: *mut TaskContext,
16    #     next_task_cx_ptr: *const TaskContext
17    # )
18    # 阶段 [2]
19    # save kernel stack of current task
20    sd sp, 8(a0)
21    # save ra & s0~s11 of current execution
22    sd ra, 0(a0)
23    .set n, 0
24    .rept 12
25        SAVE_SN %n
26        .set n, n + 1
27    .endr
28    # 阶段 [3]
29    # restore ra & s0~s11 of next execution
30    ld ra, 0(a1)
31    .set n, 0
32    .rept 12
33        LOAD_SN %n
34        .set n, n + 1
35    .endr
36    # restore kernel stack of next task
37    ld sp, 8(a1)
38    # 阶段 [4]
39    ret
```
我们手写汇编代码来实现 __switch 。在阶段 [1] 可以看到它的函数原型中的两个参数分别是当前 A 任务上下文指针 current_task_cx_ptr 和即将被切换到的 B 任务上下文指针 next_task_cx_ptr ，从 RISC-V 调用规范 可以知道它们分别通过寄存器 a0/a1 传入。阶段 [2] 体现在第 19~27 行，即根据 B 任务上下文保存的内容来恢复 ra 寄存器、s0~s11 寄存器以及 sp 寄存器。从中我们也能够看出 TaskContext 里面究竟包含哪些寄存器：

```rs
1// os/src/task/context.rs
2
3pub struct TaskContext {
4    ra: usize,
5    sp: usize,
6    s: [usize; 12],
7}
```
保存 ra 很重要，它记录了 __switch 函数返回之后应该跳转到哪里继续执行，从而在任务切换完成并 ret 之后能到正确的位置。对于一般的函数而言，Rust/C 编译器会在函数的起始位置自动生成代码来保存 s0~s11 这些被调用者保存的寄存器。但 __switch 是一个用汇编代码写的特殊函数，它不会被 Rust/C 编译器处理，所以我们需要在 __switch 中手动编写保存 s0~s11 的汇编代码。 不用保存其它寄存器是因为：其它寄存器中，属于调用者保存的寄存器是由编译器在高级语言编写的调用函数中自动生成的代码来完成保存的；还有一些寄存器属于临时寄存器，不需要保存和恢复。

我们会将这段汇编代码中的全局符号 __switch 解释为一个 Rust 函数：

```rs
 1// os/src/task/switch.rs
 2
 3global_asm!(include_str!("switch.S"));
 4
 5use super::TaskContext;
 6
 7extern "C" {
 8    pub fn __switch(
 9        current_task_cx_ptr: *mut TaskContext,
10        next_task_cx_ptr: *const TaskContext
11    );
12}
```


## 多道程序与协作式调度

上一节我们已经介绍了任务切换是如何实现的，最终我们将其封装为一个函数 __switch 。但是在实际使用的时候，我们需要知道何时调用该函数，以及如何确定传入函数的两个参数——分别代表正待换出和即将被换入的两条 Trap 控制流。本节我们就来介绍任务切换的第一种实际应用场景：多道程序，并设计实现三叠纪“始初龙”协作式操作系统

本节的一个重点是展示进一步增强的操作系统管理能力的和对处理器资源的相对高效利用。为此，对 任务 的概念进行进一步扩展和延伸：形成了

- 任务运行状态：任务从开始到结束执行过程中所处的不同运行状态：未初始化、准备执行、正在执行、已退出
- 任务控制块：管理程序的执行过程的任务上下文，控制程序的执行与暂停
- 任务相关系统调用：应用程序和操作系统直接的接口，用于程序主动暂停 sys_yield 和主动退出 sys_exit

### 多道程序背景与 yield 系统调用

![image](https://user-images.githubusercontent.com/91481507/178380161-6f18fc97-61a6-4ece-a728-9f2515ad45b7.png)

#### sys_yield 的缺点

请同学思考一下， sys_yield 存在哪些缺点？

当应用调用它主动交出 CPU 使用权之后，它下一次再被允许使用 CPU 的时间点与内核的调度策略与当前的总体应用执行情况有关，很有可能远远迟于该应用等待的事件（如外设处理完请求）达成的时间点。这就会造成该应用的响应延迟不稳定或者很长。比如，设想一下，敲击键盘之后隔了数分钟之后才能在屏幕上看到字符，这已经超出了人类所能忍受的范畴。但也请不要担心，我们后面会有更加优雅的解决方案

### 任务控制块与任务运行状态

在第二章批处理系统中我们只需知道目前执行到第几个应用就行了，因为在一段时间内，内核只管理一个应用，当它出错或退出之后内核会将其替换为另一个。然而，一旦引入了任务切换机制就没有那么简单了。在一段时间内，内核需要管理多个未完成的应用，而且我们不能对应用完成的顺序做任何假定，并不是先加入的应用就一定会先完成。这种情况下，我们必须在内核中对每个应用分别维护它的运行状态，目前有如下几种：

```rs
1// os/src/task/task.rs
2
3#[derive(Copy, Clone, PartialEq)]
4pub enum TaskStatus {
5    UnInit, // 未初始化
6    Ready, // 准备运行
7    Running, // 正在运行
8    Exited, // 已退出
9}
```

#### Rust Tips：#[derive]

通过 #[derive(...)] 可以让编译器为你的类型提供一些 Trait 的默认实现。

实现了 Clone Trait 之后就可以调用 clone 函数完成拷贝；

实现了 PartialEq Trait 之后就可以使用 == 运算符比较该类型的两个实例，从逻辑上说只有 两个相等的应用执行状态才会被判为相等，而事实上也确实如此。

Copy 是一个标记 Trait，决定该类型在按值传参/赋值的时候采用移动语义还是复制语义。


仅仅有这个是不够的，内核还需要保存一个应用的更多信息，我们将它们都保存在一个名为 任务控制块 (Task Control Block) 的数据结构中：
```rs
1// os/src/task/task.rs
2
3#[derive(Copy, Clone)]
4pub struct TaskControlBlock {
5    pub task_status: TaskStatus,
6    pub task_cx: TaskContext,
7}
```

可以看到我们还在 task_cx 字段中维护了上一小节中提到的任务上下文。任务控制块非常重要，它是内核管理应用的核心数据结构。在后面的章节我们还会不断向里面添加更多内容，从而实现内核对应用更全面的管理。

### 任务管理器

我们还需要一个全局的任务管理器来管理这些用任务控制块描述的应用

### 实现 sys_yield 和 sys_exit 系统调用

![image](https://user-images.githubusercontent.com/91481507/178380423-c1b497d9-0bca-42ba-a5d1-a37b4c702ef4.png)

### 分时多任务系统与抢占式调度

#### 时间片轮转调度

简单起见，本书中我们使用 时间片轮转算法 (RR, Round-Robin) 来对应用进行调度，只要对它进行少许拓展就能完全满足我们的需求。本章中我们仅需要最原始的 RR 算法，用文字描述的话就是维护一个任务队列，每次从队头取出一个应用执行一个时间片，然后把它丢到队尾，再继续从队头取出一个应用，以此类推直到所有的应用执行完毕

### RISC-V 架构中的中断

时间片轮转调度的核心机制就在于计时。操作系统的计时功能是依靠硬件提供的时钟中断来实现的。在介绍时钟中断之前，我们先简单介绍一下中断。

在 RISC-V 架构语境下， 中断 (Interrupt) 和我们第二章中介绍的异常（包括程序错误导致或执行 Trap 类指令如用于系统调用的 ecall ）一样都是一种 Trap ，但是它们被触发的原因却是不同的。对于某个处理器核而言， 异常与当前 CPU 的指令执行是 同步 (Synchronous) 的，异常被触发的原因一定能够追溯到某条指令的执行；而中断则 异步 (Asynchronous) 于当前正在进行的指令，也就是说中断来自于哪个外设以及中断如何触发完全与处理器正在执行的当前指令无关。

#### 从底层硬件的角度区分同步和异步

从底层硬件的角度可能更容易理解这里所提到的同步和异步。以一个处理器的五级流水线设计而言，里面含有取指、译码、算术、访存、寄存器等单元，都属于执行指令所需的硬件资源。那么假如某条指令的执行出现了问题，一定能被其中某个单元看到并反馈给流水线控制单元，从而它会在执行预定的下一条指令之前先进入异常处理流程。也就是说，异常在这些单元内部即可被发现并解决。

而对于中断，可以理解为发起中断的是一套与处理器执行指令无关的电路（从时钟中断来看就是简单的计数和比较器），这套电路仅通过一根导线接入处理器。当外设想要触发中断的时候则输入一个高电平或正边沿，处理器会在每执行完一条指令之后检查一下这根线，看情况决定是继续执行接下来的指令还是进入中断处理流程。也就是说，大多数情况下，指令执行的相关硬件单元和可能发起中断的电路是完全独立 并行 (Parallel) 运行的，它们中间只有一根导线相连。

### 时钟中断与计时器

由于软件（特别是操作系统）需要一种计时机制，RISC-V 架构要求处理器要有一个内置时钟，其频率一般低于 CPU 主频。此外，还有一个计数器用来统计处理器自上电以来经过了多少个内置时钟的时钟周期。在 RISC-V 64 架构上，该计数器保存在一个 64 位的 CSR mtime 中，我们无需担心它的溢出问题，在内核运行全程可以认为它是一直递增的。

另外一个 64 位的 CSR mtimecmp 的作用是：一旦计数器 mtime 的值超过了 mtimecmp，就会触发一次时钟中断。这使得我们可以方便的通过设置 mtimecmp 的值来决定下一次时钟中断何时触发。

可惜的是，它们都是 M 特权级的 CSR ，而我们的内核处在 S 特权级，是不被允许直接访问它们的。好在运行在 M 特权级的 SEE （这里是RustSBI）已经预留了相应的接口，我们可以调用它们来间接实现计时器的控制：

### 抢占式调度

## 练习

