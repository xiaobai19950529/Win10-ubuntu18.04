# Win10-ubuntu18.04-
Win10+ubuntu18.04双系统双硬盘 真不容易啊- -好多的坑

这两天重装系统折腾了很久，本来以为很简单的事在实践中出现了很多以前没遇到过的问题，即兴在此记录一下踩过的坑，已解决的和未解决的。

重装Win10其实很简单，下一个镜像用大白菜或者UltraISO等U盘装系统制作软件即可，但要注意启动盘打开方式和顺序，系统装在SSD上，启动的时候要从对应的SSD启动

移动硬盘崩掉： 主要问题是结构损坏无法读取和参数错误。
解决方法： DiskGenius软件，恢复数据后，对硬盘进行格式化，但速度下降了不止一星半点，传小文件速度极其缓慢，基本可以废弃。虽说小文件非连续读取速度会下降，但是一直数小时保持在1M/s以下甚至几KB/s的情况真的可以丢弃。换了块新的西数2T移动硬盘，传输速度130M/s上下，20G的文件，20分钟

USB 3.0的使用：以前一直没注意驱动的事，机箱和设备，数据线都是3.0以为肯定是3.0的速度，结果这次发现速度不对劲，最后终于查出来是驱动的问题，有的电脑bios里默认关闭了USB的某些接口设置，需要去bios里全部开起来，再安装驱动，重启后就能使用USB3.0，达到百兆级的读写速度。
注意： USB 2.0 设备（U盘，移动硬盘等）的灯是白色的，而3.0是蓝色的。
Ubuntu 18.04和Win10 在 固态+机械硬盘 上的双系统安装
Win10就不谈了，很简单，下面说说如何装ubuntu18.04
计算机管理里专门开一个大小为100-200GB的卷，用于装ubuntu
用UltraISO将从交大的镜像库里下载好的Ubuntu 18.04.1镜像写到
U盘里，从此无尽的坑开始了……

1.	首先是U盘的识别问题，有的时候可能重启下电脑的时候拔U盘发现识别不出来，也就是说根本没法从U盘启动，就别提装系统了。
    此时需要进Windows里调出命令行，输入msconfig，在服务里将“隐藏所有Microsoft服务“勾选上，然后再去设备管理器下的“通用串行总线”里将未识别的USB设备     卸载，然后拔出U盘重新插。

2.	Ubuntu的设置分区问题，必须先设置逻辑分区再设置主分区，否则设置到后面会有一部分空间不可用。具体分配： 
    
    swap=内存大小
    
    /boot分区 500M  
    
    /  剩余所有空间 
    
    引导分区位置必须和/boot分区一致，需要调整
    在此我所有分区都设置为逻辑分区，因为我主分区在固态上，Win10装在SSD上，机械硬盘上没有主分区。

3.	安装中途发生Grub installation failure问题，直接导致安装程序崩溃，安装失败。

    原因猜想：出现此问题的原因可能是grub被装在了efi分区，而boot被挂载到非efi分区，导致grub找不到引导程序。而网上的很多教程里说用Easybcd里添加新条目     里的NeoGrub来安装配置，我配置后重启发现并没有找到文件，可能是这种方法不适合我的情况。

    实际原因： Win10启动方式和U盘启动方式不同，Win10是通过传统方式启动，U盘是通过UEFI方式启动（因为传统方式一直失败），最后发生Grub冲突。
    查看Win10启动方式： Win+R组合键里输入msinfo32，查看电脑Bios模式一栏：是Legacy BIOS或UEFI

    解决办法： U盘启动方式要与Win10保持一致，所以必须以传统方式启动。

4.	U盘在系统启动项选择时，有一个传统项和UEFI项，UEFI项可以打开，而传统项不行，会报错，”Failed to load ldlinux.c32” 

    解决方法： （以前从来没遇到过）用UltraISO写入硬盘的时候写入方式调为raw， 其他所有的写入方式一概不行（之前用USB-HDD+）。

5.	在以上问题都解决后，终于可以通过U盘以传统的方式启动进入Ubuntu18.04的安装程序，在以EXT4日志系统设置完挂载/boot和/根分区后安装完毕，拔出U盘，神奇     问题再次降临，启动ubuntu系统时直接进入grub命令行，并且ls命令都无法识别。
    
    解决方法： 搜索万千种方法后终于通过改为EXT3的挂载方式挂载了/boot和/根分区后 成功装上Ubuntu18.04。


在此后又翻阅了一些关于如何从Ext3无损转Ext4的资料发现都无法契合我自己的实际情况，因为没法直接Kill /boot分区和/根分区的所有进程，也就没法umount，这是暂未解决的问题。

另外小知识： (hdx,y) 代表了连接在本机上的第x个硬盘，第y个分区，比如C盘如果在第一块硬盘上的第一个分区，通常就是(hd0,0),因为C盘通常为主分区，主分区的编号从0到3，依次为第一个、第二个主分区，一块硬盘最多4个主分区。 逻辑分区最小从4开始，所以如果D盘是逻辑分区，那么D盘就是（hd0,4）
