---
title: Android-如何使用注解进行代码检查
date: 2022-09-15 16:36:34
categories:
 - Android
tags:
 - 注解
cover: https://s1.ax1x.com/2022/03/23/q3s3Oe.jpg
---

### Nullness 注解
* 使用 Nullness 注解可以检查给定变量、参数和返回值是否允许 null值
  * @Nullable ：表示可以为 null 的变量、参数或返回值，
  * @NonNull ：表示不可为 null 的变量、参数或返回值。

```java
@NonNull
@Override
public View onCreateView(String name, @NonNull Context context,@NonNull AttributeSet attrs) {
    //...
}
```

### 资源注解
* 资源注解的使用可使得在源码阶段让编辑器检查书写的不规范，一定程度上优化代码结构
  * @StringRes：      表示检查是否包含R.string引用
  * @ColorRes：       表示检查是否包含R.color引用
  * @ColorInt：       表示检查是否包含表示颜色的整型
  * @DrawableRes：    表示检查是否包含R.drawable引用
  * @DimenRes：       表示检查是否包含R.dimen引用
  * @InterpolatorRes：表示检查是否包含插值器引用

### 线程注解
 * 线程注解可以检查某个方法是否从某个特定类型的线程中调用
   * @MainThread：表示主线程
   * @UiThread：表示 UI 线程
   * @WorkerThread：表示工作线程
   * @BinderThread：表示Binder线程
   * @AnyThread：表示任何一个线程

### 值约束注解
* 使用值约束注解可验证传递的参数的值的合法性，可以借此指定参数的设置范围，可在一定程度上减少代码在主观程度上出现的错误
  * @IntRange：表示可以验证整型参数是否在指定范围内 
  * @FloatRange：表示可以验证浮点型参数是否在指定范围内
  * @Size：表示可以验证集合、数组、字符串参数是否在指定范围内，可指定最大值、最小值以及确切值

### 权限注解
* 权限注解 @RequiresPermission 可以验证方法调用方的权限，即当使用了权限注解的方法时会检查有没有指定的权限，如果没有则会提示要在 AndroidManifest.xml 文件中申明权限，如果是危险权限还有进行权限动态申请

```java
/**
 * 单个权限检查
 * @param message
 */
@RequiresPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE)
public void setMessage(String message) {
}

/**
 * 全部权限检查
 * @param message
 */
@RequiresPermission(allOf = {
        Manifest.permission.WRITE_EXTERNAL_STORAGE,
        Manifest.permission.READ_EXTERNAL_STORAGE})
public void setMesage(String message) {
}

/**
 * 某个权限检查
 * @param message
 */
@RequiresPermission(anyOf = {
        Manifest.permission.WRITE_EXTERNAL_STORAGE,
        Manifest.permission.READ_EXTERNAL_STORAGE})
public void setMesage(String message) {
}
```

### 返回值注解
* 返回值注解 @CheckResult 会检查某个方法的返回值是否被使用，如果没有被使用，则会根据 suggest 配置建议使用相同公民没有返回值的另一个方法，如果返回值使用了，则和未加该注解的方法一样

```java
@CheckResult(suggest="#enforcePermission(String,int,int,String)")
public  int checkPermission(@NonNull String permission, int pid, int uid){
    return 0;
}
``` 

### CallSuper 
* 使用 @CallSuper 注解会验证子类的重写方法是否调用父类的实现，这样约束的好处是可保证父类的实现不会修改，当然，如果不使用该注解，子类重写父类的方法可以不调用弗父类的默认实现

```java
/**
 * 父类
 * @CallSuper注解的使用
 */
public class Test {
    //使用@CallSuper注解，子类重写该方法时必须调用该方法
    @CallSuper
    protected void onCreate(){
        
    }
}
```

### Typedef注解
* 使用 @IntDef 和 @StringDef 注解 可以创建整型和字符串的枚举注解来验证其他代码中使用的某些整型和字符串，可以保证代码中的某些常量整型或常量字符串是某些具体定义的常量集，这两个注解的位置只能是注解

```java
/**
 * Typedef 注解的定义
 */
public class ActionType {

    public static final int ACTION_TYPE_0 = 0;
    public static final int ACTION_TYPE_1 = 1;
    public static final int ACTION_TYPE_2 = 2;

    @Retention(RetentionPolicy.SOURCE)
    @IntDef({ACTION_TYPE_0,ACTION_TYPE_1,ACTION_TYPE_2})
    public @interface ActionTypeDef{

    }
}
```

### 可访问性注解
* 可访问性注解是 @VisibleForTesting 和 @Keep 可以表示方法、字段、类的可访问性
 
  * @VisibleForTesting：表示注解的某个代码块的可见性高于能够测试时需要的水平
  * @Keep：表示被注解的代码块将不会被混淆。