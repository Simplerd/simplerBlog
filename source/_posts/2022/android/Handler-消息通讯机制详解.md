---
title: Handler-消息通讯机制详解
date: 2022-04-19 16:25:10
categories:
 - Android
tags:
 - Handler
cover: https://s1.ax1x.com/2022/03/23/q3s3Oe.jpg
---

# 前言
* Android开发中，Handler媒介机制十分常用，今天我们来详细聊一聊Handler通讯机制是如何实现的。

# 目录

[![Omb4mt.png](https://s1.ax1x.com/2022/05/05/Omb4mt.png)](https://s1.ax1x.com/2022/05/05/Omb4mt.png)

# 使用场景
在多线程的应用场景中，将工作线程中需更新UI的操作信息 传递到 UI主线程，从而实现工作线程对UI线程的更新，最终实现异步消息的处理。

# 概念描述
Handler、Message、MessageQueue、Looper相关概念如下图。
<!--MarkDown 不支持单元格合并，即使用Html -->
<table>
<!--this is title-->
    <tr align = "center" bgcolor = "#91bdb6">
        <td>概念</td>
        <td>定义</td>
        <td>作用</td>
        <!-- <td>备注</td> -->
    </tr>
    <!--this is Main-->
    <tr align="center">
        <td>主线程<br/><font color = "#91bdb6">( UI线程、Main Thread )</font></td>
        <td>系统首次启动，会创建主线程</td>
        <td>处理UI相关操作<br/><font color = "#91bdb6">( 如：更新UI )</font></td>
        <!-- <td rowspan="2">主线程与子线程<br/>通讯媒介Handler</td> -->
    </tr>
      <!--this is Thread-->
    <tr align="center">
        <td>子线程<br/><font color = "#91bdb6">（ 工作线程 ）</font></td>
        <td>人为开启的线程</td>
        <td>执行耗时任务<br/><font color = "#91bdb6">( 如 网络请求、数据加载等 )</font></td>
    </tr>
      <!--this is Message-->
    <tr align="center">
      <td>消息<br/><font color = "#91bdb6">（ Message ）</font></td>
      <td>线程间通讯数据单元<br/><font color = "#91bdb6">（handler接收&处理消息对象）</font></td>
      <td>存储业务操作信息<br/><font color = "#91bdb6">（ Message ）</font></td>
    </tr>
      <!--this is Message Queue-->
    <tr align="center">
        <td>消息队列<br/><font color = "#91bdb6">（ Message Queue ）</font></td>
        <td>数据结构<br/><font color = "#91bdb6">（ 以时间顺序排列 ）</font></td>
        <td>存储Handler发送的消息<br/><font color = "#91bdb6">（ Message ）</font></td>
    </tr>
      <!--this is Handler-->
    <tr align="center">
        <td>处理着<br/><font color = "#91bdb6">（ Handler ）</font></td>
        <td>主线程与子线程通讯的媒介<br/>线程消息的处理者</td>
        <td>将消息( Message ）添加到<br/>消息队列( Message Queue )中<br/> 处理循环器（ Looper ）分派的消息</td>
    </tr>
    <tr align="center">
        <td>循环器<br/><font color = "#91bdb6">（ Looper ）</font></td>
        <td>消息队列( Message Queue )<br/>与处理者 （ Handler ）<br/>通讯媒介</td>
        <td>消息循环 即：<br/><font color = "#91bdb6"> 消息获取：循环取出消息队列<br/>（ Message Queue ）中的<br/>消息（ Message ）<br/> 消息分发：将取出来的消息<br/>（ Message ）发送给对应的<br/>处理者（ Handler ）</font></td>
    </tr>
</table>

# 机制说明
Handler消息传递机制:多个线程并发更新UI时，保证线程安全
[![示例图.png](https://s1.ax1x.com/2022/05/06/OmxK4P.png)](https://s1.ax1x.com/2022/05/06/OmxK4P.png)
注：[图解Java线程安全](https://juejin.cn/post/6844903890224152584)

# 原理解析
### 相关概念
  * 异步通信( 主线程中创建 )

    >> 处理者（ Handler ）对象
    >>> 注：Handler会自定绑定主线程中的Looper&MessageQueue

    >> 处理器（ Looper ）对象
      >>> 注：Looper & MessageQueue都是在主线程中

    >> 消息队列（ Message Queue ）对象
      >>>注： 创建MessageQueue后，Looper会自动进入循环状态，有消息则循环取，没有，则阻塞状态，发现消息后会被唤醒继续循环读取。

  * 消息入队列
  > 工作线程中通过Handler的sendMessage()方法发送消息（ Message ）到消息队列（ MessageQueue ）中，这是消息完成入队列操作。
  >> 注：此消息内容就是工作线程对UI主线程的操作

  * 消息循环
  > 第一步：消息出队列：Looper循环取出消息队列（ MessageQueue ）中的消息（ Message ）交给第二步消息分发。
  > 第二步：消息分发：Looper将取出来的消息（ Message ）发送给创建消息的处理者（ Handler ）
  >> 注：循环器( Looper )循环取消息时，若发现消息队列（ MessageQueue ）中为空，则会线程阻塞，待又消息时再次被唤醒继续循环读取。

  * 消息处理
  >处理者( Handler ) 在接收处理器（ Looper ）发送过来的消息（ Message ）会进行UI操作。

[![示意图.png](https://s1.ax1x.com/2022/05/06/OuFqzR.png)](https://s1.ax1x.com/2022/05/06/OuFqzR.png)

# 总结
Android开发中，UI操作在主线程中进行，但是主线程又不能进行耗时操作，否则会阻塞线程，产生我们常见的ANR异常，所以常常把耗时操作放到其它子线程进行。如果在子线程中需要更新UI，一般是通过 Handler发送消息( message ），主线程接收消息并且进行相应的处理。以上就是Handler通讯消息传递机制的详解。

### <font color=#9dc2af>如果觉得此文章对你有所帮助，👍🏻请点个赞👍🏻</font>
