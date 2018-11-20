---
title: "Effective Java Third Edition Notes 2"
date: 2018-11-12T22:20:10-05:00
draft: false
---

当`constructor`的参数太多时，考虑使用builder。这篇笔记对应原书

> Item 2: Consider a builder when faced with many constructor parameters

有时一个Java class会有许多fields，其中少数是必须要赋值的，其余的一部分对于不同的instance赋不同的值，还有的只需保持默认值即可。

举例来说，假设有个class是食品的营养成分表，包含的field有体积，重量，卡路里，脂肪含量，碳水化合物含量，钠含量，钙含量等等等，超多的field。其中体积和重量是必须要赋值的，因为不同的食物有不同的值。至于钠含量等微量元素，可能有的食物根本没有，那么用默认值`0`就好；如果某个食物含有钠，就需要赋值。

这种情况下，创建object时，初始化的套路有几种，作者一一分析了。

第一种套路，class里有若干`constructor`，每个`constructor`有不同的参数，由`client`来选择合适的`constructor`来使用。

```java
public class NutritionFacts {
    private final int servingSize;  // (mL)            required
    private final int servings;     // (per container) required
    private final int calories;     // (per serving)   optional
    private final int fat;          // (g/serving)     optional
    private final int sodium;       // (mg/serving)    optional
    private final int carbohydrate; // (g/serving)     optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
    
}
```

这种方法可行，但是代码可读性太差。那么多`constructor`很凌乱。另外调用`constructor`时传入的那串数字，很容易颠倒顺序搞错了。出了问题debug都不好找。

初始化的套路二，`constructor`只负责对必须要赋值的field赋值，然后调用每个field对应的`setter`方法来依次赋值。

```java
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize  = -1; // Required; no default value
    private int servings     = -1; // Required; no default value
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

这种方法也很常见，然而它的问题也很明显，甚至还不如套路一呢。

套路一虽然看着丑，但是它是可以实现immutable class的；套路二暴露了若干setter，immutable是不可能了。

另外，套路二里面，`client`需要手动调用一系列setter来赋值，假设漏掉了一个，不会出现任何报错信息，直到将来某个用到这个instance的时刻，bug才会爆出来。

最后，作者隆重介绍了套路三，也就是要设计模式里的builder模式。综合了套路一和套路二的优点，同时又没有它们的缺点。

代码如下，

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
```

`builder` 现在是 `NutritionFacts` 里面的一个static的class，这个初始化的语句清晰明确 

```java
NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
```

中间的`Builder(240, 8).calories(100).sodium(35).carbohydrate(27)`是在赋值，最后的`build()`一次性创建object。

最后作者还贴心的讲解了，这种builder在继承关系的class层级之间如何使用，怎样避免类型转换，我这里就不多说了。

这种builder的方法，在面对field众多的情况时，特别好用。当然如果你的class没那么复杂时，也可以使用，只是有点用宰牛刀杀鸡的感觉了。另外，因为创建builder本身也有开销，作者说当performance要求很高时，有可能有问题。