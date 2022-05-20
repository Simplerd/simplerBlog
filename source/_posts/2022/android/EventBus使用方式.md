---
title: EventBus使用方式
date: 2022-05-09 11:53:31
categories:
 - Android
tags:
 - EventBus
cover: https://s1.ax1x.com/2022/03/23/q3s3Oe.jpg
---
# 前言
* Android开发中 EventBus的使用
# 目录
[![示意图.png](https://s1.ax1x.com/2022/05/09/OGINDg.png)](https://s1.ax1x.com/2022/05/09/OGINDg.png)
# 简介
* 一套 <font color= "#c85863" >Android / Java</font> 事件订阅 / 发布框架，由greenrobot团队开源，它的出现简化了应用程序内各个组件之间进行通信的复杂度，尤其是碎片之间进行通信的问题。

# 使用方式
## 引入依赖
```java
    eventbus : 'org.greenrobot:eventbus:3.2.0'
```
## 注册、注销

```java
    @Override
    protected void onStart() {
        super.onStart();
        EventBus.getDefault().register(this);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
         EventBus.getDefault().unregister(this);
    }
```
## 订阅者
```java 
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onUpDateEvent(BaseMessageEvent messageEvent ){
        //do something
    }
```

## 发布事件

### 普通事件
```java 
    EventBus.getDefault().post(new BaseMessageEvent(""))
```
### 粘性事件
* 黏性事件，就是指发送了该事件之后再订阅者依然能够接收到的事件。使用黏性事件的时候有两个地方需要做些修改。分别在发布和订阅中来使用。
#### 发布
```java 
    EventBus.getDefault().postSticky(new BaseMessageEvent(""));
```
#### 订阅
```java 
    @Subscribe(threadMode = ThreadMode.MAIN , sticky = true)
    public void onUpDateEvent(BaseMessageEvent messageEvent ){
        //do something
    }
```
#### 获取事件
```java
    EventBus.getDefault().getStickyEvent(BaseMessageEvent.class);
```
#### 移除事件
```java
    EventBus.getDefault().removeStickyEvent(BaseMessageEvent.class);
```

## 优先级
* 即priority。它用来指定订阅方法的优先级，是一个整数类型的值，默认是0，值越大表示优先级越大。在某个事件被发布出来的时候，优先级较高的订阅方法会首先接受到事件。
* 特别注意：
    *  订阅方法，需要与threadMode一致，并且优先级略高
    *  优先级不会影响具有不同ThreadModes的订阅者的传递顺序

```java
    @Subscribe(threadMode = ThreadMode.POSTING , sticky = true, priority = 1)
    public void onUpDateEvent(BaseMessageEvent messageEvent ){
        //do something
    }

```
# 取消事件传递
* 事件的优先级越高接收的数据最快，所以当优先级不想分发事件给低级别的事件时，可以使用cancelEventDelivery （Object event）取消事件传递

* 注：当优先级更高的想取消事件传递时，只有当threadMode = ThreadMode.POSTING处于此状态才能取消事件传递有效。其他线程模型属性不行

# 总结
EventBus基本使用方式就介绍到这里，EventBus源码分析请看：{% post_link EventBus原理分析 EventBus源码分析 %}


