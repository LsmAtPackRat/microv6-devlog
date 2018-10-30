### 本文要点

本文中，将完成microv6的实验环境的搭建，microv6基于xv6内核改造，所以首先需要能够编译并运行xv6。

编译两样东西：

- qemu；
- xv6；

### 编译qemu

首先，git clone一份qemu的源码（选用了mit的版本），并更新子模块：

```shell
git clone https://github.com/geofft/qemu.git -b 6.828-1.7.0
git submodule update --init pixman
git submodule update --init dtc
```

然后，把一些编译要用的工具，没装的装上（可能有些不需要，可以按需安装）：

```shell
sudo apt-get install libsdl1.2-dev
sudo apt-get install autoreconf
sudo apt-get install dh-autoreconf
sudo apt-get install autoconf2.13
sudo apt-get install libglib2.0-dev
sudo apt-get install zlib1g-dev
```

配置编译：

```shell
./configure --disable-kvm --disable-werror --target-list="i386-softmmu x86_64-softmmu"
make
make install
```

运行一下试试：

```shell
qemu-system-i386
```

如果看到了一个qemu进程启动起来，那么就ok了。



### 编译xv6

首先，git clone一份xv6的源码：

```shell
git clone https://github.com/mit-pdos/xv6-public.git
```

然后，编译之：

```shell
cd xv6-public/
make
```

这样，就编译好了，源码根目录下出现了xv6.img、fs.img两个镜像文件。接下来就要在qemu上启动xv6：

```shell
make qemu
```

这时候就可以看到启动了一个qemu的GUI窗口，其中运行了xv6的shell。

启动也可以通过：

```shell
qemu-system-i386 -serial mon:stdio -hdb fs.img xv6.img -smp 2 -m 512 -nographic
```

上面的命令中，-serial指定了串口输出到host的shell，-nographic表明不要qemu的GUI界面。-smp表示启动的core数，-m表示使用多少MB的RAM，-hdb指定镜像文件。



### 结语

此时xv6就已经启动起来了。可以试用一下它简陋的功能。接下来要对其进行源码分析+改造，逐步形成一个：

- 基于微内核架构，使用消息在各个OS service/Application之间通信；
- 能够针对性能需求弹性伸缩各个OS service/Application所占用的资源；

的新的kernel——microv6（代码仓库：https://github.com/LsmAtPackRat/microv6 ）。



这是目前的初步愿景。

2018.10.30