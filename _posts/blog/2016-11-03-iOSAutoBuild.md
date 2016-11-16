---
layout:     post
title:      iOS自动化编译打包
category:   blog
description: 从xcodebuild到shenzhen，再到Jenkins，完美演绎自动化操作。

---
##xcodebuild自动构建命令

确保项目证书等配置都没问题，可以完美运行。

###简介
首先说明下使用文档：

```
man xcodebuild
```
基本上现在的包管理都是以pod来的，也就是以workspace的形式，所以基本的形式为：

```
xcodebuild [-project projectname] [-target targetname ...] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [buildaction ...] [setting=value ...] [-userdefault=value ...]
```
* target，可以通过 xcodebuild -list 查看
* configrtion，也可以通过 xcodebuild -list 查看
* sdk，可用 xcodebuild -showsdks，一般默认就行

可以查看项目配置选项:

```
xcodebuild -target Demo -configuration Debug -showBuildSettings
```
###构建
基本的构建命令：

```
xcodebuild -workspace Demo.xcworkspace -scheme Demo -configuration Debug -sdk iphoneos10.1
```
命令运行成功后会提示 `** BUILD SUCCEEDED **`，一般会在项目目录下生成build文件夹,可以在里面看到你的生成的包。

对于`workspace`的形式来说，基本上也差不多：

```
xcodebuild -workspace Demo.xcworkspace -scheme Demo -configuration Debug -sdk iphoneos10.1
```
> 好像对workspace构建后不会在项目目录下生成build文件夹，可以在你的命令后面添加SYMROOT=buildDir指定一个build文件夹）。

###生成ipa文件
生成文件的命令是`xrun`：

```
xcrun -sdk iphoneos -v PackageApplication ./build/Release-iphoneos/Demo.app -o ~/Desktop/Demo.ipa
```
打包成功后，会在桌面找到你的ipa。

##利用 shenzhen 进行打包
利用github上一个开源项目：[shenzhen](https://github.com/nomad/shenzhen) 可以在命令行为ios项目进行打包并发布。
具体安装步骤如下：

```
gem install shenzhen
```

如果安装过程出现错误有可能是ruby的源找不到，可以到 [RubyGems 镜像](https://ruby.taobao.org/) 改变ruby源。

如果还是出现问题可以更新下gem即可（sudo gem update）。

一切准备完毕就能在控制台上运行ipa命令了：

```
$ ipa 

Build and distribute iOS apps (.ipa files)

Commands:
build                    Create a new .ipa file for your app
distribute:crashlytics   Distribute an .ipa file over Crashlytics
distribute:deploygate    Distribute an .ipa file over deploygate
distribute:fir           Distribute an .ipa file over fir.im
distribute:ftp           Distribute an .ipa file over FTP
distribute:hockeyapp     Distribute an .ipa file over HockeyApp
distribute:itunesconnect Upload an .ipa file to iTunes Connect
distribute:pgyer         Distribute an .ipa file over Pgyer
distribute:rivierabuild  Distribute an .ipa file over RivieraBuild
distribute:s3            Distribute an .ipa file over Amazon S3
distribute:testfairy     Distribute an .ipa file over TestFairy
help                     Display global or [command] help documentation
info                     Show mobile provisioning information about an .ipa file
```
可以看出通过bulid参数就能创建ipa文件，比如输入命令：

```
ipa build
```
会直接在当前目录下生成ipa文件以及dSYM文件。

如果你的工程项目有很多targets，则ipa bulid命令会列出现在所有targets，我们可以选择一个进行打包。

如简单的打包蒲公英事例：

```
ipa distribute:pgyer -u USER_KEY -a APP_KEY
```

iTunes Connect Distribution:

```
ipa distribute:itunesconnect -a me@email.com -p myitunesconnectpassword -i appleid --upload
```

##Jenkins自动化
###安装
在 Mac 环境下，我们需要先安装 JDK，在[Jenkins 的官网](https://jenkins.io/)下载最新的 war 包。下载完成后，打开终端，进入到 war 包所在目录，执行以下命令：

```
java -jar jenkins.war --httpPort=8080
#或简单的写法
java -jar jenkins.war
```

待Jenkins启动后，在浏览器中输出一下地址：
http://localhost:8080

这样就打开Jenkins管理页面了。

基本界面如下：

![Reveal](/images/blog/iOSAutoBuild/build_3_1.png)

###创建项目
点击左上角的新建，或是店家开始创建一个新任务，出现下面的页面:

![Reveal](/images/blog/iOSAutoBuild/build_3_2.png)

这里输入的名字为Demo，并选择 `构建一个自由风格的软件项目`，点击OK进入到下一页面：

![Reveal](/images/blog/iOSAutoBuild/build_3_3.png)

其中这里在General中，点击高级，先使用本地项目做测试：

![Reveal](/images/blog/iOSAutoBuild/build_3_4.png)

源码管理暂选None，构建触发器和构建环境不需要选择：

![Reveal](/images/blog/iOSAutoBuild/build_3_5.png)

构建，选择shell形式，使用`shenzhen`来构建并直接上传到蒲公英:

![Reveal](/images/blog/iOSAutoBuild/build_3_6.png)
 
![Reveal](/images/blog/iOSAutoBuild/build_3_7.png)

其中，`USER_KEY` 和 `API_KEY` 可以在蒲公英的「账户设置」中找到，之后进行相应替换。

构建后的操作我们也不需要，直接点击保存。

###构建

保存之后进入到项目工作目录，点击立即构建：

![Reveal](/images/blog/iOSAutoBuild/build_3_8.png)

会在构建历史中显示构建结构，点击进入查看：

![Reveal](/images/blog/iOSAutoBuild/build_3_9.png)

点击 `Console Output` 查看日志信息:

![Reveal](/images/blog/iOSAutoBuild/build_3_10.png)

会有一堆信息，成功的话会提示去蒲公英查看。

进入到蒲公英后台，会发现我们的应用已经发布上去，可以进行测试了。

###配置远程仓库
首先配置SSH:

![Reveal](/images/blog/iOSAutoBuild/build_3_11.png)

创建global的类型：

![Reveal](/images/blog/iOSAutoBuild/build_3_12.png)

进去后点击左侧的 ` Add Credentials`:

![Reveal](/images/blog/iOSAutoBuild/build_3_13.png)

选择SSH类型，输入自己的用户名，`Private Key` 直接从 `~/.ssh` 目录下读取就好。

工程的配置，跟本地的区别就是不需要配置自定义的工作空间，同时选择源码管理中的`Git`，填写对应的地址信息：

![Reveal](/images/blog/iOSAutoBuild/build_3_14.png)

然后其他的构建、查看过程都一致。

到蒲公英上检查，果然存在。完美！

###便捷设置
以上面的方式运行的`Jenkins`的，命令行是不能关闭的，为了方便的话，需要设置在后台运行：

```
nohup java -jar jenkins.war &
```
将命令写入到sh文件中，比如就叫 start.sh，运行的时候直接跑脚本就好，附上文件内容：

```
#!/bin/sh
nohup java -jar /Users/home/Desktop/jenkinsWorkspace/jenkins/jenkins.war &
```

同样的，关闭命令也可以直接使用，不过在使用关闭之前，需要下载个 `jenkins-cli.jar`文件：

> 首页 -> 系统管理 -> Jenkins CLI

里面同样包含好多其他命令，可以根据自己需要来调试。

设置关闭 `Jenkins` 的脚本：

```
#!/bin/sh
java -jar /Users/home/Documents/jenkins/jenkins-cli.jar -s http://localhost:8080/ shutdown

```

别忘记修改为自己的路径。

一般的命令可以直接在网址上体现出来，比如重启: [http://localhost:8080/restart](http://localhost:8080/restart)

上面既然用了iOS的打包，所以脚本感觉也是用swift来写也是比较配套的，在这里就不贴出来了，喜欢研究的童鞋就google一下~，我将之命名为`begin.swift`和`end.swift`，恩，感觉还汗不错的


##参考：
[iOS 自动构建命令——xcodebuild](http://www.jianshu.com/p/3f43370437d2)

[分享查询网](http://www.fx114.net/qa-58-777704.aspx)

[用Jenkins给iOS自动打包](http://www.jianshu.com/p/9934a678c17c)

[蒲公英](https://www.pgyer.com/doc/view/jenkins_ios)