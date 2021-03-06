---
title: GoF设计模式
category: 
- UML与模式
date: 2016-05-15
---

### 适配器

名称: 适配器
问题: 如何解决不相兼容的接口问题,或者如何为具有不同接口的类似构件提供稳定的接口
解决方案: 通过中介适配器对象,将构件的原有接口转换为其他接口

### 单实例类

名称: 单实例类
问题; 只有唯一实例的类即为"单实例类".对象需要全局可见性和单点访问
解决方案: 对类定义静态方法用以返回单实例

通常用于服务类

### 策略

名称:  策略
问题: 如何设计变化但相关的算法? 如何设计才能是这些算法能有可变更的能力?
解决方案: 在单独的类中分别定义每种算法,并且使其具有共同的接口.


### 组合

名称: 组合
问题: 如何能够像处理非组合对象一样,多态地处理一组对象或者具有组合结构的对象呢?
解决方案: 定义组合和原子对象的类,使它们实现相同的接口

### 外观

名称: 外观
问题: 对一组完全不同的接口或者实现需要提供公共同意的接口.可能会与子系统内部的大量事物产生耦合,或者子系统的实现可能会改变.怎么办?
解决方案: 对子系统定义唯一的接触点-使用外观对象封装子系统.该外观对象提供了唯一和统一的接口,并负责与子系统构件进行协作.

### 观察者

名称: 观察者(Observer) (发布-订阅(Publish-Subscribe))
问题: 类型的订阅者对象关注于发布者对象的状态变化或者事件,并且想要在发布者产生事件时以自己独特的方式作出发应.此外,发布者想要保持与订阅者的低耦合.如何对此进行设计呢?
解决方案: 定义订阅者或者监听者接头,订阅者实现此接口.发布者可以动态注册关注某事件的订阅者,并在事件发生时通知它们.c
