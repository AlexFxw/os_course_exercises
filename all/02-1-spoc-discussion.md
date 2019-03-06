# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
- x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

  主引导记录，根据这个去读取加载程序。不直接读入操作系统内核映像是因为不同的操作系统采用的文件系统有所差异，BIOS如果直接读入，则需要针对种类繁多的文件系统修改，通用性差，因此将加载内核的工作留给加载程序。

- 比较UEFI和BIOS的区别。

  UEFI 是一种电脑系统贵个，定义了操作系统和系统固件之间的软件界面，规范了在所有平台上一致的操作系统启动服务，是BIOS的一种替代方案。其有以下优点：

  * 安全性强：UEFI从独立分区启动，将操作系统和系统启动文件隔离
  * 启动配置灵活
  * 支持容量大，BIOS因为主引导记录的限制，无法引导超过2TB以上硬盘，UEFI则没有

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

  2个字节的标识符55AA。

- x86中在UEFI中的可信启动有什么作用？

  验证引导记录的可信度，避免BIOS从不可信的来源启动操作系统造成损失。

- RV中BBL的启动过程大致包括哪些内容？

  > 参考[lowRISC](https://www.lowrisc.org/docs/untether-v0.2/bootload/)

  1. 从 SD 将 BBL (Berkeley bootloader) 加载到DDR Ram（在拷贝之前，将DRAM直接映射到I/O以避开Cache）。
  2. 软启动：将DDR Ram的BBL映射到boot memory。
  3. 启动BBL，BBL初始化所有的外设、设定页表以及虚拟内存，并从SD将Linux 内核加载到虚存里，最终启动内核。

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

  1. 中断：通常是来自外部设备的请求，CPU会先检查中断屏蔽字，如果接受中断，则操作系统会保留当前堆栈、程序计数器等信息并切换至内核态的中断处理程序，中断处理完成后返回现场。
  2. 异常：对指令执行意外的响应，例如除0计算。
  3. 系统调用：用户程序中需要使用硬件接口时不直接访问硬件，而是使用系统提供的调用接口。

- 中断、异常和系统调用的处理流程有什么异同？

  - 相同点：
    - 都会切换到内核态
  - 不同：
    - 响应方式不同，中断异步，异常同步，系统调用二者皆可。

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？

  在syscall.c文件里，可以发现以下代码：

  ```c
  static int (*syscalls[])(uint32_t arg[]) = {
      [SYS_exit]              sys_exit,
      [SYS_fork]              sys_fork,
      [SYS_wait]              sys_wait,
      [SYS_exec]              sys_exec,
      [SYS_yield]             sys_yield,
      [SYS_kill]              sys_kill,
      [SYS_getpid]            sys_getpid,
      [SYS_putc]              sys_putc,
      [SYS_pgdir]             sys_pgdir,
      [SYS_gettime]           sys_gettime,
      [SYS_lab6_set_priority] sys_lab6_set_priority,
      [SYS_sleep]             sys_sleep,
      [SYS_open]              sys_open,
      [SYS_close]             sys_close,
      [SYS_read]              sys_read,
      [SYS_write]             sys_write,
      [SYS_seek]              sys_seek,
      [SYS_fstat]             sys_fstat,
      [SYS_fsync]             sys_fsync,
      [SYS_getcwd]            sys_getcwd,
      [SYS_getdirentry]       sys_getdirentry,
      [SYS_dup]               sys_dup,
  };
  ```

  可以看出，uCore的系统调用按照功能大致分成以下几类：

  * 文件管理：open/close/read/write/seek/fstat/fsync/getcwd/getdirentry/dup
  * 进程管理：fork/exit/wait/exec/yield/kill/getpid/sleep
  * 内存管理：pgdir
  * 外设管理：putc

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。


## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？
  - 汇编指令不同
    - 系统调用使用INT、IRET指令
    - 函数调用使用CALL、RET指令
  - 系统调用会切换堆栈，函数调用不会。
  - 系统调用会切换特权级，函数调用不会。
  - 系统调用的切换会有额外的开销，函数调用时间开销小。
  - 函数调用如果是静态编译，则空间开销大于系统调用。
  - 系统调用较为安全。
- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
