# Lab 4 实验报告

## 说明
`编程实验在自己的机器上测试时make grade可以得到全部分数。如果助教测试时发生Bug，希望能够进行通知。`

`由于文本编辑器的自动格式化功能，某些文件的结尾多余换行或行尾空格可能被删除。有些删除可能发生在敏感区域（如正确性检查程序），但实际上这些更改并不试图扰乱任何测试机制。`

## 练习1
alloc_proc的实现较为简单，主要是为一个proc_struct结构体对象分配内存，初始化其中的成员并返回这个对象。注意到alloc_proc中进行的初始化只是为了避免结构体的内容不确定。很多真正的初始化工作（例如进程ID分配什么的）都是要留到练习二里面再去做的。

关于context和tf这两个变量，context指的是进程的“上下文”，实际上其中保存了进程运行过程中的几乎全部寄存器，只有这样才能让进程返回时能够处在和原来一样的状态（eax之所以没有保存是因为不需要，fork时eax本身就是用来做返回值的），而tf是每个进程内核栈中的一个地址，指向该进程的陷入帧。在进程切换返回之后，系统可以根据这个陷入帧回到进程之前的状态。

## 练习2

do_fork的执行流程基本如注释所解释的那样，首先为新的进程控制块分配内存，接下来，将fork出来的进程的父进程设置为当前进程。新的进程需要的内核栈和内存管理对象mm也会被初始化。最后，进程会被添加到一个链表和一个以进程ID为散列函数的散列表之中，方便进程的查找和维护。新创建的进程最后会被唤醒。

uCore的进程id是唯一的，分配进程ID的任务由get_pid函数完成。一般情况下，get_pid会维护一个下一次创建的进程的ID，这个数值每次自增1，万一进程序号用完了，会重新从1开始回转，get_pid会检查已经分配的进程的序号避免重复和进行补救。

# 练习3
LAB4的实验中创建了两个内核线程，一个是idle线程，一个是init线程。idle线程是一个特殊的线程，用于在没有其他进程活动的时候占用CPU时间，在实际系统中idle进程占用的时间和CPU使用率及其功耗管理密切相关。init线程是真正执行任务的第一个线程，在本例中它也只是打印了一些字符串而已。

语句local_intr_save(intr_flag);....local_intr_restore(intr_flag);用于保存/恢复当前的中断标志位。在这段代码中间的代码执行时，中断都处于关闭状态。这用于保护关键代码执行不被中断打断——例如进程调度的相关代码执行时如果有跑出一个计时器中断引发调度显然有可能产生不可预期的后果。
