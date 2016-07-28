---
layout: post
title: ios单例模式
date: 2016-07-28
---

单例 保证一个对象只实例化一次 全局使用的都是同一个对象

- 一是某个类只能有一个实例；
- 二是它必须自行创建这个实例；
- 三是它必须自行向整个系统提供这个实例。

第一种写法：
<pre> +(instancetype)shareInstance{  
static PXYGuidePageHelper *instance;   
       	@synchronized(self) {  
            if (instance == nil) {  
                instance = [PXYGuidePageHelper new];  
            }   
        }   
    return instance;  
}   
</pre>

第二种写法：
<pre> +(instancetype)shareInstance{
    static dispatch_once_t onceToken;
    static PXYGuidePageHelper *instance;
    dispatch_once(&onceToken, ^{
        instance = [PXYGuidePageHelper new];
    });
    return instance;
}
</pre>

首先说下第一种写法：
单例实例是全局使用的，因此要要定义成全局变量 用static修饰 static介绍
@synchronized这个指令是解决多个线程同时执行同一个代码块 ，@synchronized相当于给这个代码块加锁（防止死锁）
这里判断当前对象时候存在，不存在创建，存在则返回该对象。

第二种写法：
dispatch_once这个就是保证只执行一次 所以这里确保 该实例只创建一次

两者区别：
第一种在每次执行shareInstance方法是都会加一次锁，然后在代码块里面判断 if (instance == nil) 这个来决定是否实例化，这里每次都会有开销。

>“实际上，如果你去看这个函数所在的头文件，你会发现目前它的实现其实是一个宏，进行了内联的初始化测试，这意味着通常情况下，你不用付出函数调用的负载代价，并且会有更少的同步控制负载。”

这样子分析的GCD创建单例更加优雅点。

－－－－  
第一种相当于是 懒汉式单例类 双检锁写法

对于第一种的每次都要加锁的写法，可以使用双检锁写法来提高效率。
<pre>+(instancetype)shareInstance{
    static PXYGuidePageHelper *instance;
    if (instance == nil) {
        @synchronized(self) {
            if (instance == nil) {
                instance = [PXYGuidePageHelper new];
            }
        }
    }
    return instance;
}</pre>

这样子就能保证只有在第一次进行加锁开销。

还有一种 饿汉式单例
是在程序一启动就实例化 +(void)load函数里面实现，然后在allocWithZone进行加锁判空操作，这样子无论你是否用特定的方法获取实例，都会返回同一个对象。

以上