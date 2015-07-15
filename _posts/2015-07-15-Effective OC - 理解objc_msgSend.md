---
layout: post
title: Effective OC - 理解objc_msgSend
categories: 技术

---

在Objective-C中，向对象传递消息是通过动态绑定来决定需要调用的方法的。在底层，所有的方法都是普通的C函数。
给对象发消息大概是这个样子：

{% highlight objective-c %}

id returnValue = [someObject messageName:parameter];

{% endhighlight objective-c %}

底层对应的C函数原型如下：

{% highlight objective-c %}

void objc_msgSend(id self,SEL cmd,...)

{% endhighlight objective-c %}

其具体的消息转换为：

{% highlight objective-c %}

id returnValue = objc_msgSend(someObject,@selector(messageName:),parameter)

{% endhighlight objective-c %}

当向一个对象发送消息时，objc_msgSend方法根据对象的isa指针找到对象的类，然后在类的调度表(dispatch table)中查找selector；若无法找到，则继续通过指向父类的指针找到父类，并在父类的调度表中查找，直至追踪到NSObject。一旦找到selector,objc_msgSend根据调度表中的内存地址调用该实现，通过这种方式实现方法的动态绑定；若在NSObject中仍找不到selector，则执行消息转发(message forwarding).

为了保证消息发送与执行的效率，objc_msgSend会将匹配结果缓存在快速映射表(fast map)里，没个类都有这样一块缓存，之后再次发送相同的消息则执行起来就变快了。