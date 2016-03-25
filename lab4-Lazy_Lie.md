# Lab4 report

## [练习一]
**设计实现**
> 按照注释的提示，将相关变量置为0或NULL。

** 请说明proc_struct中struct context context和struct trapframe *tf成员变量含义和在本实验中的作用是啥？**

> context类似进程的入口。tf保存在中断时寄存器的值，在进程启动的时候也帮助跳转。

## [练习二]
**设计实现**
> 按照注释的提示实现

**请说明ucore是否做到给每个新fork的线程一个唯一的id？请说明你的分析和理由。**
> 是的，因为在操作系统中每个进程都有一个唯一的id，而fork会产生一个进程，因此会分配一个id。`get_pid`函数保证了产生的id与现有id不冲突。

## [练习三]
**分析**
> 首先设置esp和cr3，然后执行`switch_to`。在`switch_to`中构造了一个调用栈，通过`ret`实现新进程的启动。

**在本实验的执行过程中，创建且运行了几个内核线程？**
> 2

**语句local_intr_save(intr_flag);....local_intr_restore(intr_flag);在这里有何作用?请说明理由**
> 关闭和恢复中断标识。

