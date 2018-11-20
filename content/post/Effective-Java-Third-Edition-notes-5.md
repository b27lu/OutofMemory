---
title: "Effective Java Third Edition Notes 5"
date: 2018-11-19T21:39:49-05:00
draft: false
---

许多class都依赖于一个或者多个下游的资源。例如，一个做拼写检查的class依赖于一个字典。

有人把这种class写成静态工具class，这样的例子并不难找。就像这样

```java
// 反面教材，不够灵活且难以测试
public static SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() {}   //Noninstantiable

    public static boolean isValid(String word) { ... }
}
```

有人把这种class写成singleton，就像这样

```java
// 反面教材，误用singleton，在这里并不合适
public class SpellChecker {
    private final Lexicon dictionary = ...;
    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);
    public static boolean isValid(String word) { ... }    
}
```

这两种方法都不好，因为它们都默认只使用一种字典。实际上，每种语言都会需要自己的字典，而且做测试的时候，一般会使用特殊准备的字典。

那么如何解决这个问题？我们可以把`dictionary`变成不是final的，然后加一个改变字典的method。这不够简洁，同时也容易出错。

>如果一个class，它的行为由底层所依赖的资源来决定，那么用静态工具class或者singleton是不合适的。

我们真正想实现的效果是，client提供字典给拼写检查器使用，不同的拼写检查器可以使用完全不同的字典。换句话说就是，拼写检查器只做拼写检查这个活，完全是按图索骥即可，“图”由client来决定。

按图索骥不够口语化，举个日常生活的例子。你家新买了个房子要装修，于是请了工人来。工人只负责施工，但是具体怎么装修由你来决定。你给工人木地板，那工人就把铺上木地板；你给工人瓷砖，工人就把地铺上瓷砖。工人只负责“铺”这个动作，具体铺什么，由你(client)决定。

想满足这个要求，一个简单的套路就是资源作为constructor的参数传给class，这样当创建新instance的时候，直接使用传入的资源就行了。这就是dependency injection(依赖注入)的一种。在这里，字典就是依赖，然后字典被通过constructor注入到拼写检查器中了。

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellCheck(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
}
```

依赖注入其实是一个非常简单的概念，可能许多人一直在用，只是不知道这个套路还有一个名字，而这个名字比这个概念本身更难理解。

上面的例子是手动依赖注入，如果你的项目依赖太多，还是要用专业的依赖注入框架。大名鼎鼎的`Spring`就是作为一个依赖注入框架起家的。当然现在`Spring`能做的事已经远远不止依赖注入了。

---

本篇笔记对应原书

>Item 5: Prefer dependency injection to hardwiring resources