---
layout: post
title: 用dumpdecrypted给App砸壳
category: opinion
description: 从商店下载下来的APP会带有一层壳，使得我们的其他工具，如chass-dump、IDA等无法使用，在小黄书上面曾介绍 AppCrackr 来砸壳，迫于一些原因，该工具已下架，现在来试试使用更偏geek一些的dumpdecrypted方式来给App砸壳。
---

##下载dumpdecrypted的源码
源码下载地址是 “[https://github.com/stefanesser/dumpdecrypted/archive/master.zip100](https://github.com/stefanesser/dumpdecrypted/archive/master.zip)”，下载后请将其解压至你习惯的位置，例如楼猪为/Users/mfw/reveal/，解压后生成/Users/mfw/reveal/dumpdecrypted-master。

##编译源码
进入到 dumpdecrypted-master 目录下，使用 make 命令来编译。

![CydiaSubstrate](/images/blog/options/dumpdecrypted/dumpdecrypted_1.jpg)

上面的make命令执行完毕后，会在当前目录下生成一个dumpdecrypted.dylib文件，这就是我们等下砸壳所要用到的榔头。此文件生成一次即可，以后可以重复使用，下次砸壳时无须再重新编译。

##用ps命令定位待砸壳的可执行文件

![CydiaSubstrate](/images/blog/options/dumpdecrypted/dumpdecrypted_2.jpg)

因为iOS上只打开了一个APP，所以唯一的那个含有/var/mobile/Containers/Bundle/Application/字样的结果就是TargetApp可执行文件的全路径(这里以新浪微博为例)。

注：可从Cydia中安装 adv-cmds 来使用 ps 命令。

##用 Cycript 找出 TargetApp 的 Documents 目录路径。

使用 TargetApp 或使用 PID 进入 cy，输入文件位置方法找出 Documents 路径。

	Warning:~ root# cycript -p WeiXin 
	cy#
	Warning:~ root# cycript -p 42365
	cy#
	cy#  [[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask][0]
	#"file:///var/mobile/Containers/Data/Application/F58D1528-46CE-442D-BA48-72A1E1D379E4/Documents/"
	cy#

##将dumpdecrypted.dylib拷贝到Documents目录下。拷贝命令如下：

	mfw at Warning.local in [~]
	16:53:52 › scp ~/reveal/dumpdecrypted-master/dumpdecrypted.dylib root@192.168.1.100:/var/mobile/Containers/Data/Application/F58D1528-46CE-442D-BA48-72A1E1D379E4/Documents/
	root@192.168.1.100's password:
	dumpdecrypted.dylib  		100%  193KB 192.9KB/s   00:00
	
	mfw at Warning.local in [~]
	
这里采用的是scp方式，也可以使用iFunBox等工具来操作。

好，到这一步，前期工作算是完成，到了期待已久的砸壳过程了，是不是很兴奋 ~ ~ ~

##开始砸壳
砸壳指令为：

	DYLD_INSERT_LIBRARIES=/path/to/dumpdecrypted.dylib /path/to/executable
	
实际操作起来就是：

	DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib /var/mobile/Containers/Bundle/Application/38EBD1D3-3D7D-48A5-BE36-4F4F5B3DAA25/WeiboHDPro.app/WeiboHDPro mach-o decryption dumper
	
会出现如下提示

	DISCLAIMER: This tool is only meant for security research purposes, not for application crackers.

	[+] detected 64bit ARM binary in memory.
	[+] offset to cryptid found: @0x1000acc08(from 0x1000ac000) = c08
	[+] Found encrypted data at address 00004000 of length 8962048 bytes - type 1.
	[+] Opening /private/var/mobile/Containers/Bundle/Application/38EBD1D3-3D7D-48A5-BE36-4F4F5B3DAA25/WeiboHDPro.app/WeiboHDPro for reading.
	[+] Reading header
	[+] Detecting header type
	[+] Executable is a FAT image - searching for right architecture
	[+] Correct arch is at offset 13680640 in the file
	[+] Opening WeiboHDPro.decrypted for writing.
	[+] Copying the not encrypted start of the file
	[+] Dumping the decrypted data into the file
	[+] Copying the not encrypted remainder of the file
	[+] Setting the LC_ENCRYPTION_INFO->cryptid to 0 at offset d0cc08
	[+] Closing original file
	[+] Closing dump file
	
查看当前目录下文件，是否有个 WeiboHDPro.decrypted 。出现了则说明成功了。使用 scp 导出来使用 class-dupp、IDA 来分析她吧 ~ ~ ~

##把dumpdecrypted.dylib拷贝到Documents目录下操作的原因

StoreApp对沙盒以外的绝大多数目录没有写权限。dumpdecrypted.dylib要写一个decrypted文件，但它是运行在StoreApp中的，与StoreApp的权限相同，那么它的写操作就必须发生在StoreApp拥有写权限的路径下才能成功。StoreApp一定是能写入其Documents目录的，因此我们在Documents目录下使用dumpdecrypted.dylib时，保证它能在当前目录下写一个decrypted文件，这就是把dumpdecrypted.dylib拷贝到Documents目录下操作的原因。

该文章转载自：[http://bbs.iosre.com/t/dumpdecrypted-app/22](http://bbs.iosre.com/t/dumpdecrypted-app/22)， 亲测可行，有问题欢迎提问 ~
