---
title: Glide-缓存那些事儿
date: 2022-03-23 23:29:41
categories:
 - Android
tags:
 - Android
cover: https://s1.ax1x.com/2022/03/23/q3s3Oe.jpg
---

# 前言：
 &ensp;在我们开发的时候对图片加载是必须使用的 这篇文章我们就聊聊Glide缓存。

## Glide 缓存分几种
答：缓存分为两种：内存缓存、磁盘缓存

内存缓存：
内存缓存又分为两种：一种是：弱引用缓存、一种是： Lrucache缓存。
往下看Glide内存缓存是如何实现的，源码如下：
```java
public <R> LoadStatus load(
      GlideContext glideContext,
      Object model,
      Key signature,
      int width,
      int height,
      Class<?> resourceClass,
      Class<R> transcodeClass,
      Priority priority,
      DiskCacheStrategy diskCacheStrategy,
      Map<Class<?>, Transformation<?>> transformations,
      boolean isTransformationRequired,
      boolean isScaleOnlyOrNoTransform,
      Options options,
      boolean isMemoryCacheable,
      boolean useUnlimitedSourceExecutorPool,
      boolean useAnimationPool,
      boolean onlyRetrieveFromCache,
      ResourceCallback cb,
      Executor callbackExecutor) {
    long startTime = VERBOSE_IS_LOGGABLE ? LogTime.getLogTime() : 0;

    // 生成key
    EngineKey key =
        keyFactory.buildKey(
            model,
            signature,
            width,
            height,
            transformations,
            resourceClass,
            transcodeClass,
            options);

    EngineResource<?> memoryResource;
    synchronized (this) {
    //进到loadFromMemory方法里面看看做了什么
      memoryResource = loadFromMemory(key, isMemoryCacheable, startTime);

     ...

 @Nullable
  private EngineResource<?> loadFromMemory(
      EngineKey key, boolean isMemoryCacheable, long startTime) {
    if (!isMemoryCacheable) {
      return null;
    }

  // 弱引用
    EngineResource<?> active = loadFromActiveResources(key);
    if (active != null) {
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Loaded resource from active resources", startTime, key);
      }
      return active;
    }

   // LruCaChe
    EngineResource<?> cached = loadFromCache(key);
    if (cached != null) {
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Loaded resource from cache", startTime, key);
      }
      return cached;
    }

  // 如果返回null ，从网络或其它地方获取
    return null;
  }

```
可以看到，Engine类中load方法图片加载策略，其中生成key的参数中包含width 、height
 ```java
  EngineKey key = keyFactory.buildKey(model, signature, width, height, transformations,
        resourceClass, transcodeClass, options);
 ```
*  width  & height 就是ImageView的尺寸，生成key时为什么会传入ImageView的尺寸呢？
* 注：Glide 通过key读取缓存图片时，会校验当前的图片尺寸是否与imageView的尺寸相等，如果相等直接取出来显示，如果不相等，会重新生成新的缓存数据。

### 弱引用缓存：
 &ensp;接着往下看，首先从弱引用中获取数据：
  ```
  EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
  //方法内部实现
 @Nullable
  private EngineResource<?> loadFromActiveResources(Key key) {
    EngineResource<?> active = activeResources.get(key);
    if (active != null) {
      active.acquire();
    }

    return active;
  }
//方法 acquire
synchronized void acquire() {
    if (isRecycled) {
      throw new IllegalStateException("Cannot acquire a recycled resource");
    }
    ++acquired;
  }

  ```
* 注：如果弱引用中获取到数据，计数器acquired+1，如果获取不到active对象在合适的时机被回收，走LruCaChe缓存，接着往下看：

### LruCaChe缓存：
弱引用中未获取到数据，则通过LruCaChe获取，源码如下：
```java
private EngineResource<?> loadFromCache(Key key) {
    EngineResource<?> cached = getEngineResourceFromCache(key);
    if (cached != null) {
      cached.acquire();
      activeResources.activate(key, cached);
    }
    return cached;
  }
```
* 注：如果cached对象不为空，则调用activeResources.activate方法将其加入弱引用缓存中，且计数器acquired+1

进入getEngineResourceFromCache方法看下具体实现：
```java
  private EngineResource<?> getEngineResourceFromCache(Key key) {
    Resource<?> cached = cache.remove(key);

    final EngineResource<?> result;
    if (cached == null) {
      result = null;
    } else if (cached instanceof EngineResource) {
      // Save an object allocation if we've cached an EngineResource (the typical case).
      result = (EngineResource<?>) cached;
    } else {
      result =
          new EngineResource<>(
              cached, /*isMemoryCacheable=*/ true, /*isRecyclable=*/ true, key, /*listener=*/ this);
    }
    return result;
  }
```
总结：之所以采用内存缓存是为防止应用重复将图片读入到内存，造成内存资源浪费。
* 注：从上面源码中我们看到，读内存缓存的流程为：先从弱引用中取数据，如果取到，计数器acquired+1返回，如果不存在，则从LruCaChe中取，如果取到，计数器acquired+1返回。如果LruCaChe中不存key对应在数据，则会走磁盘缓存，往下看：

### 磁盘缓存：
先了解以下磁盘缓存的策略有哪些?
* ALL:缓存原图(SOURCE)和处理图(RESULT)
* NONE:什么都不缓存
* SOURCE:只缓存原图(SOURCE)
* RESULT:只缓存处理图(RESULT) ---默认值

上面讲的是内存读取数据，如果内存没有取到，则Glide会从磁盘缓存取数据，通过EngineJob开启线程池去加载图片。这里有2个关键类：DecodeJob 和EngineJob。
一个图片的加载在 DecodeJob 这个类中，这个任务由 EngineJob 这个线程池去执行的。去到 run 方法，源码如下：
```java
private void runWrapped() {
    switch (runReason) {
      case INITIALIZE:
        stage = getNextStage(Stage.INITIALIZE);
        currentGenerator = getNextGenerator();
        runGenerators();
        break;
      case SWITCH_TO_SOURCE_SERVICE:
        runGenerators();
        break;
      case DECODE_DATA:
        decodeFromRetrievedData();
        break;
      default:
        throw new IllegalStateException("Unrecognized run reason: " + runReason);
    }
  }
```
EngineJob 内部维护了线程池，用来管理资源加载，当资源加载完毕的时候通知回调；
DecodeJob是线程池中的一个任务。
&ensp;磁盘缓存是通过DiskLruCache来管理的,根据不同的缓存策略，会有2种类型的图片：
* DATA：原始图片
*  RESOURCE：转换后的图片
磁盘缓存依次通过ResourcesCacheGenerator（转换过的缓存数据）、SourceGenerator（未经转换的原始的缓存数据）、DataCacheGenerator（网络获取图片数据后根据缓存策略的不同去缓存不同的图片到磁盘上）来获取缓存数据。
总结：防止应用重复的从网络或者其他地方下载和读取数据

关于LruCache源码请看：{% post_link LinkedHashMap-原理分析 LinkedHashMap原理分析 %}
关于LinedHashMap原理请看： {% post_link LruCache-原理分析 LruCache原理分析 %}


## Glide读取图片，获取顺序
答： 弱引用--> LruCache算法--> 磁盘缓存(DiskLruCache)

> 我们APP中加载某张图片时，先去弱引用（WeakReference）中寻找，如果弱引用（WeakReference）中有，则从WeakReference中取出图片使用，如果WeakReference中没有图片则去LruCache中寻找图片，如果LruCache中有，则取出来使用，并将该图片放入WeakReference中，如果LruCache中没有，则从磁盘缓存/网络中加载图片。


## Glide缓存图片，写入顺序
答：弱引用 --> LruCache算法 --> 磁盘缓存中(DiskLruCache)
>当图片不存在的时候，会先从网络下载，然后将图片存入弱引用中，Glide会采用一个acquired变量用来记录图片被引用的次数
当acquired变量大于0的时候，说明图片正在使用中，也就是将图片放到弱引用缓存当中。
如果acquired变量等于0了，说明图片已经不再被使用了，那么此时会调用方法来释放资源，首先会将缓存图片从弱引用中移除，然后再将它put到LruResourceCache当中。
这样也就实现了正在使用中的图片使用弱引用来进行缓存，不在使用中的图片使用LruCache来进行缓存的功能。
上面磁盘缓存分析中我们知道，当图片资源加载到DecodeJob这个类中，任务是有EngineJob线程池去执行的，假设我们将缓存策略设置为 DiskCacheStrategy.ALL 分析下源码中是如何执行的，下面先看runWrapped方法：
```java
private void runWrapped() {
    switch (runReason) {
      case INITIALIZE:
        stage = getNextStage(Stage.INITIALIZE);
        currentGenerator = getNextGenerator();
        runGenerators();
        break;
      case SWITCH_TO_SOURCE_SERVICE:
        runGenerators();
        break;
      case DECODE_DATA:
        decodeFromRetrievedData();
        break;
      default:
        throw new IllegalStateException("Unrecognized run reason: " + runReason);
    }
  }
```
注：由于我们的策略为DiskCacheStrategy.ALL 所以diskCacheStrategy.decodeCachedResource() 为true，即会解析解码的流程，所以 State 被赋值为 Stage.RESOURCE_CACHE，如下：
进入getNextStage方法：
```java
private Stage getNextStage(Stage current) {
    switch (current) {
      case INITIALIZE:
        return diskCacheStrategy.decodeCachedResource()
            ? Stage.RESOURCE_CACHE
            : getNextStage(Stage.RESOURCE_CACHE);
      case RESOURCE_CACHE:
        return diskCacheStrategy.decodeCachedData()
            ? Stage.DATA_CACHE
            : getNextStage(Stage.DATA_CACHE);
      case DATA_CACHE:
        // Skip loading from source if the user opted to only retrieve the resource from cache.
        return onlyRetrieveFromCache ? Stage.FINISHED : Stage.SOURCE;
      case SOURCE:
      case FINISHED:
        return Stage.FINISHED;
      default:
        throw new IllegalArgumentException("Unrecognized stage: " + current);
    }
  }
```
注：currentGenerator = getNextGenerator() 拿到当前的解码器为 ResourceCacheGenerator然后 调用 runGenerators() 方法。看下里面具体实现。
```java
private void runGenerators() {
    currentThread = Thread.currentThread();
    startFetchTime = LogTime.getLogTime();
    boolean isStarted = false;
    while (!isCancelled
        && currentGenerator != null
        && !(isStarted = currentGenerator.startNext())) {
      stage = getNextStage(stage);
      currentGenerator = getNextGenerator();

      //stage == Stage.SOURCE)时退出,并调用reschedule
      if (stage == Stage.SOURCE) {
        reschedule();
        return;
      }
    }
    // We've run out of stages and generators, give up.
    if ((stage == Stage.FINISHED || isCancelled) && !isStarted) {
      notifyFailed();
    }

    // Otherwise a generator started a new load and we expect to be called back in
    // onDataFetcherReady.
  }
```
注：while循环不断通过startNext()解析不同的缓存策略，直到stage == Stage.SOURCE 后退出。

下面进入startNext看下源码：
```java
 @Override
  public boolean startNext() {
    List<Key> sourceIds = helper.getCacheKeys();
    ...
  //由于第一次拿不到key，while会一直查找
    while (modelLoaders == null || !hasNextModelLoader()) {
      resourceClassIndex++;
      if (resourceClassIndex >= resourceClasses.size()) {
        sourceIdIndex++;
    //直到返回false
        if (sourceIdIndex >= sourceIds.size()) {
          return false;
        }
        resourceClassIndex = 0;
      }

      Key sourceId = sourceIds.get(sourceIdIndex);
     ...
      currentKey =
          new ResourceCacheKey( // NOPMD AvoidInstantiatingObjectsInLoops
              helper.getArrayPool(),
              sourceId,
              helper.getSignature(),
              helper.getWidth(),
              helper.getHeight(),
              transformation,
              resourceClass,
              helper.getOptions());
      cacheFile = helper.getDiskCache().get(currentKey);
      if (cacheFile != null) {
        sourceKey = sourceId;
        modelLoaders = helper.getModelLoaders(cacheFile);
        modelLoaderIndex = 0;
      }
    }

    loadData = null;
    boolean started = false;
    while (!started && hasNextModelLoader()) {
      ModelLoader<File, ?> modelLoader = modelLoaders.get(modelLoaderIndex++);
      loadData =
          modelLoader.buildLoadData(
              cacheFile, helper.getWidth(), helper.getHeight(), helper.getOptions());
      if (loadData != null && helper.hasLoadPath(loadData.fetcher.getDataClass())) {
        started = true;
        loadData.fetcher.loadData(helper.getPriority(), this);
      }
    }

    return started;
  }
```
下面看下reschedule()方法源码：
```java
  @Override
  public void reschedule() {
    runReason = RunReason.SWITCH_TO_SOURCE_SERVICE;
    callback.reschedule(this);
  }
```
注：重复调用 runWrapped() 方法，但此时的 runReason 已经变成了 SWITCH_TO_SOURCE_SERVICE，所以它会执行 runGenerators() 方法
```java
 private void runGenerators() {
    currentThread = Thread.currentThread();
    startFetchTime = LogTime.getLogTime();
    boolean isStarted = false;
    while (!isCancelled
        && currentGenerator != null
        && !(isStarted = currentGenerator.startNext())) {
      stage = getNextStage(stage);
      currentGenerator = getNextGenerator();

      if (stage == Stage.SOURCE) {
        reschedule();
        return;
      }
    }
    if ((stage == Stage.FINISHED || isCancelled) && !isStarted) {
      notifyFailed();
    }
  }
```
注：此时 currentGenerator 为 SourceGennerator 已经不为null。

接着看SourceGennerator类中startNext：
```java
@Override
  public boolean startNext() {
    if (dataToCache != null) {
      Object data = dataToCache;
      dataToCache = null;
      // 存储到 DiskLruCache
      cacheData(data);
    }

    if (sourceCacheGenerator != null && sourceCacheGenerator.startNext()) {
      return true;
    }
    sourceCacheGenerator = null;

    loadData = null;
    boolean started = false;
    while (!started && hasNextModelLoader()) {
      loadData = helper.getLoadData().get(loadDataListIndex++);
      if (loadData != null
          && (helper.getDiskCacheStrategy().isDataCacheable(loadData.fetcher.getDataSource())
              || helper.hasLoadPath(loadData.fetcher.getDataClass()))) {
        started = true;
        /加载数据
        startNextLoad(loadData);
      }
    }
    return started;
  }
```
> 硬盘缓存时通过在 EngineJob 中的 DecodeJob 中完成的，先通过ResourcesCacheGenerator、DataCacheGenerator 看是否能从 DiskLruCache 拿到数据，如果不能，从SourceGenerator去解析数据，并把数据存储到 DiskLruCache 中，后面通过 DataCacheGenerator 的 startNext() 去分发 fetcher 。
最后会回调 EngineJob 的 onResourceReady() 方法了，该方法会加载图片，并把数据存到弱引用中
