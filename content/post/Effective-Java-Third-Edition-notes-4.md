---
title: "Effective Java Third Edition Notes 4"
date: 2018-11-15T22:21:46-05:00
draft: false
---

有时候你需要一个只包含static field和static method的java class作为工具类(utility class)来使用。在java源代码中也有不少这样的例子，比如`java.lang.Math`和`java.util.Arrays`。

这种工具类是不需要实例化的，也是不应该被实例化的。但是即使你没有在这个工具类里写任何的constructor，compiler也会默认加上一个默认的public的constructor。这时一不小心就可能使用这个默认的constructor创建工具类class的实例。

如何彻底防止一个class被实例化？有人认为把class设置成abstract的就行了，但是abstract class可以被继承，继承得来的subclass 一样是可以实例化的。

其实正确的解决方法很简单，那就是手动写一个没有任何参数的private的constructor即可。因为java compiler只有在一个class没有任何constructor时才会自动添加默认的constructor，一旦我们手动写了一个constructor，java compiler就不会添加默认constructor了。

```java
public class UtilityClass {
    private UtilityClass() {

    }
}
```

因为private的constructor依然可能被class内部的其他method调用，我们可以在这个constructor里加上`throw new AssertionError()`来避免这种情况发生。

最后注意，如果一个class有且只有private的constructor，那么它就不能被继承了，因为subclass总是需要直接或者间接的调用super class的constructor，而private的constructor是不能被subclass调用的。

这篇笔记对应原书

> Item 4: Enforce noninstantiability with a private constructor
