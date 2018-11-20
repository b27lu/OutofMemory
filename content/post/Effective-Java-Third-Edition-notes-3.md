---
title: "Effective Java Third Edition Notes 3"
date: 2018-11-14T18:38:21-05:00
draft: false
---

> Item 3: Enforce the singleton property with a private constructor or an enum type

singleton 单例，就是一个class **有且只有** 一个instance。

什么时候会用到singleton呢？当某个东西是 **stateless** 的时候，用singleton是很自然的，因为既然没有状态，那不管创建多少个instance，用的时候也没什么区别。还有一种情况就是，某个东西确实只有一个的时候，比如太阳，地球，或者系统里的资源管理器之类的。

singleton用着一时爽，但是也有一个问题就是singleton的client不太方便测试。假设你的class里用到了这个singleton的object，那当你想mock它的时候，没法不方便给它替换不同的实现，除非给这个singleton创建一个interface作为它的类型。

实现singleton的方法有几种，下面要说前两种方法都是基于同一思路，把constructor变成private的，同时提供一个public的static member来表示这个唯一的instance。

第一种方法，这个public static member是一个final的field

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {
        // constructor
    }
}
```

此时，constructor是private的，只在初始化`INSTANCE`的时候被调用一次，创建唯一的一个instance。这种方法可行，只有一个小问题就是如果真想打破singleton的限制，创建更多的instance，用reflection 反射的方法，`AccessOjbect.setAccessible()` 可以把private的constructor变成accessible的，然后调用它。

第二种方法和第一种类似，不过是提供一个public的static factory method来获取这个singleton的instance，而constructor和member field都是private的。

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {
        // constructor
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```

这种方法依然有第一种方法的小问题，并且也不如第一种方法简洁。那么它的优势是什么？一是方便修改，假设将来这个class不需要是singleton的了，那么只需要修改`getInstance()`方法即可，而API可以保持不变。另外就是method reference可以作为supplier，比如`Elvis::instance`就是一个`Supplier<Elvis>`。

另外注意一点，以上两种方法，如果想让它们`serializable`的话，仅仅加上`implements Serializable`是不够的。因为每次serialized instance被desearialized的时候，都会创建新的instance，不再是singleton了。解决方法是把所以的instance field设为`transient`，并提供`readResolve()`方法

```java
private Object readResolve() {
    return INSTANCE；
}
```

第三种实现singleton的方法，也是作者最推荐的一种，是通过声明一个只有一个值的enum来实现。

```java
public enum Elvis {
    INSTANCE;
}
```

这个方法最为简练，同时也是serializable的，并且即使用reflection的方法也无法创建额外的instance。可以说整合了前两种方法的优点，同时避免了它们的不足。唯一的一个限制就是这个singleton的class不能是继承自其他class的（只能是继承自enum）。