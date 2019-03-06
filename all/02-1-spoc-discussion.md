# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习

 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。

 - > 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”

 - > 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。

 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？

 - > 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”

 - > 安装好ucore实验环境，能够编译运行lab8的answer

 - > 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```

 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。

 - > 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。

 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
- 请描述在“计算机组成原理课”上，同学们做的MIPS CPU是从按复位键开始到可以接收按键输入之间的启动过程。

  - > 按复位键之后 CPU 会将 kernel 从 ROM 读入至内存，并且从 0x0000 的地址开始执行代码。

- x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

  -  第一个扇区是 MBR，其中包含 446 字节的启动代码、64 字节的硬盘分区表以及 2 字节的结束标志字
  -  因为在 BIOS 启动硬盘的时候还没有建立文件系统，因此无法直接读入系统内核映像；同时一个磁盘可能有多个操作系统内核，BIOS 无法决定选择哪一个系统。

- 比较UEFI和BIOS的区别。

  -  统一可扩展固件接口 (Unified Extensible Firmware Interface, UEFI) 是一种个人电脑系统规格，用来定义操作系统与系统固件之间的软件界面，作为BIOS的替代方案。
  -  UEFI启动对比BIOS启动的优势有三点：
     1. 安全性更强：UEFI启动需要一个独立的分区，它将系统启动文件和操作系统本身隔离，可以更好的保护系统的启动；
     2. 启动配置更灵活：EFI启动和GRUB启动类似，在启动的时候可以调用EFIShell，在此可以加载指定硬件驱动，选择启动文件。比如默认启动失败，在EFIShell加载U盘上的启动文件继续启动系统；
     3. 支持容量更大：传统的BIOS启动由于MBR的限制，默认是无法引导超过2TB以上的硬盘的。随着硬盘价格的不断走低，2TB以上的硬盘会逐渐普及，因此UEFI启动也是今后主流的启动方式。

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

  - 0x55AA

- x86中在UEFI中的可信启动有什么作用？

  - 通过启动前的数字签名检查来保证启动介质的安全性。

- RV中BBL的启动过程大致包括哪些内容？

  - BBL runs after the soft reset as DDR is now mapped to the boot address `0x00000200`. The major function of the BBL is to initialize all peripherals, set up the page table and virtual memory, load the Linux kernel from SD to virtual memory, and finally boot the kernel.

    During the kernel execution, BBL continues running underneath the kernel as a hypervisor, serving all peripheral requests from Linux using the actual FPGA hardware.

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？
  - 中断、异常和系统调用均是由内核进行处理的特殊事件
  - 中断通常来自外设的请求，由内核负责中断的屏蔽、使能，并进行异步的响应
  - 异常为不合法或者错误的程序行为，常见的异常有除零错误、内存地址读写错误等等。异常出现之后由内核进行同步处理，杀死或者重新执行出错的程序
  - 系统调用是内核对一些底层、权限较高的不安全指令和操作进行封装并对应用提供服务的一种接口。系统调用类似于函数调用，只是调用的对象是内核，因此要进行用户态到内核态的转换。
- 中断、异常和系统调用的处理流程有什么异同？
  - 相同：都要从用户态切换到内核态
  - 不同：触发的源头、响应方式、处理机制
- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？
  - 进程管理：包括 fork/exit/wait/exec/yield/kill/getpid/sleep
  - 文件操作：包括 open/close/read/write/seek/fstat/fsync/getcwd/getdirentry/dup
  - 内存管理：pgdir命令
  - 外设输出：putc命令

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。


## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？
  - 汇编指令的区别
    - 系统调用：使用INT和IRET指令
    - 函数调用：使用CALL和RET指令
  - 安全性的区别
    - 系统调用有堆栈和特权级的转换过程，函数调用没有这样的过程，系统调用相对更为安全
  - 性能的区别
    - 时间角度：系统调用比函数调用要做更多和特权级切换的工作，所以需要更多的时间开销
    - 空间角度：在一些情况下，如果函数调用采用静态编译，往往需要大量的空间开销，此时系统调用更具有优势
- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
