# 初识 LLDB 

------

## 准备工作

关闭系统 SIP 才能愉快的玩耍，在 `iTerm` 激活 `lldb` 以及 `tty`

首先是通过 `file /Applications/Xcode.app/Content/MasOS/Xcode` 命令加载了一个可执行文件：

```shell
/Applications/Xcode.app/Contents/MacOS/Xcode: Mach-O 64-bit executable x86_64
```

接着是 `process launch ` 加载进程，添加 `-e <tty_path>` 参数作为 Xcode error 的输出 console

> Xcode 的 log 是以 stderr 的方式输出的

此时我们的 `iTerm` 有两个 tab，一个是 `lldb` 另一个是 `xcode stderr`，新建一个 Swift 工程开始玩耍

## 一个 LLDB 例子

`Control+C` 中断 Xcode，然后输入 `br -[NSView hitTest:]` 添加断点，然后输入 continue，此时点击 Xcode 会发现 `lldb` 有 log 打出，因为 `AppKit` 响应了 `-[NSView hitTest:]` 这个方法，触发了断点：

```shell
Process 76353 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 4.1
    frame #0: 0x00007fff3007a115 AppKit`-[NSView hitTest:]
AppKit`-[NSView hitTest:]:
->  0x7fff3007a115 <+0>: pushq  %rbp
    0x7fff3007a116 <+1>: movq   %rsp, %rbp
    0x7fff3007a119 <+4>: pushq  %r15
    0x7fff3007a11b <+6>: pushq  %r14
Target 0: (Xcode) stopped.
```

输入 continue 发现又再次打断，因为 `-[NSView hitTest:]` 的调用层级非常多

### 增加参数

Xcode 10 代码渲染部分是通过 `IDESourceEditorView` 这一私有类负责的，我们可以给断点添加一个条件以此来过滤输出：`breakpoint modify -c '(BOOL)[NSStringFromClass((id)[$rdi class]) containsString:@"IDESourceEditorView"]' -G0`  

> - modify 是 `breakpoint` 的一个 subcommend，这里表示对所有断点进行修改
>
> - -c 代表条件，Xcode 中的 condition 选项
>
> - `$rdi` 是 x64 汇编下调用函数的第一个参数，这里即 `objc_msgSend(receiver, @selector(hitTest:))` 中的 receiver，对应了当前的 `IDESourceEditorView` 实例

点击代码框，`lldb` 打印如下：

```shell
(lldb)  po $rdi
IDESourceEditorView: Frame: (0.0, 0.0, 868.0, 488.0), Bounds: (0.0, 0.0, 868.0, 488.0) contentViewOffset: 0.0
```

这里的表示 Xcode 中负责渲染代码的那个 `NSView`

### 指针

使用 `po` 命令时，`lldb` 会检查当前的内容是否为一个合法的 `NSObject` 或者是 Swift 对象，如果合法会自动调用 `[object description]` 方法。想打印指针地址可以用 `p/x ` 命令以 hex 的形式打印出当前的指针地址：

```shell
(lldb) (unsigned long) $96 = 0x0000000111094000
```

此时我们可以记录下 `$96` 所存储的指针地址 `0x111094000`，后续即使 `rdi` 寄存器的值被修改， `0x111094000` 只要没有被释放便依旧可以使用。

通过 `e [$rid setHidden:YES];[CATransaction flush]` 我们可以把 Xcode 玩坏 👨🏻‍💻	

> e 表示 expression， `lldb` 允许我们在开发阶段调用任何 API，理论上可以用 `lldb` 写完整个 App ...

### 调试语言

当我们直接在 Xcode 中断程序，或者工程语言为 Objective-C 的时候，`lldb` 默认使用的是 Objective-C 作为输入语言

> 如果是 Swift 工程，通过 Xcode 添加的断点被触发后， `lldb` 使用的是 Swift 作为调试语言；但如果直接通过 Xcode 中断程序，`lldb` 会默认使用 Objective-C 

假如我们需要强制切换调试语言，我们通过 `expression` 的 `-l` 参数：

```shell
(lldb) ex -l swift -- import Foundation
(lldb) ex -l swift -- import AppKit
```

对于要使用的 API，我们需要跟开发阶段一样，先 import 对应的 module，下一步我们可以开始调用了。

由于 Swift 淡化了指针，我们需要用 `unsafeBitCast` 把指针转化为一个合法的 Swift 对象：

```shell
(lldb) ex -l swift -o -- unsafeBitCast(0x111094000, to: NSView.self).insertText("Hack")
```

除了通过 `expression -l` 来指定语言，我们还可以使用 `Swift REPL` 来输入调试语言：

```shell
(lldb) ex -r -- 
1> let object = unsafeBitCast(0x10100f1d0, to:UIViewController.self)
object: Hello_Debuger.ViewController = {
  UIKit.UIViewController = {
    UIKit.UIResponder = {
      baseNSObject@0 = {
        isa =
      }
    }
  }
}
2> object.view
$R22: UIView? = 0x00000001010110a0 {
  baseUIResponder@0 = {
    NSObject = {
      isa =
    }
  }
}
```

但是 `Swift REPL` 的使用体验非常糟糕 ... 所以建议还是用 Objective-C 吧 🤪。使用 `expression` 命令要求默认一行输入全部内容，如果需要换行可以输入空格回车：

```shell
(lldb) e -o --
Enter expressions, then terminate with an empty line to evaluate:
1 func a(){
2 	print("haha")
3 }
4  
5 a()
6 
haha
```

## Help & Apropos 命令

一开始使用 `lldb` 时，对于去哪里查文档一脸懵逼，`lldb.llvm.org` 上并没有很完整的文档，其实完整的文档藏在 `lldb` 里面 ... 

### help 命令

在 `lldb` 中输入 `help`：

```shell
(lldb) help
Debugger commands:
  apropos           -- List debugger commands related to a word or subject.
  breakpoint        -- Commands for operating on breakpoints (see 'help b' for shorthand.)
  bugreport         -- Commands for creating domain-specific bug reports.
  command           -- Commands for managing custom LLDB commands.
  disassemble       -- Disassemble specified instructions in the current target.  Defaults to the current function for the current thread and stack frame.
  expression        -- Evaluate an expression on the current thread.  Displays any returned value with LLDB's default formatting.
  frame             -- Commands for selecting and examing the current thread's stack frames.
  gdb-remote        -- Connect to a process via remote GDB server.  If no host is specifed, localhost is assumed.
  gui               -- Switch into the curses based GUI mode.
  help              -- Show a list of all debugger commands, or give details about a specific command.
  kdp-remote        -- Connect to a process via remote KDP server.  If no UDP port is specified, port 41139 is assumed.
  language          -- Commands specific to a source language.
  log               -- Commands controlling LLDB internal logging.
  memory            -- Commands for operating on memory in the current target process.
  platform          -- Commands to manage and create platforms.
  plugin            -- Commands for managing LLDB plugins.
  .
  .
  .
  
```

想针对某个命令比如 `breakpoint` 可以通过 `help breakpoint` 进一步查看：

```
(lldb) help breakpoint
     Commands for operating on breakpoints (see 'help b' for shorthand.)

Syntax: breakpoint <subcommand> [<command-options>]
```

从上面我们可以看到 `breakpoint` 还包含 `<subcommand>`，我们可以通过 `help breakpoint set` 这样的方式获得更近一步的信息。

### apropos 命令

在 `lldb` 中输入 `apropos "<key_word>"` 可以搜索相关命令，但目前 Xcode 10 的 `lldb` 有坑 ... 无法给出完整路径

## 调试进程

`lldb` 可以介入一个进程从而进行调试，这实际是通过一个叫 `debugserver` 的东西（位于 Xcode.app/Contents/SharedFrameworks/LLDB.framework/Resources/ 之下）来实现的，如果是一个 remote 进程（比如 iOS）， `debugserver` 会在目标设备运行

### 介入运行进程

假如我们想介入一个进程，通常有以下方式：

* 通过名称介入一个正在运行的进程

  ```shell
  ➜  ~ lldb -n "Xcode"
  (lldb) process attach --name "Xcode"
  ```

* 通过 pid 介入一个正在运行的进程

  ```shell
  ➜  ~ pgrep -x "Xcode"	
  50362
  ➜  ~ lldb -p 50362
  (lldb) process attach --pid 50362
  ```

> 可以通过 `pgrep -x <name>` 来查询对应的 `pid`

* 通过名称介入一个 **即将** 运行的进程

  ```shell
  ➜  ~ lldb -x "Xcode" -w
  ```

### 介入未运行线程

我们还可以通过先加载可执行文件，然后执行进程的方式介入，比如我们想通过 `lldb` 介入 `ls` 程序 

1. 加载可执行文件：

   ```shell
   ➜  ~ file /bin/ls
   /bin/ls: Mach-O 64-bit executable x86_64
   
   ➜  ~ lldb -f /bin/ls
   (lldb) target create "/bin/ls"
   Current executable set to '/bin/ls' (x86_64).
   ```

2. 加载进程：

   ```shell
   (lldb) process launch
   Process 66165 launched: '/bin/ls' (x86_64)
   ```

我们可以在加载进程时使用一些参数：

* 指定程序当前的工作路径 ( working-dir )：

  ```shell
  (lldb) process launch -w "/Application"
  ```

* 传递参数到当前程序：

  ```shell
  (lldb) process launch -- "~/desktop"
  Process 66842 launched: '/bin/ls' (x86_64)
  ls: ~/desktop: No such file or directory
  ```

  出错了，因为 `lldb` 默认是不解读 `~` 缩略符的，我们可以使用 `-X true` 这一参数修复下：

  ```shell
  (lldb) process launch -X true -- "~/desktop"
  Process 68241 launched: '/bin/ls' (x86_64)
  desktop.txt
  ```

  `process launch -X true` 这个命令等价于 `run`：

  ```shell
  (lldb) help run
  'run' is an abbreviation for 'process launch -X true --'
  ```

### stdin, stdout & stderr

`lldb` 默认采用了 `stdin`( standard input ) 的方式进行输入，我们可以通过 `-i` 改变输入源：

```shell
➜  ~ echo "/Applications/" > path.txt
➜  ~ lldb -f /bin/ls
(lldb) target create "/bin/ls"
Current executable set to '/bin/ls' (x86_64).

(lldb) process launch -i /users/dctang/path.txt
Process 70741 launched: '/bin/ls' (x86_64)
Applications	Downloads	Music		bin
Desktop		Library		Pictures	path.txt
Documents	Movies		Public
Process 70741 exited with status = 0 (0x00000000)
```

这对于我们平时在使用 Xcode 的时候，如果想要输入一堆东西时，非常有帮助。

除了修改输入源，我们还可以：

* 修改输出：

  ```shell
  (lldb) process launch -o /users/dctang/desktop/result.txt -- /users/dctang/desktop
  Process 75215 launched: '/bin/ls' (x86_64)
  Process 75215 exited with status = 0 (0x00000000)
  ```

* 重定向 stderr 输出：

  ```shell
  (lldb) process launch -e /dev/ttys001
  ```

## 断点

