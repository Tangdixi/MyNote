## Tour of Computer system

### bits + context
看看下面这段 `hello.c`

```

#include <stdio.h>
int main() {
	printf("Hello World");
}


```
在操作系统中, 一个文件是用一连串 `bit` 来表示的, 每个 `bit` 包含 0 或 1. 而当这些文件处于不同的 `context` 时, 所代表的东西也不同, 它可以表示一个 `integer` 或者是一个 `float` 等.  

--
### 程序的形式转换

当操作系统要开始运行 `hello.c` 时, 它必须被转换成一连串**底层的机器语言指令**(`low-level machine language instructions`), 最后会被打包成**可执行目标程序**, 以二进制的形式存储在磁盘中.

```Bash

user> gcc -o hello hello.c  


```

`hello.c` 将经历 **预处理** **编译** **汇编** **链接** 4个步骤后, 才能成为一个**可执行目标文件**.
> - **预处理 ( *Preprocessing* ) :** 预处理器根据头文件 `#include <stdio>` 来对原始文件进行调整, 生成 `hello.i`.
> - **编译 ( *Compilation* ) :** 编译器将 `hello.i`的文本转换为**汇编语言**文本 `hello.s`.
> - **汇编 ( *Assembly* ) :** 汇编器将 `hello.s` 的文本转换为**机器语言指令**, 并打包成**定位目标程序** `hello.o`, 这个时候假如你使用文本阅读器来打开 `hello.o` 的话, 你会看到一堆乱码. 
> - **链接 ( *Linker* ) :** `hello.c` 中包含 `printf()` 函数, 它由**标准C库**生成一个 `printf.o`, 链接负责将两个文件合并, 最终生成一个**可执行目标文件** `hello` .

![image](https://raw.githubusercontent.com/Tangdixi/MyNote/master/Operation%20System/Tour%20of%20computer%20system-1.png)

--
### 执行程序
我们可以通过下面这段代码, 在 ***Terminal*** 中运行 `hello`  

```Bash

user> ./hello
Hello World


```
想要了解操作系统是如何执行 `hello` 的, 我们需要了解一个系统的硬件组成, 看图:

![image](https://raw.githubusercontent.com/Tangdixi/MyNote/master/Operation%20System/Tour%20of%20computer%20system-2.png)

很复杂对吧 😂, 我们一个一个来.
> - **总线 ( *Buses* ) :** 总线为不同组件传递信息提供了一条电子管道, 在总线中传递的一般是固定直接大小的信息, 称为**字 ( *word* )**, 根据实际机器的不同, 一个**字**的大小可以是 **32-bits** 或 **64-bits** 
> - **主存 ( *Main Memory* ) :** 主存在处理器执行程序时, 提供一个暂时存储程序及数据的地方, 一般是 **DRAM**.
> - **处理器 ( *CPU* ) :** 处理器负责执行存储在主存里面的指令, 它内部核心是一个被称为 **PC ( *program counter* )** 的存储设备, 它的大小是一个**字**, 无论何时, PC 总是指向主存中的那些机器语言指令.

首先, 我们通过键盘输入 `./hello`, 这条语句会被读到 CPU 的寄存器中, 接着存储到主存.  
接着, 我们按下回车键, 通过 **DMA ( *direct memory access* )** 无需经过 CPU 寄存器, 直接从磁盘中寻找并将 `hello` 复制到主存里面.  
最后, 主存中的数据跟指令被读到 CPU 寄存器并开始执行, 得到运算结果后, 传输到显示器上. 

--
### 缓存
看完上面, 系统执行程序的这个流程, 我们发现大多数时间都是消耗在信息传递上, 从磁盘到主存, 从主存到CPU, 这一系列的操作会非常耗时的, 也决定了一个程序的执行时间.

根据物理定律, 大容量的储存设备的读取速度往往比小容量设备慢 (例如 CPU 从主存中读取数据快于从磁盘中读取). 这个时候, CPU 缓存的出现很好的解决了这个问题.

![image](https://raw.githubusercontent.com/Tangdixi/MyNote/master/Operation%20System/Tour%20of%20computer%20system-3.png)

> CPU 缓存位于主存跟 CPU 寄存器之间, 更强大的 CPU 拥有更多的缓存 (L1, L2, L3), 缓存均为 **SRAM**, 后一级缓存的读取速度慢于前一级. 当 CPU 需要请求内存数据时, 先在缓存中寻找, 若命中, 则可直接使用.

--
### 硬件

操作系统主要做了两件事:
> - 保证程序失控时, 硬件不会滥用.
> - 通过简单一致的机制来操控负责的底层硬件设备.

这里引出操作系统的几个抽象概念:
> - **进程 ( *Process* ) :** 当一个程序在运行时, 系统会制造只有一个程序 (如 `hello.c`)在运行的假象. 一个操作系统可以并发的运行多条进程, 这里单核 CPU 跟多核 CPU 有差别:
> 	- **单核 *CPU* :** 单核 CPU 通过内核切换进程来达到并发的效果, 这个机制称为 **context switch**.
>  - **多核 *CPU* :** 多核 CPU 可以通过不同核心达到并发的效果.
> - **线程 ( *Thread* ) :** 一条进程由多条线程组成, 线程位于进程的 context 上, 线程在数据共享及运行速度方面比线程更有优势.

![image](https://raw.githubusercontent.com/Tangdixi/MyNote/master/Operation%20System/Tour%20of%20computer%20system-4.png)

> - **虚拟内存 ( *Virtual Memory* ) :** 当一个程序在运行时, 系统会制造一种每个进程都在独立使用主存的假象, 这块放以后详细介绍.
> - **文件 ( *File* ) :** 文件就是字节序列, 就跟前面那个 `hello.c` 一样, 会被转化为字节序列, 存储到磁盘中.

--

### Concurrency vs Parellelism
在操作系统的发展史上, 有两个主要驱动力, 一个是让计算机做的更多, 一个是让计算机做得更快. 
> - **并发 ( *Concurrency* ) :** 并发指的是一个能同时具有多个任务的系统.
> - **并行 ( *Parellelism* ) :** 并行指的是通过并发, 让系统运行的更快.

--

### 小结
一个计算机操作系统是由硬件跟软件组合而成, 他们相互合作, 运行程序.
