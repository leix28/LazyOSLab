# Lab1 report

## [练习1]
**1.1 操作系统镜像文件 ucore.img 是如何一步一步生成的?(需要比较详细地解释 Makefile 中
每一条相关命令和命令参数的含义,以及说明命令导致的结果)**

> 使用`make "V="`可以看到具体执行了那些命令。

```
# 创建bin/kernel
# cc用gcc对指定的.c和.s文件进行编译，产生.o。
+ cc kern/init/init.c
gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o
# 之后的cc命令与上一条完全类似。
+ cc kern/libs/stdio.c
+ cc kern/libs/readline.c
+ cc kern/debug/panic.c
+ cc kern/debug/kdebug.c
+ cc kern/debug/kmonitor.c
+ cc kern/driver/clock.c
+ cc kern/driver/console.c
+ cc kern/driver/picirq.c
+ cc kern/driver/intr.c
+ cc kern/trap/trap.c
+ cc kern/trap/vectors.S
+ cc kern/trap/trapentry.S
+ cc kern/mm/pmm.c
+ cc libs/string.c
+ cc libs/printfmt.c
# 链接之前产生的.o。
+ ld bin/kernel
ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel  obj/kern/init/init.o obj/kern/libs/stdio.o obj/kern/libs/readline.o obj/kern/debug/panic.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/picirq.o obj/kern/driver/intr.o obj/kern/trap/trap.o obj/kern/trap/vectors.o obj/kern/trap/trapentry.o obj/kern/mm/pmm.o  obj/libs/string.o obj/libs/printfmt.o

------------
# 创建bootblock
+ cc boot/bootasm.S
+ cc boot/bootmain.c
+ cc tools/sign.c
# 编译sign工具
gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign
# 链接bootblock
+ ld bin/bootblock
ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
'obj/bootblock.out' size: 488 bytes
build 512 bytes boot sector: 'bin/bootblock' success!

------------
# 创建ucore.img
dd if=/dev/zero of=bin/ucore.img count=10000
dd if=bin/bootblock of=bin/ucore.img conv=notrunc
```


**1.2 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么?**

> 主引导扇区只有512字节。第510个字节是0x55，第511个字节是0xAA。



## [练习2]

**2.1 从 CPU 加电后执行的第一条指令开始,单步跟踪 BIOS 的执行。**

> 可以单步跟踪，方法如下：
 
> 1 修改 lab1/tools/gdbinit,内容为:

```
set architecture i8086
target remote :1234
```

> 2 在 lab1目录下，执行`make debug`

> 3 在看到gdb的调试界面(gdb)后，在gdb调试界面下执行`si`即可单步跟踪BIOS了。

> 4 在gdb界面下，可通过`x /2i $pc`来看BIOS的代码

**在初始化位置0x7c00 设置实地址断点,测试断点正常。**

> 1 把tools/gdbinit改为


```
file bin/kernel
target remote :1234
set architecture i8086
b *0x7c00
c
x /10i $pc
set architecture i386
```
	
> 运行"make debug"便可得到

```
=> 0x7c00:      cli
   0x7c01:      cld
   0x7c02:      xor    %ax,%ax
   0x7c04:      mov    %ax,%ds
   0x7c06:      mov    %ax,%es
   0x7c08:      mov    %ax,%ss
   0x7c0a:      in     $0x64,%al
```

**在调用qemu 时增加-d in_asm -D q.log 参数，便可以将运行的汇编指令保存在q.log 中。
将执行的汇编代码与bootasm.S 和 bootblock.asm 进行比较，看看二者是否一致。**

> 把tools/gdbinit改为

```
file bin/kernel
target remote :1234
set architecture i8086
b *0x7c00
c
x /10i $pc
set architecture i386
```

> 在`makefile`中`debug:`的对应位置加入`-d in_asm -D q.log`

> 便可以在q.log中读到"call bootmain"前执行的命令

```
----------------
IN: 
0x00007c00:  cli    

----------------
IN: 
0x00007c01:  cld    
0x00007c02:  xor    %ax,%ax
0x00007c04:  mov    %ax,%ds
0x00007c06:  mov    %ax,%es
0x00007c08:  mov    %ax,%ss

----------------
IN: 
0x00007c0a:  in     $0x64,%al

----------------
IN: 
0x00007c0c:  test   $0x2,%al
0x00007c0e:  jne    0x7c0a

----------------
IN: 
0x00007c10:  mov    $0xd1,%al
0x00007c12:  out    %al,$0x64
0x00007c14:  in     $0x64,%al
0x00007c16:  test   $0x2,%al
0x00007c18:  jne    0x7c14

----------------
IN: 
0x00007c1a:  mov    $0xdf,%al
0x00007c1c:  out    %al,$0x60
0x00007c1e:  lgdtw  0x7c6c
0x00007c23:  mov    %cr0,%eax
0x00007c26:  or     $0x1,%eax
0x00007c2a:  mov    %eax,%cr0

----------------
IN: 
0x00007c2d:  ljmp   $0x8,$0x7c32

----------------
IN: 
0x00007c32:  mov    $0x10,%ax
0x00007c36:  mov    %eax,%ds

----------------
IN: 
0x00007c38:  mov    %eax,%es

----------------
IN: 
0x00007c3a:  mov    %eax,%fs
0x00007c3c:  mov    %eax,%gs
0x00007c3e:  mov    %eax,%ss

----------------
IN: 
0x00007c40:  mov    $0x0,%ebp

----------------
IN: 
0x00007c45:  mov    $0x7c00,%esp
0x00007c4a:  call   0x7d0d
```

> 其与bootasm.S和bootblock.asm中的代码相同，一些标识符被替换为相应的地址。

## [练习3]
**分析bootloader 进入保护模式的过程。**

> 从`%cs=0 $pc=0x7c00`，进入后。先将`ds`, `es`, `ss`置为0。

```
.code16
    cli
    cld
    xorw %ax, %ax
    movw %ax, %ds
    movw %ax, %es
    movw %ax, %ss
```

> 开启A20：通过将键盘控制器上的A20线置于高电位，全部32条地址线可用，
可以访问4G的内存空间。

```
seta20.1:             
    inb $0x64, %al       
    testb $0x2, %al     
    jnz seta20.1        

    movb $0xd1, %al     
    outb %al, $0x64     

seta20.1:               
    inb $0x64, %al       
    testb $0x2, %al     
    jnz seta20.1        

    movb $0xdf, %al     
    outb %al, $0x60      
```

> 初始化GDT表：一个简单的GDT表和其描述符已经静态储存在引导区中，载入即可

```
    lgdt gdtdesc
```

> 进入保护模式：通过将cr0寄存器PE位置1便开启了保护模式

```
    movl %cr0, %eax
    orl $CR0_PE_ON, %eax
    movl %eax, %cr0
```

>通过长跳转更新cs的基地址

```
	 ljmp $PROT_MODE_CSEG, $protcseg
	.code32
	protcseg:
```

> 设置段寄存器，并建立堆栈

```
    movw $PROT_MODE_DSEG, %ax
    movw %ax, %ds
    movw %ax, %es
    movw %ax, %fs
    movw %ax, %gs
    movw %ax, %ss
    movl $0x0, %ebp
    movl $start, %esp
```
> 转到保护模式完成，进入boot主方法

```
    call bootmain
```


## [练习4]
**bootloader如何读取硬盘扇区的？**

> 读取扇区需要把参数写道相应的地址，然后等待磁盘操作，最后从内存地址获取读到的内容。

```
static void
readsect(void *dst, uint32_t secno) {
    // wait for disk to be ready
    waitdisk();

    outb(0x1F2, 1);                         // count = 1
    outb(0x1F3, secno & 0xFF);
    outb(0x1F4, (secno >> 8) & 0xFF);
    outb(0x1F5, (secno >> 16) & 0xFF);
    outb(0x1F6, ((secno >> 24) & 0xF) | 0xE0);
    outb(0x1F7, 0x20);                      // cmd 0x20 - read sectors

    // wait for disk to be ready
    waitdisk();

    // read a sector
    insl(0x1F0, dst, SECTSIZE / 4);
}
```

**bootloader是如何加载ELF格式的OS？**
> 先读取第一个段，检查合法性。

```
readseg((uintptr_t)ELFHDR, SECTSIZE * 8, 0);

if (ELFHDR->e_magic != ELF_MAGIC) {
    goto bad;
}
```

> 根据第一段的数据读取之后的段。

```
struct proghdr *ph, *eph;

ph = (struct proghdr *)((uintptr_t)ELFHDR + ELFHDR->e_phoff);
eph = ph + ELFHDR->e_phnum;
for (; ph < eph; ph ++) {
    readseg(ph->p_va & 0xFFFFFF, ph->p_memsz, ph->p_offset);
}

((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))();
```


## [练习5] 
**实现函数调用堆栈跟踪函数** 

```
int i;
uint32_t ebp = read_ebp();
uint32_t eip = read_eip();
for (i = 0; ebp && i < STACKFRAME_DEPTH; i++) {
	cprintf("ebp:0x%08x eip:0x%08x ", ebp, eip);
	uint32_t *args = (uint32_t*)ebp + 2;
	cprintf("args:0x%08x 0x%08x 0x%08x 0x%08x\n", \
	        args[0], args[1], args[2], args[3]);
	print_debuginfo(eip - 1);
	eip = args[-1];
	ebp = args[-2];
}
```


> 最后一项输出为

```
ebp:0x00007bf8 eip:0x00007d6e args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8
    <unknow>: -- 0x00007d6d --
```

> 其对应的是第一个使用堆栈的函数，bootmain.c中的bootmain。

## [练习6]
**中断向量表中一个表项占多少字节？其中哪几位代表中断处理代码的入口？**

中断向量表一个表项占用8字节，其中2-3字节是段选择子，0-1字节和6-7字节拼成位移，
两者联合便是中断处理程序的入口地址。

**请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init。**


**请编程完善trap.c中的中断处理函数trap，在对时钟中断进行处理的部分填写trap函数。**


------------
**完成实验后，请分析ucore_lab中提供的参考答案，并请在实验报告中说明你的实现与参考答案的区别**

> 我的实现和参考答案大体上一致，在最初，我错误的把ticks进行了清零操作，这是不正确的，因为会影响到之后的结果。

**列出你认为本实验中重要的知识点，以及与对应的OS原理中的知识点，并简要说明你对二者的含义，关系，差异等方面的理解（也可能出现实验中的知识点没有对应的原理知识点）**

> 实验主要包括了X86的栈使用方法，以及中断机制。

**列出你认为OS原理中很重要，但在实验中没有对应上的知识点**

