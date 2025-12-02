
# Java部分
## 反射
  * Jav反射-高级特性 - 已完结
 
## 链表|数据结构
  * LinedHashMap原理分析  已完结
  * LruCache原理分析  已完结
  * Java-链表（单链表、循环链表、双链表）已完结

## 注解
  * Java注解之秒懂 - 进行中
   default方法 jdk1.8之前接口中只能定义方法名，在jdk1.8 可以实现具体代码 例如：default void test(){ todo...具体的业务代码}

## 动态代理
  * java 动态代理 - 未开始
  
## 设计模式
  * 六大原则：
    * 单一原则：未开始
    * 开闭原则：未开始
    * 里氏替换原则：未开始
    * 接口隔离原则：未开始
    * 迪米特原则：未开始
  * 单利模式 -未开始
  * 工厂模式

## Java IO
  * 未开始

## 类加载过程
  * 未开始

## GC机制
  * 未开始

# Android
## 四大组件
  * Activity
  * service 
  * BroadcaseReciver
  * ContentProvide
  * Application
  * Intent

## 常用组件 
  * Fragment
  * LinearLayout、RelativeLayout相同层级 效率比较
  * RecyclerView 4级缓存+局部刷新 【 与listView比较 】
  * WebView - JsBridge、deeplink、首屏加速、离线包
  * Windows - Toast、Dialog、popWindow

# Android常见机制
## View 体系
  * 自定义view基础
    * 自定义属性
    * 坐标系了解
  * 绘制流程 
    * onMeasure
    * onLayout
    * onDraw
  * 自定义组合控件
    * 编写布局文件
    * 构造方法
    * 初始化UI
    * 提供对外方法
    * 布局中引用
  * 继承系统控件
    * 例如：TextView 、ImageView ...
  * 继承View 
    * 例如：extends View
  * 继承ViewGroup
    * 例如：extends ViewGroup
  * 事件分发（ 滑动冲突解决方案 ）
    * onTouchEvent
    * dispatchTouchEvent
    * onInterceptTouchEvent
  * 滑动嵌套，冲突

## 线程通信
  * Handler 
    * UI线程 - Handler.post、View.post、Activity.runOnThread
    * 子线程 - Looper.prepaeer + Looper.loop 或 HandlerThread
    * [Handler内存泄露原因分析](https://simpler.org.cn/relaxed/Handler-%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)
    * [Handler消息通讯原理分析](https://simpler.org.cn/relaxed/Handler-%E6%B6%88%E6%81%AF%E9%80%9A%E8%AE%AF%E6%9C%BA%E5%88%B6%E8%AF%A6%E8%A7%A3/)

## 进程通信（IPC）
  * Binder
    * AIDL 
      * 同进程 - asInterface -> 强转 ->直接调用
      * 夸进程 - asInterface -> Proxy{mRemote.tansact} -> Stub.onTransact
    * 数据序列化 - Serializable、Parcelable
  * 四大组件 
    * Messenger
    * Socket

## 动画
 * 动画类型
   * 属性动画
   * 帧动画
   * 位移动画

## BitMap 
  * 计算大小,内存分配 - inPreferredConfig = [ARGB_8888、RGB_565]
  * 裁剪、压缩等优化
  * bitmap复用
## 本地存储
  * sqlite、sharedPreferences、文件

# Android 常用第三方lib
## 网络
  * Retrofit - 动态代理，运行时注解注入 GsonConverter、Rxjava2CallAdapter
  * Okhttp - 拦截器（责任链模式）、超时重传&重定向、HTTP缓存、Socket连接池复用
  * Volley - todo
   
## 图片加载 
  * Glide - 生命周期控制、二级缓存、BitmapPool复用
    * [Glide缓存哪些事儿](https://simpler.org.cn/relaxed/glide-%E7%BC%93%E5%AD%98%E9%82%A3%E4%BA%9B%E4%BA%8B%E5%84%BF/)
    * [Glide最全解析](https://blog.csdn.net/guolin_blog/category_9268670.html)
  * Picasso - todo
  * Fresco - todo 

## 异步
 * RxJava
  * 常用操作符 - conpose、zip、map、flatMap、concatMap
  * 线程调度 - subsrcibe、observeOn（对下游生效、多次）
  * 异常处理 - onErrorReturn 、onErrorResumeNext
  * Flowable(背压) - todo

## 事件
  * EventBus - 注册、注销、post、四种模式、粘性事件 【观察者、解耦】
    * [EventBus使用详解](https://simpler.org.cn/relaxed/EventBus%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F/)
  * RxBus - 注册、注销、post、四种模式 【观察者、解耦】
  * RxBus与EventBus的区别是Rxbus 没有发送粘性事件的功能
  * 什么是粘性事件
    * 之前说的使用方法，都是需要先注册(register)，再post,才能接受到事件；如果你使用postSticky发送事件，那么可以不需要先注册，也能接受到事件，也就是一个延迟注册的过程。 
   普通的事件我们通过post发送给EventBus，发送过后之后当前已经订阅过的方法可以收到。但是如果有些事件需要所有订阅了该事件的方法都能执行呢？例如一个Activity，要求它管理的所有Fragment都能执行某一个事件，但是当前我只初始化了3个Fragment，如果这时候通过post发送了事件，那么当前的3个Fragment当然能收到。但是这个时候又初始化了2个Fragment，那么我必须重新发送事件，这两个Fragment才能执行到订阅方法。 
   粘性事件就是为了解决这个问题，通过 postSticky 发送粘性事件，这个事件不会只被消费一次就消失，而是一直存在系统中，知道被 removeStickyEvent 删除掉。那么只要订阅了该粘性事件的所有方法，只要被register 的时候，就会被检测到，并且执行。订阅的方法需要添加 sticky = true 属性。
  
## 依赖注入
  * ButterKnife
  * Dagger2

## 数据库
  * GreenDao

## 内存泄漏排查 
  * LeakCanary - 原理

# Android 进阶
## 优化
  * 布局优化 - 过度绘制、merge、include、viewstub
  * 内存优化 - 排查内存泄漏、OOM、图片压缩
  * 卡顿优化 - viewHodler、避免频繁GC、避免ANR
  * 冷启动优化 - 避免启动白屏
  * webView首屏优化（） - 离线包
  * apk瘦身 - 去除冗余资源文件、缩小jar
  * 流量、电量优化 - todo

## 动态化
  * 插件化
    * 类加载
      * 双亲委派
      * DexClassLoader、PathClassLoader区别
      * 如何 hook Activity启动流程
    * 资源加载

  * 热修复
  * RN/weex

## 组件化
  * 页面路由(ARouter)

## 消息推送 
  * 进程保活
  * 长链接保活 - 心跳包

## android Gradle插件
  * 未开始
 
## CoordinatorLayout基本使用
  * 未开始

## PriorityQueue队列触发器
  * 未开始

## SpannableStringBuilder
  * https://blog.csdn.net/wangxiaocheng16/article/details/56847457

## 面向切面编程（AOP） 
* 未开始

## MMKV存储框架 
* 未开始

## JVM 相关
* JVM中的线程分为两种
    * 守护线程: 守护线程是JVM自己使用的线程，比如垃圾回收（GC）就是一个守护线程。
    * 普通线程: 普通线程一般是Java程序的线程，只要JVM中有普通线程在执行，那么JVM就不会停止。
* 内存管理
* 垃圾清除算法
    * 标记清除
    * Copy
    * 标记压缩
* GC的演化
    * 随着内存大小的不断增长而演进
    * 从几招-几十兆 Serial单线程STW垃圾回收年轻代+老年代
    * 几十兆-几百兆 ps+po 多线程并行执行
    * 几十G 
* JVM 分配中两个年代
    * 新生代 
    * 老年代
* JVM调优的第一步更换垃圾回收器，使用多线程执行垃圾回收

* JVM 三色标记算法（只是做标记操作，没有做清楚处理）黑、灰、白
    * 初始标记 STW
    * 并发标记
    * 重新标记 STW
    * 并发清理  
* JVM 调优实战
    * 
* JVM 类加载器
    * 类加载器范围
      * 
    * 自定义加载器
      * 
* JVM 双亲委派
   
* JVM 父加载器
   
* JVM 编译器-大厂面试必问
* [volatile 禁止指令重排序](https://juejin.cn/post/7212101193437724728#comment)

* 对象创建过程

* 对象在内存中如何分配，栈上？堆上？

* 一个Object对象在内存总占多少个字节

* JVM性能优化调优实战

* JVM有哪些垃圾回收器？如何选择？

* JVM类加载器有哪些

* JVM有哪垃圾回收算法

* GC如何判断对象可以被回收

* 内存溢出的原因有哪些

* 如何解决线上GC频繁的问题

## 网络协议
* 未开始

## 关于Context知道的一切
  * 请看：https://blog.csdn.net/lmj623565791/article/details/40481055

## 3分钟看懂Activity启动流程
  * 请看：https://www.jianshu.com/p/9ecea420eb52

## Android Apk安装过程分析
  * 请看：https://www.jianshu.com/p/953475cea991

## Activity启动过程跟Window的关系？
  * 请看：https://juejin.cn/post/6844903974294781965

## Activity,Window,ViewRoot和DecorView之间的关系？
  * 请看：https://www.jianshu.com/p/8766babc40e0

## Android 性能优化最佳实践
  * 请看：https://juejin.cn/post/6844903641032163336

### 《Android Jetpack源码分析系列》
  * 请看：https://blog.csdn.net/mq2553299/category_9276155.html

### Android中为什么主线程不会因为Looper.loop()里的死循环卡死？
  * 请看：https://www.zhihu.com/question/34652589

### ArrayList与CopyOnWriteArrayList区别以及使用场景
  * 请看：https://www.cnblogs.com/simple-focus/p/7439919.html

### 关于HTTP协议，一篇就够了
  * 请看:https://www.cnblogs.com/ranyonsue/p/5984001.html
### HTTP 协议入门
  * 请看：http://www.ruanyifeng.com/blog/2016/08/http.html
### 网络基础知识之 HTTP 协议
  * 请看：https://zhuanlan.zhihu.com/p/24913080
### 文件上传下载
  * 请看：https://github.com/jeasonlzy/okhttp-OkGo/wiki/OkDownload#okdownload%E4%B8%BB%E8%A6%81%E5%8A%9F%E8%83%BD

### Android重构方案如何做->面试
1、注释: activity、fragment、类、方法 一定要有注释
   命名规范:统一，包括资源，java文件，布局，方法名，字段等等
3、工具类封装
4、第三方jar的统一 如网络，图片等等
5、模块化->组件化，MVP/MVVM
6、涉及模式的使用
7、网络数据结构重构
8、性能优化：卡顿，内存，启动，布局优化，apk体积
9、文档输出
10、重构之路，任重而道远，需要定期重构。

### Android 通知栏
  * https://blog.csdn.net/u011418943/article/details/105133041
  * https://blog.csdn.net/u011418943/article/details/105161112
