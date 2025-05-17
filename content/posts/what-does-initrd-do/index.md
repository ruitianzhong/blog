---
title: "What do the scripts in Linux initrd.img do?"
subtitle: "Introducing some important stuff (in my opinion) in scripts of Linux initrd.
"
summary: "Introducing some important stuff (in my opinion) in scripts of Linux initrd."

date: 2024-02-01T16:20:00+08:00

lastmod: 2024-02-01T16:20:00+08:00

draft: false

featured: false

authors:
  - admin

categories:
  - initramfs
  - Linux
---

## 为什么需要initrd？

### 系统初始化

有很多初始化内核都不会进行，比如说/proc(`proc`)、/sys(`sysfs`)和/dev(`devtmpfs`)下有很多文件/目录，它们虽然是Kernel内部的一些数据的映射/表示，但是它们并不是内核自动挂载，需要程序/脚本将它们挂载。

```bash
mount -t proc none /proc 
mount -t sysfs none /sys
mount -t devtmpfs none /dev
```

如果不运行上面的命令，并不会自动出现/proc、/dev和/sys下的各种文件和目录。

当然，也有特殊情况，在/dev目录下也可以通过`mknod`命令创建设备文件，如果将这个设备文件保存在磁盘上的话，开机的时候/dev目录下也会有相应的内容。

### 挂载根文件系统

其实，Kernel也可以直接挂载根文件系统，只要在Kernel命令行参数当中添加root=/dev/vda来挂载文件系统。可以通过以下命令查看Kernel命令行参数：

```bash
cat /proc/cmdline
```

如果需要改变启动参数，请查阅GRUB的相关资料后修改并更新GRUB的配置。

但是root=/dev/hda这种形式在设备变化的时候很容易出问题。当电脑多了一个硬盘之后，这个硬盘可能就变成了`hda`，而我们真正希望挂载的硬盘变成了`hdb`，原有的命令行参数需要修改才能够让操作系统正常挂载正确的根文件系统。

解决这个问题的一个办法就是指定PARTUUID（Disk设备，一个分区一个PARTUUID，借助了`GUID Partition Table (GPT)`）或UUID（文件系统层面）,这些ID至少在同一台电脑上是独一无二的，理论上，这样只要指定想要挂载的硬盘/文件系统ID就可以正确地挂载。


因此，在一般的电脑上，root参数一般不是root=/dev/hda这种形式，而是` root=UUID=xxxxxxxx-xxxx...`的形式。

然而，Kernel实际上并不支持这样的参数，根据 [early-lookup.c](https://elixir.bootlin.com/linux/latest/source/block/early-lookup.c#L244)中的`early_lookup_bdev()`:

```c
int __init early_lookup_bdev(const char *name, dev_t *devt)
{
	if (strncmp(name, "PARTUUID=", 9) == 0)
		return devt_from_partuuid(name + 9, devt);
	if (strncmp(name, "PARTLABEL=", 10) == 0)
		return devt_from_partlabel(name + 10, devt);
	if (strncmp(name, "/dev/", 5) == 0)
		return devt_from_devname(name + 5, devt);
	return devt_from_devnum(name, devt);
}
```

kernel并不支持`root=UUID=xxxxxx-xxxx...`这种格式，因此，这样的工作交给了initrd中相关的脚本来做。具体来说，initrd中的脚本会先把/dev/disks/{by-uuid,by-path,by-label,by-partid}等目录建立好(这些文件并不是默认存在，内核也不会进行创建)，然后直接通过`/dev/disks/by-uuid/xxxxxx-xxxx`这个符号链接直接找到`root=UUID=xxxxxx-xxxx...`对应的设备。

因此我们需要initrd中的脚本/程序来挂载最终的文件系统。

### 便于调试和扩展

在用户态进行初始化也方便调试，根据`initramfs-tools(7)`，initrd中的脚本具有debug的选项，可以打印出比较有用的信息。

此外，bash脚本便于维护，`initramfs-tools(7)`支持在脚本hook中加入用户扩展的代码。


### 执行流程简述

initrd（`Init RAM Disk`）中包含了一个比较小的文件系统。

文件系统中包括了系统初始化所需的必要的脚本、二进制文件、内核驱动等。

当系统启动的时候BIOS/UEFI将Kernel(`vmlinux`)和initrd(`initrd.img`)加载进内存，Kernel能够识别到这个initrd.img,并将它作为初始的根文件系统。

当内核内部完成初始化以后，它将会运行initrd中`/init`，运行`sudo dmesg`后可以找到这样的信息:

```bash
[    1.255173] Run /init as init process
[    1.256441]   with arguments:
[    1.256443]     /init
[    1.256444]     noibrs
[    1.256445]   with environment:
[    1.256446]     HOME=/
[    1.256447]     TERM=linux
[    1.256447]     BOOT_IMAGE=/boot/vmlinuz-5.15.0-58-generic
[    1.256448]     vga=792
```

`/init`可能是shell脚本，也可能是ELF可执行文件，在Ubuntu的initrd当中

当内核将控制权转交给用户态的/init可执行程序后，/init执行各项初始化。设备发现、驱动加载、初始化各个目录等工作完成以后，最后通过`pivot_root(2)`这个系统调用切换根文件系统，并且执行根文件系统下的`/sbin/init`

```bash
export init=/sbin/init

# omit lots of code here 

exec run-init ${drop_caps} "${rootmnt}" "${init}" "$@" <"${rootmnt}/dev/console" >"${rootmnt}/dev/console" 2>&1
echo "Something went badly wrong in the initramfs."
panic "Please file a bug on initramfs-tools."
```

查看`/sbin/init`，可以看到它指向的正是大名鼎鼎的`systemd`:

```bash
$ file /sbin/init
/sbin/init: symbolic link to /lib/systemd/systemd
```


## initrd在哪里？

在Ubuntu的/boot目录下，有一个initrd.img的软链接文件，它是一个指向一个与Linux Kernel版本相关的文件`initrd.img-5.15.0-58-generic`(文件名可能由于Kernel版本不同和这个不一样)。

```bash
file /boot/initrd.img
/boot/initrd.img: symbolic link to initrd.img-5.15.0-58-generic
file /boot/initrd.img-5.15.0-58-generic
/boot/initrd.img-5.15.0-58-generic: ASCII cpio archive (SVR4 with no CRC)
```

如果要查看其中的内容，可以使用`unmkinitramfs`命令将它提取到一个目录initrd当中:

```bash
unmkinitramfs /boot/initrd.img initrd
```

注意，提取出来的initrd目录当中可能包括了Intel的microcode，main目录才是启动时真正挂载的文件系统。

```bash
ls
early  main
cd main/
ls
bin   cryptroot  init  lib32  libx32  sbin     usr
conf  etc        lib   lib64  run     scripts  var
```


## `udev`如何发挥作用？

`udevadm`这个程序在initrd和最终挂载的根文件系统都有，不同的是在initrd中udev在脚本(`/scripts/init-top/udev`)中启动：

```bash
udevadm trigger --type=subsystems --action=add
udevadm trigger --type=devices --action=add
udevadm settle || true
```

而在根文件系统当中，udevadm集成在systemd当中，当成一个daemon运行。

当还没有挂载最终的根文件系统之前，udev创建了/dev/by-{partuuid,uuid}等目录，并且根据已经挂载的设备加载相应的驱动，为最终挂载相应设备做准备。

### udev怎样根据已有的设备找到对应的驱动

拿阿里云ECS服务器上的vda（virtual disk）设备驱动加载举例，假定virtio_block驱动没有被编译进内核，udev怎样加载vda的相应的驱动？

为方便说明，vda的最终路径为`/sys/devices/pci0000:00/0000:00:05.0/virtio2/block/vda`



首先，udev通过Netlink和内核建立连接（理解为一个可以通信的信道就行了，细节可以暂时不深究），当有新的设备被发现的时候，内核在`/sys/devices/`目录下创建新的目录，比如先发现`pci0000:00`再发现`0000:00:05.0`再发现`virtio2`等，从而形成上面我们看到的完整路径，这样的路径也表示出PCI的物理上的树状结构。每发现一层，都会向uevent发送设备的相关信息。查看这些信息可以用如下命令：

```bash
$ cat /sys/devices/pci0000:00/0000:00:05.0/uevent
DRIVER=virtio-pci
PCI_CLASS=10000
PCI_ID=1AF4:1001
PCI_SUBSYS_ID=1AF4:0002
PCI_SLOT_NAME=0000:00:05.0
MODALIAS=pci:v00001AF4d00001001sv00001AF4sd00000002bc01sc00i00   
```

```bash
$ cat /sys/devices/pci0000:00/0000:00:05.0/virtio2/uevent
DRIVER=virtio_blk
MODALIAS=virtio:d00000002v00001AF4
```

```bash
cat /sys/devices/pci0000:00/0000:00:05.0/virtio2/block/vda/uevent
MAJOR=252
MINOR=0
DEVNAME=vda
DEVTYPE=disk
DISKSEQ=9
```

可以看到内核向udev发送了丰富的信息，比如说vda的主、次设备号和设备类型，我们还可以看到内核还向kernel发送了
DRIVER和MODLIAS这两个字段，而这两个字段都可以分别指定了udev应该加载哪些驱动。

拿上面的virtio_blk驱动举例，它的别名（MODALIAS）是`virtio:d00000002v00001AF4`。

通过名字找到驱动，从而加载相应驱动：


```bash
$ find /lib/modules/5.15.0-58-generic | grep virtio_blk.ko
/lib/modules/5.15.0-58-generic/kernel/drivers/block/virtio_blk.ko
```

通过别名同样可以找到相应的驱动，首先查看modules.alias这个文件中的内容：
```bash
$ cat /lib/modules/5.15.0-58-generic/modules.alias | grep virtio_blk
alias virtio:d00000002v* virtio_blk
```

可以看到`virtio:d00000002v00001AF4`匹配这条记录，因此可以通过MODALIAS查找到对应驱动的名称，从而加载相应的驱动。

因此通过MODALIAS和DRIVER都可以分别找到对应的驱动，但具体udev是怎样实现的还有待研究。但这些说明了一个事实：用户程序udev可以通过内核提供的某个信息找到正确的驱动并且将其动态加载。

### 一些细节

1. 当initrd的udev启动的时候可能有一些设备已经被发现了，udev是否会错过这些事件？

并不会错过，因为`udevadm trigger`模式下会重放所有的事件。

2. udev实际上是非常灵活的，有特定的一套配置语言，主要放在`/lib/udev/rules.d/`,比如说，和块设备命名相关的配置文件名字为`60-persistent-storage.rules`。关于udev的更多信息，可以查阅`udev(7)`。

## Reference

* [Using the initial RAM disk (initrd)](https://www.kernel.org/doc/html/latest/admin-guide/initrd.html)