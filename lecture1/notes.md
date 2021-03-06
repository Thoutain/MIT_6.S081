```markdown
6.S081 2020 第 1 讲：O/S 概述

概述

* 6.S081 目标
  * 了解操作系统 (O/S) 设计和实现
  * 扩展小型 O/S 的实践经验
  * 编写系统软件的实践经验

* O/S 的目的是什么？
  * 抽象硬件以方便和便携
  * 在许多应用程序之间复用硬件
  * 隔离应用程序以包含错误
  * 允许协作应用程序之间共享
  * 控制共享以确保安全
  * 不要妨碍高性能
  * 支持广泛的应用

* 组织：分层图片
  [用户/内核图]
  - 用户应用程序：vi、gcc、DB、&c
  - 内核服务
  - 硬件：CPU、RAM、磁盘、网络等
  * 我们非常关心接口和内部内核结构

* O/S 内核通常提供哪些服务？
  * 进程（正在运行的程序）
  * 内存分配
  * 文件内容
  * 文件名、目录
  * 访问控制（安全）
  * 许多其他：用户、IPC、网络、时间、终端

* 什么是应用程序/内核接口？
  *“系统调用”
  * 来自 UNIX（例如 Linux、macOS、FreeBSD）的 C 语言示例：

            fd = open("out", 1);
            写（fd，“你好\n”，6）；
            pid = fork();

  * 这些看起来像函数调用，但它们不是 

* 为什么 O/S 设计 + 实施既困难又有趣？
  * 无情的环境：古怪的硬件，难以调试
  * 许多设计张力：
    - 高效 vs 抽象/便携/通用
    - 强大与简单的界面
    - 灵活 vs 安全
  * 功能交互：`fd = open(); 叉（）`
  * 用途多种多样：笔记本电脑、智能手机、云、虚拟机、嵌入式
  * 不断发展的硬件：NVRAM、多核、快速网络

* 如果您...
  *关心引擎盖下发生的事情
  * 像基础设施
  * 需要追踪错误或安全问题
  * 关心高性能

类结构

* 在线课程信息：
  https://pdos.csail.mit.edu/6.S081/——日程安排、作业、实验室
  广场——公告、讨论、实验室帮助

* 讲座
  * 操作系统的想法
  * xv6 的案例研究，一个小的 O/S，通过代码和 xv6 书
  * 实验室背景
  * O/S 文件
  * 在讲课前提交有关每次阅读的问题。

* 实验室： 
  重点：亲身体验
  每次最多一个星期。
  三种：
    系统编程（下周到期……）
    O/S 原语，例如线程切换。
    O/S 内核扩展到 xv6，例如网络。
  使用广场询问/回答实验室问题。
  讨论很好，但请不要看别人的解决方案！

* 分级：
  70% 的实验室，基于测试（您运行的相同测试）。
  20% 的实验室检查会议：我们会向您询问随机选择的实验室。
  10% 的家庭作业和课堂/广场讨论。
  不考试，不考试。
  请注意，大部分成绩来自实验室。早点开始吧！

UNIX 系统调用简介

* 应用程序通过系统调用查看操作系统；该界面将是一个很大的焦点。
  让我们从程序如何使用系统调用开始。
  您将在第一个实验中使用这些系统调用。
  并在随后的实验室中扩展和改进它们。

* 我将展示一些示例，并在 xv6 上运行它们。
  xv6 与 UNIX 系统（如 Linux）具有相似的结构。
  但要简单得多——你将能够消化所有的 xv6
    随附的书解释了 xv6 的工作原理以及原因
  为什么是 UNIX？
    开源，有据可查，干净的设计，广泛使用
    如果您需要深入了解 Linux，学习 xv6 会有所帮助
  xv6 在 6.S081 中有两个角色：
    核心功能示例：虚拟内存、多核、中断等
    大多数实验室的起点
  xv6 在 RISC-V 上运行，如当前 6.004
  您将在 qemu 机器模拟器下运行 xv6

* 示例：copy.c，将输入复制到输出
  从输入读取字节，将它们写入输出
  $复制
  copy.c 是用 C 写的
    Kernighan and Ritchie (K&R) 的书很适合学习 C
  您可以通过网站上的时间表找到这些示例程序
  read() 和 write() 是系统调用
  第一个 read()/write() 参数是一个“文件描述符”（fd）
    传递给内核以告诉它要读/写哪个“打开的文件”
    之前必须已打开
    FD 连接到文件/设备/套接字/&c
    一个进程可以打开很多文件，有很多FD
    UNIX 约定：fd 0 是“标准输入”，1 是“标准输出”
  第二个 read() 参数是一个指向要读取的内存的指针
  第三个参数是要读取的最大字节数
    read() 可能读得更少，但不会读得更多
  返回值：实际读取的字节数，或 -1 表示错误
  注意：copy.c 不关心数据的格式
    UNIX I/O 是 8 位字节
    解释是特定于应用程序的，例如数据库记录、C 源代码等
  文件描述符从何而来？

* 示例：open.c，创建文件
  $开盘
  $猫输出.txt
  open() 创建一个文件，返回一个文件描述符（或 -1 表示错误）
  FD 是一个小整数
  FD 索引到由内核维护的每个进程表中
  [用户/内核图]
  不同的进程有不同的 FD 命名空间
    即 FD 1 通常对不同的过程意味着不同的东西
  这些例子忽略了错误——不要这么草率！
  xv6 书中的图 1.2 列出了系统调用参数/返回
    或查看 UNIX 手册页，例如“man 2 open”

* 当程序调用像 open() 这样的系统调用时会发生什么？
  看起来像一个函数调用，但它实际上是一个特殊的指令
  硬件保存一些用户寄存器
  硬件增加特权级别
  硬件跳转到内核中的已知“入口点”
  现在在内核中运行 C 代码
  内核调用系统调用实现
    open() 在文件系统中查找名称
    它可能会等待磁盘
    它更新内核数据结构（缓存、FD 表）
  恢复用户注册
  降低权限级别
  跳回到程序中的调用点，它继续
  我们将在课程后面看到更多细节

* 我一直在输入 UNIX 的命令行界面 shell。
  shell 打印“$”提示。
  shell 允许您运行 UNIX 命令行实用程序
    对系统管理、文件处理、开发、脚本编写很有用
    $ls
    $ ls > 出
    $ grep x < 输出
  UNIX 也支持其他风格的交互
    窗口系统、GUI、服务器、路由器等。
  但是通过 shell 进行分时是 UNIX 最初的重点。
  我们可以通过 shell 执行许多系统调用。

* 示例：fork.c，创建一个新进程
  shell 为您键入的每个命令创建一个新进程，例如
    $回声你好
  fork() 系统调用创建一个新进程
    $叉
  内核复制调用进程
    指令、数据、寄存器、文件描述符、当前目录
    “父”和“子”进程
  唯一的区别：fork() 在父中返回 pid，在子中返回 0
  pid（进程 ID）是一个整数，内核给每个进程一个不同的 pid
  因此：
    fork.c 的“fork() 返回”在 *both* 进程中执行
    "if(pid == 0)" 允许代码区分
  好的，fork 让我们创建一个新进程
    我们如何在该过程中运行程序？

* 示例：exec.c，用可执行文件替换调用进程
  shell如何运行程序，例如
    $回声abc
  程序存储在文件中：指令和初始内存
    由编译器和链接器创建
  所以有一个名为 echo 的文件，其中包含说明
  $执行
  exec() 用一个可执行文件替换当前进程
    丢弃指令和数据存储器
    从文件加载指令和内存
    保留文件描述符
  exec（文件名，参数数组）
    参数数组保存命令行参数；exec 传递给 main()
    猫用户/echo.c
    echo.c 显示程序如何查看其命令行参数

* 示例：forkexec.c，fork() 一个新进程，exec() 一个程序
  $ forkexec
  forkexec.c 包含一个常见的 UNIX 习惯用法：
    fork() 一个子进程
    exec() 子进程中的一个命令
    父等待（）让孩子完成
  shell 为您输入的每个命令执行 fork/exec/wait
    在wait()之后，shell打印下一个提示
    在后台运行 -- & -- shell 跳过 wait()
  退出（状态）-> 等待（&状态）
    状态约定：0 = 成功，1 = 命令遇到错误
  注意：fork() 复制，但 exec() 丢弃复制的内存
    这可能看起来很浪费
    您将透明地消除“写时复制”实验室中的副本

* 示例：redirect.c，重定向命令的输出
  外壳为此做了什么？
    $ echo hello > out
  答案：fork，在child中改变FD 1，exec echo
  $重定向
  $猫输出.txt
  注意：open() 总是选择最低的未使用 FD；1 由于关闭(1)。
  fork、FDs 和 exec 很好地交互以实现 I/O 重定向
    单独的 fork-then-exec 让孩子有机会在 exec 之前更改 FD
    FD 提供间接性
      命令只使用 FD 0 和 1，不必知道它们去哪里
    exec 保留 sh 设置的 FD
  因此：只有 sh 必须知道 I/O 重定向，而不是每个程序

* 关于设计决策，值得问“为什么”：
  为什么会有这些 I/O 和进程抽象？为什么不是别的？
  为什么要提供文件系统？为什么不让程序以自己的方式使用磁盘？
  为什么是FD？为什么不将文件名传递给 write()？
  为什么文件是字节流，而不是磁盘块或格式化记录？
  为什么不结合 fork() 和 exec()？
  UNIX 设计运行良好，但我们将看到其他设计！

* 示例：pipe1.c，通过管道通信
  shell是如何实现的
    $ ls | grep x
  $pipe1
  FD 可以指“管道”，也可以指文件
  pipe() 系统调用创建了两个 FD
    从第一个 FD 读取
    写入第二个 FD
  内核为每个管道维护一个缓冲区
    [u/k 图表]
    write() 附加到缓冲区
    read() 等待直到有数据

* 示例：pipe2.c，进程间通信
  管道与 fork() 很好地结合实现 ls | grep x
    shell 创建一个管道，
    然后叉（两次），
    然后将 ls 的 FD 1 连接到管道的写入 FD，
    和 grep 的 FD 0 到管道
    [图表]
  $ pipe2 -- 简化版 
  管道是一个单独的抽象，但与 fork() 结合得很好

* 示例：list.c，列出目录中的文件
  ls 如何获取目录中的文件列表？
  您可以打开一个目录并读取它 -> 文件名
  “。” 是进程当前目录的伪名称
  有关更多详细信息，请参阅 ls.c

* 概括
  * 我们已经研究了 UNIX 的 I/O、文件系统和进程抽象。
  * 接口很简单——只有整数和 I/O 缓冲区。
  * 抽象结合得很好，例如用于 I/O 重定向。

您将在下周到期的第一个实验中使用这些系统调用。
```