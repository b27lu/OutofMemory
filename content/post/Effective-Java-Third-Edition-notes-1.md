---
title: "Effective Java Third Edition Notes 1"
date: 2018-11-11T21:45:16-05:00
draft: false
---

Effective Java 第三版读书笔记-(1)

这篇读书笔记是关于`Item1: Consider static factory methods instead of constructors`的内容。

如果Java的某个class想允许使用者获得这个类的一个instance，传统方法是提供一个`public`的`constructor`. 其实还有另外一种方法，那就是提供一个`public`的 `static factory`方法。当然这里的`static factory`只是一个概念，方法的名字不一定就叫`XXXFactory`。

作者举一个Java源代码里的例子

```java
    public static Boolean valueOf(boolean b) {
        return b ? Boolean.TRUE : Boolean.FALSE;
    }
```

注意这个`static factory`和设计模式里的`factory method`没什么关系。

用这种`static factory`代替`constructor`有好处也有坏处。

好处如下：

1. `static factory`有名字，而`constructor`没有。刚开始看你会觉得这不是废话吗？！是啊，谁都知道`constructor`的名字必须和`class`的名字一样，就算你有8个`constructor`，它们的名字也必须一样。但是如果你真有8个`constructor`，如果想弄清楚每个`constructor`都是干嘛用的也没那么容易。这时`static factory`的优势就体现出来了，你可以给它们起一个意义明确的名字，让使用者别那么懵逼。
2. 每次调用`constructor`都必然会新建一个`instance`，但是通过`static factory`你可以灵活控制，比如返回一个以前创建好的`instance`，或者每次调用都返回同一个`instance`。这有什么用处吗？有时候还真有。比如你需要`singleton`的时候。比如，你需要一个immutable的class保证当且仅当`a == b`的时候，`a.equals(b)`。
3. `static factory`可以返回这个`class`的subclass；而`constructor`就不行。什么时候需要这么做呢？作者举了一个`java.util.Collections`的例子。因为直到Java8之前，Java的interface里都是不能有`static`的方法的，所以传统上，如果某个interface叫`Type`那么它对应的`static factory`方法会被放在一个叫`Types`的不能实例化的`class`里。Java的Collection interface，`public interface Collection<E> extends Iterable<E>` 有40多个implementation；它们都通过`public class Collections`里的若干`static factory`方法暴露给用户，而不是直接提供给用户使用。因此Java的collection API看上去简洁了许多。
4. 每次调用`static factory`方法获得的object可以根据`static factory`方法的输入参数决定。这个很好理解，不多说了。
5. 第五个好处是，当你写包含`static factory`方法的class的时候，这个方法所返回的object的class并不需要存在。这个说的太虚幻了，作者赶紧又举了个例子->JDBC.JDBC 这类东西叫`service provider framework`，其中provider提供了一种service，系统让client可以使用这种service，于是client和service的具体implementation之间就解耦合了。比如当你想连数据库的时候，不同类型的数据库，MySQL，DB2 等等的driver自然有人写好了，你只需要指明你的数据库类型，之后对应的driver就注册上了，然后就可以用了。这种`service provider framework`里面，有几个基本组成部分，`service interface`代表某种implementation。`provider registration API`是provider 用来注册`implementation`的。`service access API`是client用来获取service使用的。还有一种optional的组成部分，是`service provider interface`，它负责生成`service interface`的instance。对应到JDBC的情况，`Connection`就是这里的`service interface`，`DriverManager.registerDriver`是`provider registration API`。`DriverManager.getConnection`是`service access API`。`Driver`是`service provider interface` 从Java6 开始，Java提供了一个通用的`service provider framework`叫`java.util.ServiceLoader`，所以通常情况下，你不需要自己从头写一个`service provider framework`了。然而JDBC并没有用它，因为JDBC出现的时间要早得多。

用`static factory`方法代替`constructor`的缺点是
1. 如果不提供`public`或者`protected`的`constructor`，那这个`class`就别想被继承了。
2. 因为`static factory`方法没统一的命名，在文档里不好找啊。



