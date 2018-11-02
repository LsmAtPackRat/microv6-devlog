### 本文要点

本文分析xv6的启动流程。主要分为以下几个步骤：

- 



objdump -h bootblock.o

objdump -h kernel

其中的LMA就是物理地址（研究一下ld）



引导扇区的最后两个字节（结束标志）必须是55AAH，否则bios不认为0扇区是mbr。mbr不属于任何一个OS，而是公用的引导性质。

### 参考资料

dd命令的用法：http://czmmiao.iteye.com/blog/1748748

分析xv6的bootloader：https://www.cnblogs.com/fatsheep9146/p/5216681.html

gnu make：http://www.gnu.org/software/make/manual/make.html#Catalogue-of-Rules，提到了.S会自动cpp到.s，再as到.o文件。所以Makefile中可以不用写相关文件的构建过程。

一个简短的Makefile启动部分的分析：https://www.cnblogs.com/hygblog/p/9343068.html

阮一峰解释0x7C00：http://www.ruanyifeng.com/blog/2015/09/0x7c00.html

wiki BIOS interrupt：https://en.wikipedia.org/wiki/BIOS_interrupt_call

又一篇解释7c00：https://www.cnblogs.com/kuwoyidai/archive/2011/06/23/2105617.html

Oracle assembly language reference manual：https://docs.oracle.com/cd/E26502_01/html/E28388/eoiyg.html