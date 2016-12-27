---
layout: post
title: iOS 面试题
description: ios
headline: ios
category: iOS
tags: [面试题]
image: 
comments: true
mathjax:
---

<section id="table-of-contents" class="toc">
  <header>
    <h1>Features</h1>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

## runtime 如何实现 weak 属性

### weak特点
要实现 weak 属性，首先要搞清楚 weak 属性的特点：

> weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同 assign 类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。

### runtime 实现weak自动置nil

那么 runtime 如何实现 weak 变量的自动置nil？

> runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc，假如 weak 指向的对象内存地址是a，那么就会以a为键， 在这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。

### 对象的内存销毁时间表

对象的内存销毁时间表，分四个步骤：

1. 调用 -release ：引用计数变为零
     * 对象正在被销毁，生命周期即将结束.
     * 不能再有新的 __weak 弱引用， 否则将指向 nil.
     * 调用 [self dealloc] 
2. 子类 调用 -dealloc
     * 继承关系中最底层的子类 在调用 -dealloc
     * 如果是 MRC 代码 则会手动释放实例变量们（iVars）
     * 继承关系中每一层的父类 都在调用 -dealloc
3. NSObject 调 -dealloc
     * 只做一件事：调用 Objective-C runtime 中的 object_dispose() 方法
4. 调用 object_dispose()
     * 为 C++ 的实例变量们（iVars）调用 destructors 
     * 为 ARC 状态下的 实例变量们（iVars） 调用 -release 
     * 解除所有使用 runtime Associate方法关联的对象
     * 解除所有 __weak 引用
     * 调用 free()

### 验证

生成weak属性的对象， 并指向一个对象，当对象销毁之后，指向该对象的weak属性的对象也会随之销毁。

<pre class="sunlight-highlight-objective-c">
__weak id weakObj1 = nil;
__weak id weakObj2 = nil;
{
    id obj = [[NSObject alloc] init]; // 引用计数会增加，这里不会是weak的
    weakObj1 = obj;
    weakObj2 = obj;
    NSLog(@"obj: %@", obj);
    NSLog(@"weakObj1: %@", weakObj1);
    NSLog(@"weakObj2: %@", weakObj2);
}
NSLog(@"weakObj1: %@", weakObj1);
NSLog(@"weakObj2: %@", weakObj2);
</pre>

输出为

<pre class="sunlight-highlight-objective-c">
obj: <NSObject: 0x60800000bdc0>
weakObj1: <NSObject: 0x60800000bdc0>
weakObj2: <NSObject: 0x60800000bdc0>
weakObj1: (null)
weakObj2: (null)
</pre>

因为 weak 本身是非拥有的，所有如果是把weakObj1设置为nil的话，是不会对其他的造成影响的。

 参考：
 
 <a href="https://github.com/ChenYilong/iOSInterviewQuestions/blob/master/01%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88/%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88%EF%BC%88%E4%B8%8A%EF%BC%89.md" style="color:blue">iOSInterviewQuestions</a>