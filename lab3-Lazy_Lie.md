# Lab3 report

## [练习一]
**设计实现**
> 按照注释的提示，使用ucore中相关的宏定义进行了实现。这不是一个具有创造性的工作。

**请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中组成部分对ucore实现页替换算法的潜在用处。**
> PDE中包含了权限、是否有效等信息，PTE中包含

```
#define PTE_P           0x001                   // Present
#define PTE_W           0x002                   // Writeable
#define PTE_U           0x004                   // User
#define PTE_PWT         0x008                   // Write-Through
#define PTE_PCD         0x010                   // Cache-Disable
#define PTE_A           0x020                   // Accessed
#define PTE_D           0x040                   // Dirty
#define PTE_PS          0x080                   // Page Size
#define PTE_MBZ         0x180                   // Bits must be zero
#define PTE_AVAIL       0xE00                   // Available for software use

```

> 这写标记可以用于页替换算法选择被换出的页

**如果ucore的缺页服务例程在执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？**

> 保存错误信息、寄存器的值，跳转到异常处理入口。


## [练习二]
**设计实现**
> 按照注释的提示，使用ucore中相关的宏定义进行了实现。这不是一个具有创造性的工作。

**如果要在ucore上实现"extended clock页替换算法"请给你的设计方案，现有的swap_manager框架是否足以支持在ucore中实现此算法？如果是，请给你的设计方案。如果不是，请给出你的新的扩展和基此扩展的设计方案。并需要回答如下问题**

> 现有的框架支持extended clock方法。对于内存中的页建立一个双向链表，每次根据extended clock的规则选取换出的页，以及修改对应的标志位。

- 需要被换出的页的特征是什么？
> 脏标记和使用标记都为0
- 在ucore中如何判断具有这样特征的页？
> 根据PTE表项中PTE_D, PTE_A标记位
- 何时进行换入和换出操作？
> 当出现缺页且内存已满的情况