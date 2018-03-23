# runtime-学习札记

## 😜扯下淡
runtime这一块的内容一直是我在OC学习上的短板。
虽然看过一些资料和相关代码，但所知道的内容也仅仅浮于表面。
前段时间，面试时，被问到这块的内容，也老实交代这一块掌握的不多。很尴尬😰😰😰.

接下来会在这篇文章中陆续记录学习runtime的成果和体会......

## 前言

runtime的源码和官方api地址：

* 源码：[objc-runtime](https://github.com/RetVal/objc-runtime)
* 官方api：[Objective-C Runtime Reference](https://developer.apple.com/documentation/objectivec/objective_c_runtime?language=objc)

所谓的runtime，就是运行时。。。

我们都知道oc是动态语言，所谓的动态语言就是指程序在运行时可以改变其结构：新的函数可以被引进，已有的函数可以被删除等在结构上的变化的语言。

因为它可以在程序执行过程中对类或者变量等等做操作，而不是代码写完了，程序就定型了。

而iOS中的runtime就是可以实现语言动态的一组API.
你可以理解：它就仅仅是一组API而已。

只是这组API看起来比oc长得不太一样，见的少了会觉得它们比较混乱，乱起八糟的。

## OC的简介
Objective-C是一门基于对象的动态语言.
它里面所有的继承自NSObject的类本身以及类所实例化的对象都是对象，好像有点儿绕，大意就是这是一门基于对象的语言，就连类自己也是一个对象，程序运行起来以后类本身也会被初始化，也占有内存空间。

## 运行时简介

Runtime基本上是用C和汇编写的，这个库使得C语言有了面向对象的能力。

在Runtime中，对象可以用C语言中的结构体表示，而方法可以用C函数来实现，另外再加上了一些额外的特性。这些结构体和函数被runtime函数封装后，让OC的面向对象编程变为可能。

方法的最终执行代码：当程序执行[object doSomething]时，会向消息接收者(object)发送一条消息(doSomething)，runtime会根据消息接收者是否能响应该消息而做出不同的反应。

runtime的所有知识基本都围绕两个中心：

* 类的各个方面的动态配置
* 消息传递

## 消息传递
`Objective-C`是一门动态的语言，当我们在某个对象上调用方法是，`Objective—C`里面叫做`消息传递`，和C语言的函数调用在本质上就不一样，C语言使用`静态绑定`，在编译器就能决定运行时应该调用的函数：

```c
//代码一
void printHello() {
    printf("Hello, world!\n");
}

void printGoodbye() {
    printf("Goodbye, world!\n");
}

void doSomeThing(int type) {
    if (type == 0) {
        printHello();
    } else {
        printGoodbye();
    }
}
```

## 类的本质

要动态配置类就需要知道类的本质是什么，我们可以从<objc/runtime.h>里面看到类的定义：

类是由Class类型来表示的，它实际上是一个指
向objc_class结构体的指针。

```c
typedef struct object_class *Class
```

定义如下：

```c
struct object_class{
    Class isa OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
     Class super_class                        OBJC2_UNAVAILABLE;  // 父类
     const char *name                         OBJC2_UNAVAILABLE;  // 类名
     long version                             OBJC2_UNAVAILABLE;  // 类的版本信息，默认为0
     long info                                OBJC2_UNAVAILABLE;  // 类信息，供运行期使用的一些位标识
     long instance_size                       OBJC2_UNAVAILABLE;  // 该类的实例变量大小
     struct objc_ivar_list *ivars             OBJC2_UNAVAILABLE;  // 该类的成员变量链表
     struct objc_method_list *methodLists     OBJC2_UNAVAILABLE;  // 方法定义的链表
     struct objc_cache *cache                 OBJC2_UNAVAILABLE;  // 方法缓存
     struct objc_protocol_list *protocols     OBJC2_UNAVAILABLE;  // 协议链表
#endif
}OBJC2_UNAVAILABLE;
```




