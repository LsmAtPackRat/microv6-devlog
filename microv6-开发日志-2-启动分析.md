### 本文要点

本文分析xv6的启动流程。主要分为以下几个步骤：

- 



objdump -h bootblock.o

objdump -h kernel

其中的LMA就是物理地址（研究一下ld）



### 参考资料

dd命令的用法：http://czmmiao.iteye.com/blog/1748748

分析xv6的bootloader：https://www.cnblogs.com/fatsheep9146/p/5216681.html

gnu make：http://www.gnu.org/software/make/manual/make.html#Catalogue-of-Rules，提到了.S会自动cpp到.s，再as到.o文件。所以Makefile中可以不用写相关文件的构建过程。