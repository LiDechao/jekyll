---
layout: post
title: class-dump
category: opinion
description: class-dump 是用来 dump 目标对象的 class 的信息的工具。它利用 Object-C 的 runtime 特性，将存储在Mach-O文件中的文件信息提取出来，并生成对应的 .h 文件。
---

##下载地址

直接访问官网下载，[http://stevenygard.com/projects/class-dump/](http://stevenygard.com/projects/class-dump/)

##安装
下载 class-dump-3.5.dmg 之后，将 dmg 文件里的 class-dump 复制到 “/usr/bin” 下,并赋予其执行权限:

	sudo chmod 777 /usr/bin/class-dump
	
输入 class-dump，即可查看他的一些基本参数。
	
注意：Mac OS X 10.11内核下引入了Rootless机制,即使root用户无法对某些目录有写和执行权限,关闭Rootless权限的方法:

- 开机按住Command + R键，以Recovery分区启动
- 或选择图形化操作，在Security Configuration中关闭Enforce System Integrity Protection
- 重新启动电脑进入普通模式即可。

##实战
1. 新建一个程序，打包出ipa，这里不要从商店里下载，因为从商店下载的程序有壳，无法直接访问。
2. 将ipa文件解压，进入到程序的目录，使用Xcode自带的plutil工具查看Info.plist中的 “CFBundleExecutable” 字段，如下：

          plutil -p Info.plist | grep CFBundleExecutable
          出现App的可执行文件。
          "CFBundleExecutable" => "IOSAPP"

3. 使用class-dump来分析APP：class-dump -S -s -H IOSAPP -o ~/Desktop/reveal

部分参数说明：

* -S：sort methods by name
* -s：sort classes and categories by name、
* -H：generate header files in current directory, or directory specified with -o
-o：output directory used for -H

除了一些参数类型被改成了 id ，参数明用了 arg1、arg2表示之外，几乎差不多。

是不是简单容易，赶紧试一试吧~