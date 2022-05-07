---
title: Handler-内存泄漏解决方案
date: 2022-05-04 15:50:46
categories:
 - Android
tags:
 - Handler
cover: https://s1.ax1x.com/2022/05/04/OE3oQJ.jpg
---

# 前言
  * Android开发中，内存泄漏是经常发生的，今天来详细聊一聊Handler使用过程中产生内存泄漏的场景以及对应的解决方案。

# 目录
[![OEUUXD.png](https://s1.ax1x.com/2022/05/04/OEUUXD.png)](https://s1.ax1x.com/2022/05/04/OEUUXD.png)

# 问题描述
* Android中使用Handler导致内存泄漏原因分析。当一个对象已经不再被使用时，本该被回收但却因为有另外一个正在使用的对象持有它的引用从而导致它不能被回收，导致了内存泄漏。

# 原因详解
* 在Handler消息队列还有未处理的消息 / 正在处理消息时，消息队列中的Message持有Handler实例的引用，由于Handler = 非静态内部类 & 匿名内部类又默认持有外部类的引用。如图

[![示例1.png](https://s1.ax1x.com/2022/05/04/OE88YT.png)](https://s1.ax1x.com/2022/05/04/OE88YT.png)

# 解决方案
* 静态内部类+弱引用
* 静态内部类 不默认持有外部类的引用，从而使得 “未被处理&正处理的消息 -> Handler实例 -> 外部类”的引用关系不复存在。
* 使用WeakReference弱引用持有Activity实例的原因：弱引用的对象拥有短暂的生命周期。在垃圾回收器线程扫描时，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存

* 工具类封装
``` java
package com.linked.business.handler;

import android.os.Handler;
import android.os.Looper;
import android.os.Message;

import java.lang.ref.WeakReference;

public class ArrestLeakHandler extends Handler {
    private final WeakReference<MessageHandler> reference;

    public ArrestLeakHandler(MessageHandler messageHandler) {
        reference = new WeakReference<>(messageHandler);
    }

    public ArrestLeakHandler(MessageHandler messageHandler, Looper looper) {
        super(looper);
        reference = new WeakReference<>(messageHandler);
    }

    /**
     * 复写handlerMessage 做更新UI的操作
     */
    @Override
    public void handleMessage(Message msg) {
        MessageHandler messageHandler = reference.get();
        if (messageHandler != null) {
            messageHandler.handleMessage(msg);
        }
    }

    public interface MessageHandler {
        void handleMessage(Message msg);
    }
}

```
* 具体 java
```java

public class HandlerLeakActivity extends BaseActivity<ActivtyHandlerLeakBinding, BasePresenter> implements ArrestLeakHandler.MessageHandler {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        initViews();
        initHandler();
    }

    @Override
    protected void initViews() {
        //发送消息A
        vBinding.tvA.setOnClickListener(view -> {
            Message msg = new Message();
            msg.obj = "我是消息A";
            msg.what = 1;
            mHandler.sendMessage(msg);
        });

        //发送消息B
        vBinding.tvB.setOnClickListener(view -> {
            Message msg = new Message();
            msg.obj = "我是消息B";
            msg.what = 2;
            mHandler.sendMessage(msg);
        });
    }

    @Override
    protected BasePresenter initPresenter() {
        return null;
    }

    @Override
    public void onBackPressed() {
        super.onBackPressed();
        finish();
    }

    static ArrestLeakHandler mHandler, mMainHandler;
    HandlerThread mHandlerThread;

    private void initHandler() {
        //①创建子线程
        mHandlerThread = new HandlerThread("handlerThread");
        //②开启线程
        mHandlerThread.start();
        //③建立与主线程关联Handler
        mMainHandler = new ArrestLeakHandler(this);
        mHandler = new ArrestLeakHandler(this, mHandlerThread.getLooper());
    }

    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case 1:
            case 2:
                String str = (String) msg.obj;
                try {
                    Thread.sleep(1500);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                LogUtils.v("Handler Message--->" + str);
                mMainHandler.post(() -> vBinding.tvShowMessage.setText(str));
                break;
            default:
                break;
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mHandler.removeCallbacksAndMessages(null);
        mHandler = null;
    }
}

```
* 注:当外部类（Activity为例）结束生命周期时（onDestroy），应当清除Handler消息队列里的所有消息（调用removeCallbacksAndMessages(null)）方法。
```java
@Override
    protected void onDestroy() {
        super.onDestroy();
        mHandler.removeCallbacksAndMessages(null);
        mHandler = null;
    }
```
# 总结
* 当Handler消息队列中存在未处理或正在处理的消息时，并且存在引用关系:未被处理&正处理的消息->Handler实例->外部类
  当Handler的生命周期>外部类的生命周期时（当外部类需销毁时(Handler消息队列中还有未处理或正在处理消息)),将使得外部类无法被垃圾回收器(GC)回收，既造成内存泄露。

### <font color=#9dc2af>如果觉得此文章对你有所帮助，👍🏻请点个赞👍🏻</font>
