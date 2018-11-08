### 本文要点

本文分析xv6的启动流程。

xv6的启动流程，主要分为以下几个步骤：

1. BIOS从磁盘加载boot sector到内存，并运行其中的boot loader；
2. boot loader从磁盘上加载kernel到内存，由entry进入kernel；
3. 建立entrypgdir页表，并开启分页，kernel进入main函数；
4. 内核初始化；
5. 建立第一个用户进程，重点在于构造其kernel stack；
6. 启动boot processor上的scheduler（一个无限循环），开启boot processor上的进程调度，完成boot processor的设置。
7. 第一个用户进程会被某个CPU的scheduler调度，执行initcode.S中的start函数（这步实现依靠的是上一步kernel stack的构造）。第一个用户进程的start函数调用syscall：exec(init, argv)，演变成init进程；
8. init进程fork并exec出shell（sh进程），而init进程则调用wait()等待shell进程结束。

至此，kernel启动的所有步骤执行完毕。接下来对上述每一个步骤，进行细致的代码分析。

------

### BIOS加载boot sector

xv6.img分为两个部分：

（图片展示bootblock+kernel）

其中bootblock占开头的512字节，它被存放磁盘的boot sector中。紧接着的剩余部分就是kernel。



#### bootblock的构建

是bootasm.S和bootmain.c两个文件链接而成的目标文件：

```shell
bootblock: bootasm.S bootmain.c
$(CC) $(CFLAGS) -fno-pic -O -nostdinc -I. -c bootmain.c
$(CC) $(CFLAGS) -fno-pic -nostdinc -I. -c bootasm.S
# ld -Ttext ADDRESS (Set address of .text section) ? what's the meaning?
# ld -e/--entry ADDRESS (Set start address)
$(LD) $(LDFLAGS) -N -e start -Ttext 0x7C00 -o bootblock.o bootasm.o bootmain.o 
$(OBJDUMP) -S bootblock.o > bootblock.asm
# '-j .text' means only copy .text section from bootblock.o to bootblock
$(OBJCOPY) -S -O binary -j .text bootblock.o bootblock
./sign.pl bootblock
```



#### BIOS功能简介

BIOS是存储在ROM中的一个非常小的操作系统。它的任务如下：

1. 初始化硬件；
2. 加载boot loader，并将控制权转交给它。

我们现在只关心加载boot loader的细节。BIOS会加载boot sector到RAM的0x7C00处（这个一个惯例，不是xv6特有的，原因可见：[阮一峰解释0x7C00](http://www.ruanyifeng.com/blog/2015/09/0x7c00.html)），并将CPU的%ip设置为0x7C00以跳转到这个地址执行。



### boot loader加载kernel

kernel的开头4096个字节存放的是ELF header，紧接着存放的是kernel的每一个segment的program header table。



### entry开启分页



### 内核初始化

逐个启动其他non-boot (AP) processors。





### 构造第一个用户进程



### 开启boot processor的进程调度



### 第一个用户进程演变为init进程



### 启动shell进程









objdump -h bootblock.o

objdump -h kernel

其中的LMA就是物理地址（研究一下ld）



引导扇区的最后两个字节（结束标志）必须是55AAH，否则bios不认为0扇区是mbr。mbr不属于任何一个OS，而是公用的引导性质。

------



### 参考资料

dd命令的用法：http://czmmiao.iteye.com/blog/1748748

分析xv6的bootloader：https://www.cnblogs.com/fatsheep9146/p/5216681.html

gnu make：http://www.gnu.org/software/make/manual/make.html#Catalogue-of-Rules，提到了.S会自动cpp到.s，再as到.o文件。所以Makefile中可以不用写相关文件的构建过程。

一个简短的Makefile启动部分的分析：https://www.cnblogs.com/hygblog/p/9343068.html

[阮一峰解释0x7C00](http://www.ruanyifeng.com/blog/2015/09/0x7c00.html)

wiki BIOS interrupt：https://en.wikipedia.org/wiki/BIOS_interrupt_call

又一篇解释7c00：https://www.cnblogs.com/kuwoyidai/archive/2011/06/23/2105617.html

Oracle assembly language reference manual：https://docs.oracle.com/cd/E26502_01/html/E28388/eoiyg.html



ld链接脚本的官方文档解释：http://ftp.gnu.org/old-gnu/Manuals/ld-2.9.1/html_mono/ld.html#SEC21

一篇讲解AT关键字的博客：https://blog.csdn.net/redredbird/article/details/5986035

我现在理解的VMA是section被加载器加载时的虚拟地址。LMA则是section在可执行文件中的地址。如果不用AT指定的话，默认LMA=VMA，这样的话，section之间不够紧凑。可以使用AT关键字来指定section在输出目标文件中的位置，比如：

```
.data 0x8000 : AT(ADDR(.text) + SIZEOF(.text)) 
```

这种写法是的.data紧接着.text存放在elf中。可以起到压缩elf文件的作用。

XV6的用法是，将.text指定AT在XV6.img的Kernel部分的0X100000处。

```shell
lsm@lsm-VirtualBox:~/microv6$ readelf -l kernel

Elf file type is EXEC (Executable file)
Entry point 0x10000c
There are 2 program headers, starting at offset 52

Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  LOAD           0x001000 0x80100000 0x00100000 0x0a516 0x154a8 RWE 0x1000
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RWE 0x10

 Section to Segment mapping:
  Segment Sections...
   00     .text .rodata .stab .stabstr .data .bss 
   01    
```

 上述可以看出，kernel的需要load的segment只有一个，它位于elf文件的offset=0x001000处，也就是前面跳过了一个page的大小（ELF Header）。这个segment会被load到的虚拟地址是virtAddr，而加载到的物理地址是0x100000，它是由bootloader的C部分，来load的。

上面还可以看出，program header从ELF文件的offset=52开始定义。struct elfhdr的size正好是52个字节，所以说，program headers是紧随着elf header放置的。

《程序员的自我修养》的page-165提到segment的物理装载地址，就是section的LMA。