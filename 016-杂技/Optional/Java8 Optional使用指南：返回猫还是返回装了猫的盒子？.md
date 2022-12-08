# Optional用法初探



> 今天在Youtube上看Optional用法时发现Coding with John的教程很不错，浅显易懂，跟大家分享一下。
>
> 资料来源：https://www.youtube.com/watch?v=vKVzRbsMnTQ



**场景：**通过猫的姓名找到猫对象，最终输出其年龄

```java
public class OptionalTutorial {
    public static void main(String[] args) {
        Cat myCat = findCatbyName("Fred");
        // 目标:输出目标猫的年龄
        System.out.println(myCat.getAge());
    }
    // 通过姓名获取猫对象
    private static Cat findCatbyName(String name) {
        Cat cat = new Cat(name,3);
        return cat;
    }
}

// 自定义猫对象
@Data
public class Cat {
    private String name;
    private Integer age;

    public Cat(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}
```



但这个过程并非高枕无忧，在findCatbyName方法里，我们一般会通过前往数据库查找，如果库里没有数据，方法返回空，main方法里的myCat将为null，于是myCat.getAge()报空指针。所以，一般我们会这么处理：

```java
public class OptionalTutorial {
    public static void main(String[] args) {
        Cat myCat = findCatbyName("Fred");
        // 目标:输出目标猫的年龄
        if (myCat != null) {
             System.out.println(myCat.getAge());
        } else {
            // 目标猫咪不存在，输出一个默认值0
            System.out.println(0);
        }
    }
    // 通过姓名获取猫对象
    private static Cat findCatbyName(String name) {
        // 模拟从数据库获取名为name的猫咪
        Cat cat = new Cat(name,3);
        return cat;
    }
}
```



上面的if语句看着很冗余，这也是Optional想要解决的痛点——提醒对象的使用者，这个对象可能为空，你要注意处理为空的情况，免得报空指针了！

> “Optional就像一个盒子，如果你放入其中的cat实例确实存在，那盒子里放的就是它；但如果程序没有找到这只cat（即为null），那盒子里就空空如也，但Optional这个盒子还是会返给你。Optional这个类存在的目的，就是提醒使用它的人，返回泛型对象可能为null，要注意留心处理。”

如何创建一个'盒子'呢？

```java
Cat cat = new Cat("Fred",3);
// ofNullable静态方法，见字明义 —— ‘盒子’可能为空，所以cat如果为空，它也能接受
Optional<Cat> optionalCat1 = Optional.ofNullable(cat)；
// of静态方法，‘盒子’如果是空的则会报NPE
Optional<Cat> optionalCat2 = Optional.of(cat); 
```



经过一顿操作猛如虎，原来的代码可以改造为：

```java
public class OptionalTutorial {
    public static void main(String[] args) {
        Optional<Cat> myOptionalCat = findCatbyName("Fred");
        // 目标:输出目标猫的年龄
        if (myOptionalCat.isPresent()) { // isPrent()方法 -> “盒子里有东西吗？”
            System.out.println(myOptionalCat.get().getAge()); //有则获取盒子里的东西，然后调它的getAge方法
        } else {
            System.out.println(0);
        }
    }
    // 通过姓名获取猫对象，不同地是，我们返回一个‘可能装了猫咪的盒子’，而不是猫咪本身
    private static Optional<Cat> findCatbyName(String name) {
        Cat cat = new Cat(name,3);
        return Optional.ofNullable(cat);
    }
}
```



看完上面的代码，有些友友可能要抓狂了——你在逗我吗？这和没改造前的代码相比，也没便捷哪里去，甚至更不复杂了好吗？大家稍安勿躁，Optional还有一些有用的方法，比如最常用的orElse(); 它也不难理解——“盒子有目标猫咪就给我目标猫咪，如果没有，就用这个替代”：

```java
public class OptionalTutorial {
    public static void main(String[] args) {
        Optional<Cat> myOptionalCat = findCatbyName("Fred");
        // 从盒子里取猫 —— 盒子不为空则取之，为空则代之以orElse的入参
        Cat cat = myOptionalCat.orElse(new Cat("default",0));
        // 目标:输出目标猫的年龄
        System.out.println(cat.getAge());
    }
    
    // 通过姓名获取猫对象，不同地是，我们返回一个‘可能装了猫咪的盒子’，而不是猫咪本身
    private static Optional<Cat> findCatbyName(String name) {
        Cat cat = new Cat(name,3);
        return Optional.ofNullable(cat);
    }
}
```



经过上面这一步，代码是不是清爽了一些？其实还有更简洁的方法，比如我们最终要获取的是猫的年龄，可以直接对‘可能装着猫的盒子’进行映射，和StreamAPI里的map方法用起来差不多：

```java
public class OptionalTutorial {
    public static void main(String[] args) {
        Optional<Cat> myOptionalCat = findCatbyName("Fred");
        // 目标:输出目标猫的年龄
        System.out.println(
            myOptionalCat.map(Cat::getAge) // 盒子里本来放着猫, 现在映射成猫的年龄
                           .orElse(0) // 没有猫或者猫的年龄为null? —— 直接打印默认值0
        );
    }
    
    // 通过姓名获取猫对象，不同地是，我们返回一个‘可能装了猫咪的盒子’，而不是猫咪本身
    private static Optional<Cat> findCatbyName(String name) {
        Cat cat = new Cat(name, null);
        return Optional.ofNullable(cat);
    }
}
```



**补充：** 其实，除orElse()以外，Optional这个‘盒子’还提供了比如orElseThrow()方法：在盒子为空时抛出异常，但需要手动try catch捕捉‘抛出’的throwable。这个方法，笔者感觉目的在于可以定制抛出的错误类型，找不到Fred猫？抛一个自定义的“猫找不到”异常，而不是直接甩个NPE。

```java
public class OptionalTutorial {
    public static void main(String[] args) {
            Optional<Cat> myOptionalCat = findCatbyName("Fred");
            try {
                Cat myCat = myOptionalCat.orElseThrow(
                        () -> {
                            throw new RuntimeException("Can't find THE cat!");
                        }
                );
                System.out.println(myCat.getAge());
            } catch (Throwable t) { // 必须手动捕捉‘throw’并打印栈信息
                t.printStackTrace();
            }
        }

    // 通过姓名获取猫对象，不同地是，我们返回一个‘可能装了猫咪的盒子’，而不是猫咪本身
    private static Optional<Cat> findCatbyName(String name) {
        Cat cat = new Cat(name,3);
        return Optional.ofNullable(cat);
    }
}
```



