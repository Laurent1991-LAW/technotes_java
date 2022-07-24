# String常量池



## intern()方法

String类型的常量池比较特殊。它的主要使用方法有两种：

- 直接使用双引号声明出来的String对象会直接存储在常量池中；
- 使用String s1 = new String("xxx")创建出的字符串，会开辟内存空间存储为一个StringObject对象，使用String提供的intern()方法，将在 字符串常量池 中查询 当前字符串看是否存在，若不存在则会在堆中生成"xxx"的字符串对象o，将其引用存在常量池中；
- 字面量方式创建字符串String s2 =  "xxx"，由于常量池中已存在equal的字符串，直接返回o的引用给字段s2 ;



## 代码实例

```java
public class Test1 {
    @Test
    public void test(){
        String s1 = new String("2");
        s.intern();
        String s2 = "2";
        System.out.println(s1 == s2); // false
	   // ----------------------
        String s3 = new String("3") + new String("3");
        s3.intern();
        String s4 = "33";
        System.out.println(s3 == s4); // true
    }
}
```



## 解析

自JDK7开始，常量池中不再存放 字符串字面量本身，而是将该对象挪至堆中，常量池存储 对该字符串 的引用

![640](E:\doc_repo\String常量池\images\640.png)



- `String s = new String("2");`创建了两个对象，一个在堆中的StringObject对象，一个是在堆中的“2”对象，并在常量池中保存“2”对象的引用地址。
- `s.intern();`在常量池中寻找与s变量内容相同的对象，发现已经存在内容相同对象“2”，返回对象“2”的引用地址。
- `String s2 = "2";`使用字面量创建，在常量池寻找是否有相同内容的对象，发现有，返回对象“2”的引用地址。
- `System.out.println(s == s2);`从上面可以分析出，s变量和s2变量地址指向的是不同的对象，所以返回false

------

- `String s3 = new String("3") + new String("3");`创建了两个对象，一个在堆中的StringObject对象，一个是在堆中的“3”对象，并在常量池中保存“3”对象的引用地址。中间还有2个匿名的`new String(“3”)`我们不去讨论它们。
- `s3.intern();`在常量池中寻找与s3变量内容相同的对象，没有发现“33”对象，将s3对应的StringObject对象的地址保存到常量池中，返回StringObject对象的地址。
- `String s4 = "33";`使用字面量创建，在常量池寻找是否有相同内容的对象，发现有，返回其地址，也就是StringObject对象的引用地址。
- `System.out.println(s3 == s4);`从上面可以分析出，s3变量和s4变量地址指向的是相同的对象，所以返回true。



