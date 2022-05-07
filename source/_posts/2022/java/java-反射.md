---
title: Java高级特性-反射
date: 2022-03-30 18:17:18
categories:
 - Java
tags:
 - Java
cover: https://s1.ax1x.com/2022/03/30/q2mOxI.jpg
---
# 前言
 **[JAVA反射机制](https://baike.baidu.com/item/JAVA%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6/6015990)是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。**

# 应用场景
Android中最熟悉的反射，莫过于Json数据的转换，例如网络数据，数据库数据和类之间的相互转化。使用反射机制可以直接创建对象，方便代码管理。

# 相关类
[. Class](https://developer.android.google.cn/reference/java/lang/Class)
* class类下部分常用方法

| 方法名| 翻译 |
| :-----:|:----: |
| [forName(String className)](https://developer.android.google.cn/reference/java/lang/Class#forName(java.lang.String,%20boolean,%20java.lang.ClassLoader)) | 加载参数指定的类，并且初始化它。|
| [cast](https://developer.android.google.cn/reference/java/lang/Class#cast(java.lang.Object)) | 对象转换成代表类或是接口的对象|
| [getClasses()](https://developer.android.google.cn/reference/java/lang/Class#cast(java.lang.Object)) |返回一个数组，数组中包含该类中所有公共类和接口类的对象|
| [getName()](https://developer.android.google.cn/reference/java/lang/Class#getName()) |获取类路径名称|
| [newInstance](https://developer.android.google.cn/reference/java/lang/Class#newInstance()) |创建类的实例
| [getPackage()](https://developer.android.google.cn/reference/java/lang/Class#getPackage()) |获取类所在包|
| [getInterfaces()](https://developer.android.google.cn/reference/java/lang/Class#getPackage()) |确定此对象所表示的类或接口实现的接口|
| [getSuperclass()](https://developer.android.google.cn/reference/java/lang/Class#getSuperclass()) |获得当前类继承的父类的名字|
| [getClassLoader()](https://developer.android.google.cn/reference/java/lang/Class#getClassLoader()) |获取类加载器|
| [getDeclaredClasses()](https://developer.android.google.cn/reference/java/lang/Class#getDeclaredClasses()) |返回一个数组，数组中包含该类中所有类和接口类的对象|

* Class类下相关属性方法

| 方法名| 翻译 |
| :-----:|:----: |
| [getField(String name)](https://developer.android.google.cn/reference/java/lang/Class#getField(java.lang.String)) | 获取某个公有的属性对象 |
| [getFields()](https://developer.android.google.cn/reference/java/lang/Class#getFields()) | 获取所有公有的属性对象 |
| [getDeclaredField(String name)](https://developer.android.google.cn/reference/java/lang/Class#getDeclaredField(java.lang.String))) | 获取某个私有属性对象 |
| [getDeclaredFields()](https://developer.android.google.cn/reference/java/lang/Class#getDeclaredFields()) | 获取所有私有属性对象 |

* Class 类下相关注解方法

| 方法名| 翻译 |
| :-----:|:----: |
| [getDeclaredAnnotations()](https://developer.android.google.cn/reference/java/lang/Class#getDeclaredAnnotations()) | 返回对应类下所有注解对象|
| [getDeclaredAnnotation(Class<A> annotationClass)](https://developer.android.google.cn/reference/java/lang/Class#getDeclaredAnnotation(java.lang.Class%3CA%3E)) | 返回对应类中与参数类型匹配的所有注解对象|
| [getAnnotation(Class<A> annotationClass)](https://developer.android.google.cn/reference/java/lang/Class#getAnnotation(java.lang.Class%3CA%3E)) | 返回对应类中与参数类型匹配的公有注解对象|
| [getAnnotations ()](https://developer.android.google.cn/reference/java/lang/Class#getAnnotations()) | 返回对应类中所有公有注解对象|

* Class 类下其它相关方法

| 方法名| 翻译 |
| :-----:|:----: |
| [getMethods()](https://developer.android.google.cn/reference/java/lang/Class#getMethods()) | 返回该类所有公有的方法|
| [getMethod (String name, Class...<?> parameterTypes)](https://developer.android.google.cn/reference/java/lang/Class#getMethod(java.lang.String,%20java.lang.Class%3C?%3E...)) | 返回该类某个公有方法|
| [getDeclaredMethod(String name, Class...<?> parameterTypes)](https://developer.android.google.cn/reference/java/lang/Class#getDeclaredMethod(java.lang.String,%20java.lang.Class%3C?%3E...)) | 返回该类某个方法|
| [getDeclaredMethods()](https://developer.android.google.cn/reference/java/lang/Class#getDeclaredMethods()) | 返回该类所有方法|

[. Method](https://developer.android.google.cn/reference/java/lang/reflect/Field)
* Method类下部分常用方法

| 方法名| 翻译 |
| :-----:|:----: |
| [invoke(Object obj, Object... args)](https://developer.android.google.cn/reference/java/lang/reflect/Method#invoke(java.lang.Object,%20java.lang.Object...)) | 传递object对象及参数调用该对象对应的方法|

[. Field](https://developer.android.google.cn/reference/java/lang/reflect/Field)
* Field类下部分常用方法

| 方法名| 翻译 |
| :-----:|:----: |
| [get(Object obj)](https://developer.android.google.cn/reference/java/lang/reflect/Field#get(java.lang.Object)) | 获得obj中对应的属性值|
| [set(Object obj, Object value)](https://developer.android.google.cn/reference/java/lang/reflect/Field#set(java.lang.Object,%20java.lang.Object)) | 设置obj中对应属性值|

[. Constructor](https://developer.android.google.cn/reference/java/lang/reflect/Constructor)
* Constructor类下部分常用方法

| 方法名| 翻译 |
| :-----:|:----: |
| [newInstance(Object... initargs)](https://developer.android.google.cn/reference/java/lang/reflect/Constructor#newInstance(java.lang.Object...)) | 根据传递的参数创建类的对象|


# 示例
* 反射实体类
```java
public class GloryOfKings {

    private String role;
    private String skills;
    private String explain;

    public GloryOfKings() {
    }

    private GloryOfKings(String role, String skills) {
        this.role = role;
        this.skills = skills;
    }

    public String getRole() {
        return role;
    }

    public void setRole(String role) {
        this.role = role;
    }

    public String getSkills() {
        return skills;
    }

    public void setSkills(String skills) {
        this.skills = skills;
    }

    public void setExplain(String explain) {
        this.explain = explain;
    }

    public String getExplain() {
        return explain;
    }

    private String week(int flag) {
        String string = "";
        switch (flag) {
            case FlagMethod.monday:
                string = "天气晴朗";
                break;
            case FlagMethod.tuesday:
                string = "多云";
                break;
            case FlagMethod.Wednesday:
                string = "微风";
                break;
            case FlagMethod.Thursday:
                string = "阵风6级";
                break;
            case FlagMethod.Friday:
                string = "中雨";
                break;
            case FlagMethod.Saturday:
                string = "大雨";
                break;
            case FlagMethod.Sunday:
                string = "晴朗";
                break;
            default:
                string = "-.-";
        }

        return string;
    }
}

```
* 逻辑封装类
```java
public class Reflect {
    /**
     * 反射-创建对象（三种方式：.class、new MyClass().getClass，Class.forName("类的全路径")）
     */
    public static void newInstance(int flag) {
        String outMessage = "";
        try {
            switch (flag) {
                case FlagSource.newInstanceOne:
                    Class<GloryOfKings> cls = GloryOfKings.class;
                    GloryOfKings gloryOfKings = cls.newInstance();
                    gloryOfKings.setExplain("创建对象方式- GloryOfKings.class");
                    gloryOfKings.setRole("李元芳");
                    gloryOfKings.setSkills("利刃风暴");
                    outMessage = gloryOfKings.toString();
                    break;
                case FlagSource.newInstanceTwo:
                    Class<? extends GloryOfKings> cls1 = new GloryOfKings().getClass();
                    GloryOfKings gloryOfKings1 = cls1.newInstance();
                    gloryOfKings1.setExplain("创建对象方式- new GloryOfKings().getClass()");
                    gloryOfKings1.setRole("后羿");
                    gloryOfKings1.setSkills("灼日之矢”");
                    outMessage = gloryOfKings1.toString();
                    break;
                case FlagSource.newInstanceThree:
                    Class<?> cls2 = Class.forName("com.linked.business.reflect.GloryOfKings");
                    GloryOfKings gloryOfKings2 = (GloryOfKings) cls2.newInstance();
                    gloryOfKings2.setExplain("创建对象方式- Class.forName()");
                    gloryOfKings2.setRole("诸葛亮");
                    gloryOfKings2.setSkills("时空穿梭”");
                    outMessage = gloryOfKings2.toString();
                    break;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(outMessage);
    }

    /**
     * 反射-私有构造方法
     */
    public static void construction() {
        try {
            Class<?> cls = Class.forName("com.linked.business.reflect.GloryOfKings");
            Constructor<?> constructor = cls.getDeclaredConstructor(String.class, String.class);
            constructor.setAccessible(true);
            Object obj = constructor.newInstance(" 反射-设置私有构造方法：名称", " 反射-获取私有构造方法：福利");
            GloryOfKings go = (GloryOfKings) obj;
            System.out.println("反射-获取私有构造方法" + go);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 反射-私有属性
     */
    public static void reflectDeclaredAttribute() {
        try {
            Class<?> cls = Class.forName("com.linked.business.reflect.GloryOfKings");
            Object obj = cls.newInstance();
            Field field = cls.getDeclaredField("TAG");
            field.setAccessible(true);
            String str = (String) field.get(obj);
            System.out.println("反射-获取私有属性" + str);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 反射-私有方法
     * <p>将此对象的 accessible 标志设置为指示的布尔值。值为 true 则指示反射的对象在使用时应该取消 Java 语言访问检查。值为 false 则指示反射的对象应该实施 Java 语言访问检查;
     * 实际上setAccessible是启用和禁用访问安全检查的开关,并不是为true就能访问为false就不能访问 ；
     * 由于JDK的安全检查耗时较多.所以通过setAccessible(true)的方式关闭安全检查就可以达到提升反射速度的目的
     * </p>
     */
    public static void privateMethod() {
        try {
            Class<?> cls = Class.forName("com.linked.business.reflect.GloryOfKings");
            Method method = cls.getDeclaredMethod("week", int.class);
            method.setAccessible(true);
            Object obj = cls.newInstance();
            //invoke：传递object对象及参数调用该对象对应的方法
            String str = (String) method.invoke(obj, FlagMethod.monday);
            System.out.println(str);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
* 测试类
```java
public class ReflectTest {
    public static void main(String[] args) {
        try {
            Reflect.newInstance(FlagSource.newInstanceTwo);
            Reflect.attribute();
            Reflect.construction();
            Reflect.privateMethod();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
