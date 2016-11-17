---
layout: post
type: photo
title: 使用Reveal分析app结构
description: Reveal是一个很强大的UI分析工具，用来查看app的UI布局非常方便。不仅能够分析自己的app，任意其他的app UI也一览无遗。
category: development tools
tags: [development tools]
photo: picture-1.jpg
image:
	feature: reveal-1.jpg
comments: true
mathjax: 
---

##使用Reveal分析自己的app		
将Reveal的静态库直接导入iOS程序是最快捷、简单的方式。

1、 打开Reveal，选择Help → Show Reveal Library in Finder。会直接打开一个名为iOS-Libraries的新的Finder窗口。

![Reveal](/images/reveal/reveal_1_1.jpg)

2、 将*Reveal.framework*导入工程，在**Link Binary With Libraries**中将*Reveal.framework*移除。

![Reveal](/images/reveal/reveal_1_2.jpg)

3、 选择**Build Settings**，搜索**Linking**，选择**Other Linker Flags**，在**Debug**模式下添加*-ObjC -lz -framework Reveal*。

![Reveal](/images/reveal/reveal_1_3.jpg)

4、 运行程序，打开Reveal，就会有选择模拟器的程序了。若要是想真机的话，需要手机和电脑在同一WIFI下。

![Reveal](/images/reveal/reveal_1_4.jpg)

##进阶：不修改Xcode工程并加载Reveal
**此方法仅适用于在iOS模拟器上运行的应用**
		
通过不修改Xcode工程文件来加载Reveal的方式，您可以检视任何一个您正在开发的iOS应用，而不需要对这些应用的工程做任何修改。另一个好处就是，您不需要再担心，犯下一不小心将Reveal库连接到应用中发布了的错误。

1、 打开iOS工程，选择 **View → Navigators → Show Breakpoint Navigator**。

![Reveal](/images/reveal/reveal_2_1.jpg)

2、 在面板左下角，点击 + 按钮并选择**Add Symbolic Breakpoint**。
![Reveal](/images/reveal/reveal_2_2.jpg)
3、 在 **Symbol** 输入区内输入 **UIApplicationMain** 。

4、 点击 **Add Action** 按钮, 确认 **Action** 被设置为 **Debugger Command**。

5、 将以下内容拷贝到 Action 的输入区内:		

<pre>
expr (Class)NSClassFromString(@"IBARevealLoader") == nil ? (void *)dlopen("/Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/libReveal.dylib", 0x2) : ((void*)0)
</pre>
		
 *注意: 请确认Reveal.app的路径信息符合您Mac的实际位置。*
 
6、 选中 **Automatically continue after evaluating actions** 选项。
![Reveal](/images/reveal/reveal_2_3.jpg)
7、 右击刚才新创建的断点，选择 **Move Breakpoint To → User**.
*可以像其他断点一样，禁用或启用此断点。用户级别断点在所有的Xcode工程中都可以使用。*
![Reveal](/images/reveal/reveal_2_4.jpg)
8、 在iOS模拟器上构建并运行您的应用。如果一切正常运行，请切换到Reveal应用，此时您的应用应会出现在应用选择器的下拉列表当中。

9、 选中您的应用，确认可以看到此时正在模拟器中运行的应用界面截图。

##Reveal查看任意app的高级技巧
1、 越狱后iPhone上会自行安装上Cydia商店，打开Cydia，搜索并安装**Reveal loader** ，安装完成后点击重启springboard。

![Reveal](/images/reveal/reveal_3_1.jpg)

2、 在系统设置中找到Reveal，点击**Reveal - Enabled Applications**，选择你要监视的app即可。

![Reveal](/images/reveal/reveal_3_2.jpg)

![Reveal](/images/reveal/reveal_3_3.jpg)

3、 首先保证iPhone和Mac在同一局域网（WiFi）中，在iPhone中运行你要监视的app.

4、 如果app已经运行，需在后台杀死进程重新打开，保持app在前台，然后在Mac中打开Reveal，点击左上角的No Connection，然后选择即可。
