#### four topics
1. isolation
2. 内核模式 用户模式
3. 系统调用
4. xv6

#### mix
- 怎样实现接口
- xargs
- util实验
  - 管道、并发
  - pipes of primes、primes exercises、primes problem
  - 标准输入、标准输出、标准错误
  - pipe and fork   fork之后文件描述符会被复制

isolation
- 应用程序和操作系统之间的隔离性
- one cpu  无论应用程序在做什么  他必须强制每隔一段时间放弃cpu  让其他程序运行
- strawman设计
- 内存隔离
- 操作系统一个很重要的作用是在复用的同时，有强内存隔离性
- 接口抽象硬件资源  让强隔离成为可能
- 操作系统提供进程作为cpu的抽象
  - CPU时间复用在不同的应用进程
  - 多个进程不能同时使用同一个CPU
- exec提供一个内存抽象
  - exec系统调用展示了不能直接访问内存 there is no direct access to memory
  - 操作系统提供内存隔离  控制了应用程序和物理硬件之间的交互
- 文件是对磁盘的抽象
  - 在Unix系统中，访问存储系统的唯一方法就是文件
  - 操作系统控制着磁盘如何映射到文件   一个磁盘块只会映射到一个文件 用户只能读自己的文件
- cache affinity  缓存亲和性  避免缓存缺失(cache misses) 高性能网络
- proc.c  复用

#### os shoule be defensive
- app cannot crash tho os
- app cannot break isolation
- strong isolation between the apps and os
- 一般由硬件支持强隔离  typical：HW support
  - 内核模式 user / kenel mode
  - 页表，虚拟内存 virtual memory

#### user / kenel mode
- 内核模式  cpu可以执行特权指令privileged instructions
  - set up a page table register配置页表寄存器
  - set the disabling clock interrupts 设置时钟中断
- unprivileged instructions
  - add
  - sub
  - branch

#### Q
如何知道是什么模式
- 处理器里有一个标志位 用户模式使用1 内核模式使用0
- 特权指令可以设置那一位   
- 那一位是受保护的

BIOS是在操作系统之前还是之后
- BIOS是一段可信任的代码  在操作系统之前运行

#### CPU provides virtual memory
- page table 页表  将虚拟地址映射到物理地址  
- process Hs own page table
- memory isolation  每个进程都有自己的内存空间
- 操作系统位于内核模式

#### entering kernel
- ecall(n)  n是应用程序想要访问的系统调用编号
  - 进入内核中的一个由内核控制的特定位置
  - 很多程序都是通过ecall进行设计内核的操作
  - eg-write
    - write系统调用不能直接调用内核中的write代码，而是调用包装函数
    - 执行ecall指令使用参数sys_write，表示write系统调用
    - 将控制权给syscall，然后syscall可以分配到write系统调用
      - 中间涉及参数检查  内核不能写数据到不属于该应用程序的地方
    - 内核负责具体实现

#### Q
内核如何从用户手中夺回控制权，在用户程序是恶意的或者无限循环的
- 内核对硬件编程设置一个定时器  到时间了控制权就回来了

#### kernel = trusted computing base(TCB) 可信任计算
- kernel must have no bug
- kernel must treat processes as malicious
- monolithic kernel design 宏内核设计  将整个系统设计在内核里
  - 从安全角度讲  bug可能多一些
  - 好的方面是子模块联系紧密  性能好
- micro kernel design 微内核设计
  - kernel is small->few bugs
  - 所有与内核有关的东西必须经历跳入内核、跳出内核
  - 微内核设计比较难一些
  - 大多数桌面操作系统都是宏内核系统
  - 嵌入式系统多微内核

文件从C语言文件到汇编语言文件到二进制文件(.0)所有.o文件链接在一起成为我们最终想要的那个东西

lecture2 1h0m
#### 一些介绍
- 命令会有对应的十六进制编码(二进制)
- QEMU模拟RISV_V
  - 硬件技巧
  - QEMU内部支持GDB
  - 一些函数的调用顺序是特定的
  - userinit
