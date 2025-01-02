---
title: 【尚硅谷】JVM-类的加载篇
categories: 学习笔记
series: JVM
tags:
  - 尚硅谷
  - Java
  - JVM
cover: 'https://img.shiguangdev.cn/i/2024/08/28/66cedaf5d014a.jpeg'
abbrlink: 1eaac7f
date: 2024-12-12 12:02:59
---
**官方资料：**

{% link 尚硅谷JVM精讲与GC调优教程（宋红康主讲，含jvm面试真题）,在线视频,https://www.bilibili.com/video/BV1Dz4y1A7FB?p=9,https://www.bilibili.com/favicon.ico %}

{% link 尚硅谷宋红康JVM精讲与GC调优,课程资料,https://pan.baidu.com/s/1ITPpXOuTwfBW2e5dksgqNw?pwd=yyds,https://nd-static.bdstatic.com/m-static/v20-main/favicon-main.ico %}

{% link 第2篇-类的加载篇,思维导图,https://blog.shiguangdev.cn/html/atguigu/%E5%B0%9A%E7%A1%85%E8%B0%B7_JVM%E7%B2%BE%E8%AE%B2%E4%B8%8EGC%E8%B0%83%E4%BC%98%E7%AC%AC2%E7%AF%87-%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E7%AF%87.html,https://th.bing.com/th?id=ODLS.257b82c4-9cf6-4599-8c46-03342a2f9e88&w=32&h=32&qlt=90&pcl=fffffa&o=6&pid=1.2 %}

**代码仓库：**

{% link Gitee,@an_shiguang,https://gitee.com/an_shiguang/learn-jvm,https://gitee.com/favicon.ico %}

{% link GitHub,@Shiguang-coding,https://github.com/Shiguang-coding/learn-jvm,https://github.com/favicon.ico %}

# 1、类的加载过程

## 1.1、类加载分为几步？

 按照Java虚拟机规范，从class文件到加载到内存中的类，到类卸载出内存为止，它的整个生命周期包括如下7个阶段：

![image-20241212120940677](【尚硅谷】JVM-类的加载篇/675a62044c046.webp)

其中：
① 第一过程的加载(loading)也称为装载
② 验证、准备、解析 3 个部分统称为链接（Linking）

从程序中类的使用过程看：

![image-20241212121136609](【尚硅谷】JVM-类的加载篇/675a6278292af.webp)

**都谁需要加载？**

在Java中数据类型分为基本数据类型和引用数据类型。**基本数据类型由虚拟机预先定义，引用数据类型则需要进行类的加载**。

> 面试题:
>
> - Java 类加载过程?（苏宁）
> - 描述一下 JVM 加载 Class 文件的原理机制?（国美）
> - JVM底层怎么加载class文件的？（蚂蚁金服）
> - 类加载过程 （蚂蚁金服）
> - Java 类加载过程? （百度）
> - 描述一下 JVM 加载 Class 文件的原理机制? （蚂蚁金服）
> - java类加载过程  （美团）
> - 描述一下JVM加载class文件的原理机制  （美团）
> - 什么是类的加载？ （京东）
> - 讲一下JVM加载一个类的过程 （京东）

## 1.2、过程一：Loading（装载）阶段

### 1.2.1、过程一都做了什么事？

**过程一：类的装载**

<font color = "red">所谓装载，简而言之就是将Java类的字节码文件加载到机器内存中，并在内存中构建出Java类的原型——类模板对象。</font>

**装载完成的操作**

<font color = "red">装载阶段，简言之，查找并加载类的二进制数据，生成Class的实例。</font>

在加载类时，Java虚拟机必须完成以下3件事情：

- 通过类的全名，获取类的二进制数据流。
- 解析类的二进制数据流为方法区内的数据结构（Java类模型）
- 创建java.lang.Class类的实例，表示该类型。作为方法区这个类的各种数据的访问入口

### 1.2.2、过程一中什么是类模板对象？

**类模板对象**

所谓类模板对象，其实就是Java类在JVM内存中的一个快照，JVM将从字节码文件中解析出的常量池、类字段、类方法等信息存储到类模板中，这样JVM在运行期便能通过类模板而获取Java类中的任意信息，能够对Java类的成员变量进行遍历，也能进行Java方法的调用。

反射的机制即基于这一基础。如果JVM没有将Java类的声明信息存储起来，则JVM在运行期也无法反射。

**类模型的位置**

加载的类在JVM中创建相应的类结构，类结构会存储在方法区(JDK1.8之前：永久代；JDK1.8及之后：元空间)。

### 1.2.3、二进制流有哪些获取方式？

对于类的二进制数据流，虚拟机可以通过多种途径产生或获得。**（只要所读取的字节码符合JVM规范即可）**

- 虚拟机可能通过文件系统读入一个class后缀的文件**（最常见）**
- 读入jar、zip等归档数据包，提取类文件。
- 事先存放在数据库中的类的二进制数据
- 使用类似于HTTP之类的协议通过网络进行加载
- 在运行时生成一段Class的二进制信息等

在获取到类的二进制信息后，Java虚拟机就会处理这些数据，并最终转为一个java.lang.Class的实例。

如果输入数据不是ClassFile的结构，则会抛出ClassFormatError。

### 1.2.4、Class实例的位置在哪？

类将`.class`文件加载至元空间后，会在堆中创建一个Java.lang.Class对象，用来封装类位于方法区内的数据结构，该Class对象是在加载类的过程中创建的，每个类都对应有一个Class类型的对象。(instanceKlass -->mirror :Class的实例)

![image-20241212123837995](【尚硅谷】JVM-类的加载篇/675a68cd78ed6.webp)

外部可以通过访问代表Order类的Class对象来获取Order的类数据结构。

**说明：**

Class类的构造方法是私有的，只有JVM能够创建。Class类的构造方法是私有的，只有JVM能够创建。

 java.lang.Class实例是访问类型元数据的接口，也是实现反射的关键数据、入口。通过Class类提供的接口，可以获得目标类所关联的.class文件中具体的数据结构：方法、字段等信息。 

### 1.2.5、数组类的加载有什么不同？

创建数组类的情况稍微有些特殊，因为**数组类本身并不是由类加载器负责创建**，而是由JVM在**运行时**根据需要而直接创建的，但数组的元素类型仍然需要依靠类加载器去创建。创建数组类（下述简称A）的过程：

1. 如果数组的元素类型是引用类型，那么就遵循定义的加载过程递归加载和创建数组A的元素类型；
2. JVM使用指定的元素类型和数组维度来创建新的数组类。
3. 如果数组的元素类型是引用类型，数组类的可访问性就由元素类型的可访问性决定。否则数组类的可访问性将被缺省定义为public。

## 1.3、过程二：Linking（链接）阶段

### 1.3.1、环节1：链接阶段之Verification（验证）

当类加载到系统后，就开始链接操作，验证是链接操作的第一步。

<font color = "red">它的目的是保证加载的字节码是合法、合理并符合规范的。</font>

验证的步骤比较复杂，实际要验证的项目也很繁多，大体上Java虚拟机需要做以下检査，如图所示。

![image-20241212130127375](【尚硅谷】JVM-类的加载篇/675a6e26e64be.webp)



**整体说明：**

验证的内容则涵盖了类数据信息的格式验证、语义检查、字节码验证，以及符号引用验证等。

- 其中<font color = "red">格式验证会和装载阶段一起执行</font>。验证通过之后，类加载器才会成功将类的二进制数据信息加载到方法区中。
- **格式验证之外的验证操作将会在方法区中进行**。

**具体说明：**

1、**格式验证**：是否以魔数 **OxCAFEBABE**开头，主版本和副版本号是否在当前Java虚拟机的支持范围内，数据中每一个项是否都拥有正确的长度等。

2、Java虚拟机会进行字节码的**语义检查**，但凡在语义上不符合规范的，虚拟机也不会给予验证通过。比如：

- 是否所有的类都有父类的存在(在Java里，除了Object外，其他类都应该有父类)
- 是否一些被定义为final的方法或者类被重写或继承了
- 非抽象类是否实现了所有抽象方法或者接口方法
- 是否存在不兼容的方法(比如方法的签名除了返回值不同，其他都一样，这种方法会让虚拟机无从下手调度；abstract情况下的方法，就不能是final的了)

3、Java虚拟机还会进行**字节码验证**，**字节码验证也是验证过程中最为复杂的一个过程**。它试图通过对字节码流的分析，判断字节码是否可以被正确地执行。比如：

- 在字节码的执行过程中，是否会跳转到一条不存在的指令
- 函数的调用是否传递了正确类型的参数
- 变量的赋值是不是给了正确的数据类型等

栈映射帧（StackMapTable）就是在这个阶段，用于检测在特定的字节码处，其局部变量表和操作数栈是否有着正确的数据类型。但遗憾的是，100%准确地判断一段字节码是否可以被安全执行是无法实现的，因此，该过程只是尽可能地检査出可以预知的明显的问题。**如果在这个阶段无法通过检查，虚拟机也不会正确装载这个类。但是，如果通过了这个阶段的检查，也不能说明这个类是完全没有问题的**。

**在前面3次检查中，已经排除了文件格式错误、语义错误以及字节码的不正确性。但是依然不能确保类是没有问题的。**

4、校验器还将进行**符号引用的验证**。Class文件在其常量池会通过字符串记录自己将要使用的其他类或者方法。因此，在验证阶段，**虚拟机就会检查这些类或者方法确实是存在的**，并且当前类有权限访问这些数据，如果一个需要使用类无法在系统中找到，则会抛出NoClassDefFoundError,如果一个方法无法被找到，则会抛出NoSuchMethodError。
<font color = "red">此阶段在解析环节才会执行。</font>

### 1.3.2、环节2：链接阶段之Preparation（准备）

<font color = "red">简言之，为类的静态变量分配内存，并将其初始化为默认值。</font>

在这个阶段，虚拟机就会为这个类分配相应的内存空间，并设置默认初始值。Java虚拟机为各类型变量默认的初始值如表所示。

![image-20241212132408607](【尚硅谷】JVM-类的加载篇/675a737855519.webp)

注意：Java并不支持boolean类型，对于boolean类型，内部实现是int,由于int的默认值是0,故对应的，boolean的默认值就是false。



**注意**：

1. **这里不包含基本数据类型的字段用static final修饰的情况，因为final在编译的时候就会分配了，准备阶段会显式赋值**。
2. 注意这里不会为实例变量分配初始化，实例变量是会随着对象一起分配到Java堆中。
3. **在这个阶段并不会像初始化阶段中那样会有初始化或者代码被执行。**



### 1.3.3、环节3：链接阶段之Resolution（解析）

<font color = "red">简言之，将类、接口、字段和方法的符号引用转为直接引用。</font>

**具体描述：**

符号引用就是一些字面量的引用，和虚拟机的内部数据结构和和内存布局无关。比较容易理解的就是在Class类文件中，通过常量池进行了大量的符号引用。但是在程序实际运行时，只有符号引用是不够的，比如当如下println()方法被调用时，系统需要明确知道该方法的位置。

举例：输出操作System.out.println()对应的字节码：

```bash
invokevirtual #24 <java/io/PrintStream.println>
```

![image-20241212133110843](【尚硅谷】JVM-类的加载篇/675a751f83f75.webp)

以方法为例，Java虚拟机为每个类都准备了一张方法表，将其所有的方法都列在表中，当需要调用一个类的方法的时候，只要知道这个方法在方法表中的偏移量就可以直接调用该方法。**通过解析操作，符号引用就可以转变为目标方法在类中方法表中的位置，从而使得方法被成功调用**。

**小结：** 
<font color = "red">所谓解析就是将符号引用转为直接引用，也就是得到类、字段、方法在内存中的指针或者偏移量。因此，可以说，如果直接引用存在，那么可以肯定系统中存在该类、方法或者字段。但只存在符号引用，不能确定系统中一定存在该结构。</font>

不过Java虚拟机规范并没有明确要求解析阶段一定要按照顺序执行。在HotSpot VM中，加载、验证、准备和初始化会按照顺序有条不紊地执行，**但链接阶段中的解析操作往往会伴随着JVM在执行完初始化之后再执行**。

**字符串复习：**

最后，再来看一下CONSTANT_String的解析。由于字符串在程序开发中有着重要的作用，因此，有必要了解一下String在Java虚拟机中的处理。**当在Java代码中直接使用字符串常量时，就会在类中出现CONSTANT_String**,它表示字符串常量，并且会引用一个CONSTANT_UTF8的常量项。**在Java虚拟机内部运行中的常量池中，会维护一张字符串拘留表(intern)，它会保存所有出现过的字符串常量，并且没有重复项**。只要以**CONSTANT_String**形式出现的字符串也都会在这张表中。使用**String.intern()**方法可以得到一个字符串在拘留表中的引用，因为该表中没有重复项，所以任何字面相同的字符串的String.intern()方法返回总是相等的。



## 1.4、过程三：Initialization（初始化）阶段

<font color = "red">简言之，为类的静态变量赋予正确的初始值。(显式初始化)</font>

**具体描述**

类的初始化是类装载的最后一个阶段。如果前面的步骤都没有问题，那么表示类可以顺利装载到系统中。此时，类才会开始执行Java字节码。（即：**到了初始化阶段，才真正开始执行类中定义的 Java 程序代码**。）

<font color = "red">初始化阶段的重要工作是执行类的初始化方法：\<clinit>()方法。</font>

- 该方法仅能由Java编译器生成并由JVM调用，程序开发者**无法自定义**一个同名的方法，更无法直接在Java程序中调用该方法，虽然该方法也是由字节码指令所组成。
- 它是由类静态成员的赋值语句以及static语句块合并产生的。



`<clinit>()` : 只有在给类的中的static的变量显式赋值或在静态代码块中赋值了。才会生成此方法。
`<init>()` : 一定会出现在Class的method表中。

### 1.4.1、子类加载前先加载父类？

在加载一个类之前，虚拟机总是会试图加载该类的父类，因此父类的`<clinit>()`总是在子类`<clinit>()`之前被调用。也就是说，父类的static块优先级高于子类。 

**口诀：由父及子，静态先行。**

**案例：**

```java
/**
 * @author shkstart
 * @create 2020-09-13 11:55
 *
 * 过程三：初始化阶段
 */
public class InitializationTest {
    public static int id = 1;
    public static int number;

    static {
        number = 2;
        System.out.println("father static{}");
    }
}


/**
 * @author shkstart
 * @create 2020-09-13 11:59
 */
public class SubInitialization extends InitializationTest {
    static{
        number = 4;//number属性必须提前已经加载：一定会先加载父类。
        System.out.println("son static{}");
    }

    public static void main(String[] args) {
        System.out.println(number);
    }
}
```

执行结果：

![image-20241212142649216](【尚硅谷】JVM-类的加载篇/675a8228a5d87.webp)

### 1.4.2、哪些类不会生成`<clinit>`方法？

- 一个类中并没有声明任何的类变量，也没有静态代码块时
- 一个类中声明类变量，但是没有明确使用类变量的初始化语句以及静态代码块来执行初始化操作时
- 一个类中包含static final修饰的基本数据类型的字段，这些类字段初始化语句采用编译时常量表达式

```java
/**
 * @author shkstart
 * 哪些场景下，java编译器就不会生成<clinit>()方法
 */
public class InitializationTest1 {
    //场景1：对于非静态的字段，不管是否进行了显式赋值，都不会生成<clinit>()方法
    public int num = 1;
    //场景2：静态的字段，没有显式的赋值，不会生成<clinit>()方法
    public static int num1;
    //场景3：比如对于声明为static final的基本数据类型的字段，不管是否进行了显式赋值，都不会生成<clinit>()方法
    public static final int num2 = 1;
}
```

### 1.4.3、代码举例：static与final的搭配问题

```java
package com.atguigu.java;

import java.io.PrintStream;
import java.sql.SQLOutput;
import java.util.Random;

/**
 * @author shkstart
 * @create 18:55
 *
 * 一、说明：使用static + final修饰的字段的显式赋值的操作，到底是在哪个阶段进行的赋值？
 * 情况1：在链接阶段的准备环节赋值
 * 情况2：在初始化阶段<clinit>()中赋值
 *
 * 二、结论：
 * 1. 在链接阶段的准备环节赋值的情况：
 * 1）对于基本数据类型的字段来说，如果使用static final修饰，则显式赋值(直接赋值常量，而非调用方法）通常是在链接阶段的准备环节进行
 * 2）对于String来说，如果使用字面量的方式赋值，使用static final修饰的话，则显式赋值通常是在链接阶段的准备环节进行
 *
 * 2. 在初始化阶段<clinit>()中赋值的情况：
 * 排除上述的在准备环节赋值的情况之外的情况。
 *
 *
 *  总结：
 *  使用static + final 修饰的成员变量，称为：全局常量。
 *  什么时候在链接阶段的准备环节：给此全局常量附的值是字面量或常量。不涉及到方法或构造器的调用。
 *  除此之外，都是在初始化环节赋值的。
 *
 */
public class InitializationTest2 {
//    public static int a = 1;   //在初始化阶段赋值
//    public static final int INT_CONSTANT = 10;   //在链接阶段的准备环节赋值
//
//    public static Integer INTEGER_CONSTANT1 = Integer.valueOf(100); //在初始化阶段赋值
//    public static final Integer INTEGER_CONSTANT2 = Integer.valueOf(1000); //在初始化阶段赋值
//
    public static final String s0 = "helloworld0";   //在链接阶段的准备环节赋值
    public static final String s1 = new String("helloworld1"); //在初始化阶段赋值
//
    public static String s2 = "helloworld2";  //在初始化阶段赋值
//
    public static final int NUM1 = new Random().nextInt(10);  //在初始化阶段赋值

    static int a = 9;//在初始化阶段赋值
    static final int b = a; //在初始化阶段赋值

}
```

对应的字节码：

```bash
0 iconst_1
 1 putstatic #2 <com/atguigu/java/InitilizationTest1.a>
 4 bipush 100
 6 invokestatic #3 <java/lang/Integer.valueOf>
 9 putstatic #4 <com/atguigu/java/InitilizationTest1.INTEGER_CONSTANT1>
12 sipush 1000
15 invokestatic #3 <java/lang/Integer.valueOf>
18 putstatic #5 <com/atguigu/java/InitilizationTest1.INTEGER_CONSTANT2>
21 new #6 <java/lang/String>
24 dup
25 ldc #7 <helloworld>
27 invokespecial #8 <java/lang/String.<init>>
30 putstatic #9 <com/atguigu/java/InitilizationTest1.s>
33 return
```

**重要结论：**

<font color = "red">普通基本数据类型和引用类型（即使是常量）的静态变量，是需要额外调用putstatic等JVM指令的，这些是在显式初始化阶段执行，而不是准备阶段调用；而基本数据类型常量(非调用方法的显式赋值)、String类型字面量的定义方式的常量，则不需要这样的步骤，是在准备阶段完成的。</font>

### 1.4.4、`<clinit>()`的调用会死锁吗？

对于`<clinit>()`方法的调用，也就是类的初始化，虚拟机会在内部确保其多线程环境中的安全性。

<font color = "red">虚拟机会保证一个类的<clinit>()方法在多线程环境中被正确地加锁、同步</font>，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的`<clinit>()`方法，其他线程都需要阻塞等待，直到活动线程执行`<clinit>()`方法完毕。

正是**因为函数`<clinit>()`带锁线程安全的**，因此，如果在一个类的`<clinit>()`方法中有耗时很长的操作，就可能造成多个线程阻塞，引发**死锁**。并且这种死锁是很难发现的，因为看起来它们并没有可用的锁信息。

如果之前的线程成功加载了类，则等在队列中的线程就没有机会再执行`<clinit>()`方法了。那么，当需要使用这个类时，虚拟机会直接返回给它已经准备好的信息。

**案例：**

```java
package com.atguigu.java;

/**
 * @author shkstart
 * @create 12:31
 */
class StaticA {
    static {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }
        try {
            Class.forName("com.atguigu.java.StaticB");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        System.out.println("StaticA init OK");
    }
}

class StaticB {
    static {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }
        try {
            Class.forName("com.atguigu.java.StaticA");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        System.out.println("StaticB init OK");
    }
}

public class StaticDeadLockMain extends Thread {
    private char flag;

    public StaticDeadLockMain(char flag) {
        this.flag = flag;
        this.setName("Thread" + flag);
    }

    @Override
    public void run() {
        try {
            Class.forName("com.atguigu.java.Static" + flag);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        System.out.println(getName() + " over");
    }

    public static void main(String[] args) throws InterruptedException {
        StaticDeadLockMain loadA = new StaticDeadLockMain('A');
        loadA.start();
        StaticDeadLockMain loadB = new StaticDeadLockMain('B');
        loadB.start();
    }
}
```

执行结果：一直处于等待状态

![image-20241212145506111](【尚硅谷】JVM-类的加载篇/675a88ca5c12f.webp)

使用 `jps` 及 `jstack` 命令查看线程运行情况：

![image-20241212145609399](【尚硅谷】JVM-类的加载篇/675a8908cb25f.webp)

虽然未检测出死锁，显示接口处于RUNNABLE状态，但处于循环等待中。

### 1.4.5、类的初始化情况（\<clinit>）：主动使用vs被动使用

> 面试题
>
> - 类加载的时机 （百度）
> - Class的forName("Java.lang.String")和Class的getClassLoader()的loadClass("Java.lang.String")有什么区别？  （百度）
> - 哪些情况会触发类的加载？ （京东）

**类的加载 = 装载 + 链接（①②③）+ 初始化**

#### 1.4.5.1、主动使用的情况

**Class只有在必须要首次使用的时候才会被装载，Java虚拟机不会无条件地装载Class类型。Java虚拟机规定，一个类或接口在初次使用前，必须要进行初始化。这里指的“使用”，是指主动使用。**

主动使用只有下列几种情况：（即：如果出现如下的情况，则会对类进行初始化操作。而初始化操作之前的加载、验证、准备已经完成。）

1. 当创建一个类的实例时，比如使用new关键字，或者通过反射、克隆、反序列化。
2. 当调用类的静态方法时，即当使用了字节码invokestatic指令。
3. 当使用类、**接口**的静态字段时**(final修饰特殊考虑)**，比如，使用**getstatic**或者**putstatic**指令。
4. 当使用**java.lang.reflect**包中的方法反射类的方法时。比如：**Class.forName("com.atguigu.java.Test")**
5. 当初始化子类时，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
6. **如果一个接口定义了default方法，那么直接实现或者间接实现该接口的类的初始化，该接口要在其之前被初始化。**
7. 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
8. 当初次调用 MethodHandle 实例时，初始化该 MethodHandle 指向的方法所在的类。（涉及解析REF_getStatic、REF_putStatic、REF_invokeStatic方法句柄对应的类）

**针对5，说明：**
当Java虚拟机初始化一个类时，要求它的所有父类都已经被初始化，但是这条规则并不适用于接口。

- 在初始化一个类时，并不会先初始化它所实现的接口
- 在初始化一个接口时，并不会先初始化它的父接口

**因此，一个父接口并不会因为它的子接口或者实现类的初始化而初始化。只有当程序首次使用特定接口的静态字段时，才会导致该接口的初始化。**

**针对7，说明：**

JVM启动的时候通过引导类加载器加载一个初始类。这个类在调用public static void main(String[])方法之前被链接和初始化。这个方法的执行将依次导致所需的类的加载，链接和初始化。

#### 1.4.5.2、被动使用的情况

> 被动的使用，意味着不需要执行初始化环节，意味着没有`<clinit>()`的调用。

除了以上的情况属于主动使用，其他的情况均属于被动使用。**被动使用不会引起类的初始化**。
也就是说：**并不是在代码中出现的类，就一定会被加载或者初始化。如果不符合主动使用的条件，类就不会初始化**。

1. 当访问一个静态字段时，只有真正声明这个字段的类才会被初始化。
当通过子类引用父类的静态变量，不会导致子类初始化
2. 通过数组定义类引用，不会触发此类的初始化。
3. 引用常量不会触发此类或接口的初始化。因为常量在链接阶段(准备)就已经被显式赋值了。
4. 调用ClassLoader类的`loadClass()`方法加载一个类，并不是对类的主动使用，不会导致类的初始化。

#### 1.4.5.3、-XX:+TraceClassLoading

如果针对代码，设置参数`-XX:+TraceClassLoading`，可以追踪类的加载信息并打印出来。

### 1.4.6、面试题

**面试题1：**

```java
class Root{
 static{
   System.out.println("Root的静态初始化块");
 }
 {
   System.out.println("Root的普通初始化块");
 }
 public Root(){
   System.out.println("Root的无参数的构造器");
 }
}
class Mid extends Root{
 static{
   System.out.println("Mid的静态初始化块");
 }
 {
   System.out.println("Mid的普通初始化块");
 }
 public Mid(){
   System.out.println("Mid的无参数的构造器");
 }
 public Mid(String msg){
   //通过this调用同一类中重载的构造器
   this();
   System.out.println("Mid的带参数构造器，其参数值："
    + msg);
 }
}
class Leaf extends Mid{
 static{
   System.out.println("Leaf的静态初始化块");
 }
 {
   System.out.println("Leaf的普通初始化块");
 }
 public Leaf(){
   //通过super调用父类中有一个字符串参数的构造器
   super("尚硅谷");
   System.out.println("Leaf的构造器");
 }
}
public class LeafTest{
 public static void main(String[] args){
   new Leaf(); 
   System.out.println();
   new Leaf();
 }
}
```

**面试题2：**

```java
public class Test {
    static int x, y, z;

    static {
        int x = 5;
        x--;
    }

    static {
        x--;
    }

    public static void main(String[] args) {
        System.out.println("x=" + x);
        z--;
        method();
        System.out.println("result:" + (z + y + ++z));
    }

    public static void method() {
        y = z++ + ++z;
    }
}
```

**面试题3：**

```java
public class Test03 {
    public static void main(String[] args) {
        Father f = new Son();
        System.out.println(f.x);
    }
}
class Father{
    int x = 10;
    public Father(){
        this.print();
        x = 20;
    }
    public void print(){
        System.out.println("Father.x = " + x);
    }
}
class Son extends Father{
    int x = 30;
    public Son(){
        this.print();
        x = 40;
    }
    public void print(){
        System.out.println("Son.x = " + x);
    }
}
```

**面试题4：**

```java
public class T {
    public static int k = 0;
    public static T t1 = new T("t1");
    public static T t2 = new T("t2");
    public static int i = print("i");
    public static int n = 99;

    public int j = print("j");

    {
        print("构造块");
    }

    static {
        print("静态块");
    }

    public T(String str) {
        System.out.println((++k) + ":" + str + "  i=" + i + "  n=" + n);
        ++n;
        ++i;
    }

    public static int print(String str) {
        System.out.println((++k) + ":" + str + "  i=" + i + "  n=" + n);
        ++n;
        return ++i;
    }

    public static void main(String[] args) {

    }
}
```

## 1.5、过程四：类的Using(使用)

任何一个类型在使用之前都必须经历过完整的加载、链接和初始化3个类加载步骤。一旦一个类型成功经历过这3个步骤之后，便“万事俱备，只欠东风”，就等着开发者使用了。

开发人员可以在程序中访问和调用它的静态类成员信息（比如：静态字段、静态方法），或者使用new关键字为其创建对象实例。

## 1.6、过程五：类的Unloading(卸载)

### 1.6.1、类、类的加载器、类的实例之间的关系

在类加载器的内部实现中，用一个Java集合来存放所加载类的引用。另一方面，一个Class对象总是会引用它的类加载器，调用Class对象的getClassLoader()方法，就能获得它的类加载器。由此可见，代表某个类的Class实例与其类的加载器之间为双向关联关系。

一个类的实例总是引用代表这个类的Class对象。在Object类中定义了getClass()方法，这个方法返回代表对象所属类的Class对象的引用。此外，所有的Java类都有一个静态属性class，它引用代表这个类的Class对象。

![image-20241212154114308](【尚硅谷】JVM-类的加载篇/675a939ab1524.webp)

### 1.6.2、何种情况类会被卸载？

**一个类何时结束生命周期，取决于代表它的Class对象何时结束生命周期。**

当Sample类被加载、链接和初始化后，它的生命周期就开始了。当代表Sample类的Class对象不再被引用，即不可触及时，Class对象就会结束生命周期，Sample类在方法区内的数据也会被卸载，从而结束Sample类的生命周期。



参考上图，loader1变量和obj变量间接引用代表Sample类的Class对象，而objClass变量则直接引用它。

如果程序运行过程中，将上图左侧三个引用变量都置为null，此时Sample对象结束生命周期，MyClassLoader对象结束生命周期，代表Sample类的Class对象也结束生命周期，Sample类在方法区内的二进制数据被卸载。

当再次有需要时，会检查Sample类的Class对象是否存在**，如果存在会直接使用，不再重新加载**；如果不存在Sample类会被重新加载，在Java虚拟机的堆区会生成一个新的代表Sample类的Class实例(可以通过哈希码查看是否是同一个实例)。

### 1.6.3、类卸载在实际生产中的情况如何？

(1) 启动类加载器加载的类型在整个运行期间是不可能被卸载的(jvm和jls规范)

(2) 被系统类加载器和扩展类加载器加载的类型在运行期间不太可能被卸载，因为系统类加载器实例或者扩展类的实例基本上在整个运行期间总能直接或者间接的访问的到，其达到unreachable的可能性极小。

(3) 被开发者自定义的类加载器实例加载的类型只有在很简单的上下文环境中才能被卸载，而且一般还要借助于强制调用虚拟机的垃圾收集功能才可以做到。可以预想，稍微复杂点的应用场景中(比如：很多时候用户在开发自定义类加载器实例的时候采用缓存的策略以提高系统性能)，被加载的类型在运行期间也是几乎不太可能被卸载的(至少卸载的时间是不确定的)。

综合以上三点，一个已经加载的类型被卸载的几率很小至少被卸载的时间是不确定的。同时我们可以看的出来，开发者在开发代码时候，不应该对虚拟机的类型卸载做任何假设的前提下，来实现系统中的特定功能。

### 1.6.3、拓展：方法区的垃圾回收

**方法区的垃圾收集主要回收两部分内容：常量池中废弃的常量和不再使用的类型。**

HotSpot虚拟机对常量池的回收策略是很明确的，只要常量池中的常量没有被任何地方引用，就可以被回收。

判定一个常量是否“废弃”还是相对简单，而要判定一个类型是否属于“不再被使用的类”的条件就比较苛刻了。需要同时满足下面三个条件： 

- **该类所有的实例都已经被回收。也就是Java堆中不存在该类及其任何派生子类的实例。** 
- **加载该类的类加载器已经被回收。这个条件除非是经过精心设计的可替换类加载器的场景，如OSGi、JSP的重加载等，否则通常是很难达成的。** 
- **该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。**

Java虚拟机被允许对满足上述三个条件的无用类进行回收，这里说的仅仅是“被允许”，而并不是和对象一样，没有引用了就必然会回收。

# 2、类的加载器

> 面试题：什么是类加载器，类加载器有哪些?

## 2.1、作用

类加载器是 JVM 执行类加载机制的前提。

**ClassLoader的作用：**

ClassLoader是Java的核心组件，所有的Class都是由ClassLoader进行加载的，ClassLoader负责通过各种方式将Class信息的二进制数据流读入JVM内部，转换为一个与目标类对应的`java.lang.Class`对象实例。然后交给Java虚拟机进行链接、初始化等操作。

因此，**ClassLoader在整个装载（Loading）阶段，只能影响到类的加载**，而无法通过ClassLoader去改变类的链接和初始化行为。至于它是否可以运行，则由Execution Engine决定。

![image-20241212155946924](【尚硅谷】JVM-类的加载篇/675a97f33856b.webp)

类加载器最早出现在Java1.0版本中，那个时候只是单纯地为了满足Java Applet应用而被研发出来。但如今类加载器却在OSGi、字节码加解密领域大放异彩。这主要归功于Java虚拟机的设计者们当初在设计类加载器的时候，并没有考虑将它绑定在JVM内部，这样做的好处就是能够更加灵活和动态地执行类加载操作。

## 2.2、类的加载分类：显式加载与隐式加载

class文件的显式加载与隐式加载的方式是指JVM加载class文件到内存的方式。

- 显式加载：指的是在代码中通过调用ClassLoader加载class对象，如直接使用**Class.forName(name)**或**this.getClass().getClassLoader().loadClass()**加载class对象。
- 隐式加载：则是不直接在代码中调用ClassLoader的方法加载class对象，而是通过虚拟机自动加载到内存中，如在加载某个类的class文件时，该类的class文件中引用了另外一个类的对象，此时额外引用的类将通过JVM自动加载到内存中。

在日常开发以上两种方式一般会混合使用。

## 2.3、类加载机制的必要性

一般情况下，Java开发人员并不需要在程序中显式地使用类加载器，但是了解类加载器的加载机制却显得至关重要。从以下几个方面说：

- 避免在开发中遇到 java.lang.ClassNotFoundException异常或java.lang.NoClassDefFoundError异常时，手足无措。只有了解类加载器的加载机制才能够在出现异常的时候快速地根据错误异常日志定位问题和解决问题
- 需要支持类的动态加载或需要对编译后的字节码文件进行加解密操作时，就需要与类加载器打交道了。
- 开发人员可以在程序中编写自定义类加载器来重新定义类的加载规则，以便实现一些自定义的处理逻辑。

## 2.4、加载的类是唯一的吗

**何为类的唯一性？**

对于任意一个类，都需要由加载它的类加载器和这个类本身一同确认其在Java虚拟机中的唯一性。每一个类加载器，都拥有一个独立的类名称空间：比较两个类是否相等，只有在这两个类是由同一个类加载器加载的前提下才有意义。否则，即使这两个类源自同一个Class文件，被同一个虚拟机加载，只要加载他们的类加载器不同，那这两个类就必定不相等。

**命名空间**

- 每个类加载器都有自己的命名空间，命名空间由该加载器及所有的父加载器所加载的类组成
- 在同一命名空间中，不会出现类的完整名字（包括类的包名）相同的两个类
- 在不同的命名空间中，有可能会出现类的完整名字（包括类的包名）相同的两个类

## 2.5、类加载机制的基本特征

通常类加载机制有三个基本特征：

- 双亲委派模型。但不是所有类加载都遵守这个模型，有的时候，启动类加载器所加载的类型，是可能要加载用户代码的，比如JDK内部的ServiceProvider/ServiceLoader机制，用户可以在标准API框架上，提供自己的实现，JDK也需要提供些默认的参考实现。例如，Java 中JNDI、JDBC、文件系统、Cipher等很多方面，都是利用的这种机制，这种情况就不会用双亲委派模型去加载，而是利用所谓的上下文加载器。

- 可见性。**子类加载器可以访问父加载器加载的类型**，但是反过来是不允许的。不然，因为缺少必要的隔离，我们就没有办法利用类加载器去实现容器的逻辑。

- 单一性。由于父加载器的类型对于子加载器是可见的，所以父加载器中加载过的类型，就不会在子加载器中重复加载。但是注意，类加载器“邻居”间，同一类型仍然可以被加载多次，因为互相并不可见。

# 3、类的加载器分类与测试

## 3.1、类加载器的分类说明

JVM支持两种类型的类加载器，分别为**引导类加载器（Bootstrap ClassLoader）**和**自定义类加载器（User-Defined ClassLoader）**。

从概念上来讲，自定义类加载器一般指的是程序中由开发人员自定义的一类类加载器，但是Java虚拟机规范却没有这么定义，而是将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器。

无论类加载器的类型如何划分，在程序中我们最常见的类加载器结构主要是如下情况：

![image-20241212161615689](【尚硅谷】JVM-类的加载篇/675a9bcfbc914.webp)

## 3.2、子-父 类加载器的关系？

- 除了顶层的启动类加载器外，其余的类加载器都应当有自己的“父类”加载器。 

- 不同类加载器看似是继承（Inheritance）关系，实际上是包含关系。在下层加载器中，包含着上层加载器的引用。

  ```java
  class ClassLoader{
       ClassLoader parent; //父类加载器
   
       public ClassLoader(ClassLoader parent){ 
            this.parent = parent;
       }
  }
  class ParentClassLoader extends ClassLoader{
       public ParentClassLoader(ClassLoader parent){
            super(parent);
       }
  }
  class ChildClassLoader extends ClassLoader{
       public ChildClassLoader(ClassLoader parent){ //parent = new ParentClassLoader();
            super(parent);
       }
  }
  ```

## 3.3、具体类的加载器介绍

### 3.3.1、启动类加载器/引导类加载器（Bootstrap ClassLoader）

- 这个类加载使用C/C++语言实现的，嵌套在JVM内部。
- 它用来加载Java的核心库（JAVA_HOME/jre/lib/rt.jar或sun.boot.class.path路径下的内容）。用于提供JVM自身需要的类。
- 并不继承自`java.lang.ClassLoader`，没有父加载器。
- 出于安全考虑，Bootstrap启动类加载器只加载包名为**java、javax、sun**等开头的类
- 加载扩展类和应用程序类加载器，并指定为他们的父类加载器。

使用`-XX:+TraceClassLoading`参数得到：

![image-20241212162329499](【尚硅谷】JVM-类的加载篇/675a9d8116df2.webp)

![image-20241212162414792](【尚硅谷】JVM-类的加载篇/675a9dae22ff6.webp)



**启动类加载器是使用C++编写的**

- C/C++：指针函数&函数指针、C++支持多继承
- Java：由C++演变而来，(C++)--版，单继承

### 3.3.2、扩展类加载器 (Extension ClassLoader)

- Java语言编写，由**sun.misc.Launcher$ExtClassLoader**实现。
- 继承于ClassLoader类
- 父类加载器为启动类加载器
- 从**java.ext.dirs**系统属性所指定的目录中加载类库，或从JDK的安装目录的**jre/lib/ext**子目录下加载类库。如果用户创建的JAR放在此目录下，也会自动由扩展类加载器加载。

![image-20241212162815526](【尚硅谷】JVM-类的加载篇/675a9e9fb9c88.webp)

### 3.3.3、系统类加载器/应用程序类加载器（AppClassLoader）

- java语言编写，由**sun.misc.Launcher$AppClassLoader**实现
- 继承于ClassLoader类
- 父类加载器为扩展类加载器
- 它负责加载环境变量**classpath**或系统属性 **java.class.path** 指定路径下的类库 
- **应用程序中的类加载器默认是系统类加载器。**
- 它是用户自定义类加载器的默认父加载器
- 通过ClassLoader的getSystemClassLoader()方法可以获取到该类加载器

## 3.4、用户自定义类加载器

- 在Java的日常应用程序开发中，类的加载几乎是由上述3种类加载器相互配合执行的。在必要时，我们还可以自定义类加载器，来定制类的加载方式。
- 体现Java语言强大生命力和巨大魅力的关键因素之一便是,Java开发者可以自定义类加载器来实现类库的动态加载，加载源可以是本地的JAR包，也可以是网络上的远程资源。
- **通过类加载器可以实现非常绝妙的插件机制**，这方面的实际应用案例举不胜举。例如，著名的OSGI组件框架，再如Eclipse的插件机制。类加载器为应用程序提供了一种动态增加新功能的机制，这种机制无须重新打包发布应用程序就能实现。
- 同时，**自定义加载器能够实现应用隔离**，例如 Tomcat，Spring等中间件和组件框架都在内部实现了自定义的加载器，并通过自定义加载器隔离不同的组件模块。这种机制比C/C++程序要好太多，想不修改C/C++程序就能为其新增功能，几乎是不可能的，仅仅一个兼容性便能阻挡住所有美好的设想。
- 所有用户自定义类加载器通常需要继承于抽象类**java.lang.ClassLoader**。

## 3.5、测试不同的类加载器

每个Class对象都会包含一个定义它的ClassLoader的一个引用。

**获取ClassLoader的途径：**

- 获得当前类的ClassLoader：`clazz.getClassLoader()`
- 获得当前线程上下文的ClassLoader：`Thread.currentThread().getContextClassLoader()`
- 获得系统的ClassLoader：`ClassLoader.getSystemClassLoader()`

**说明：**

站在程序的角度看，引导类加载器与另外两种类加载器（系统类加载器和扩展类加载器）并不是同一个层次意义上的加载器，引导类加载器是使用C++语言编写而成的，而另外两种类加载器则是使用Java语言编写而成的。由于引导类加载器压根儿就不是一个Java类，因此在Java程序中只能打印出空值。

数组类的Class对象，不是由类加载器去创建的，而是在Java运行期JVM根据需要自动创建的。对于数组类的类加载器来说，是通过Class.getClassLoader()返回的，与数组当中元素类型的类加载器是一样的；如果数组当中的元素类型是基本数据类型，数组类是没有类加载器的。



```java
String[] strArr = new String[6];
System.out.println(strArr.getClass().getClassLoader());
// 运行结果：null

ClassLoaderTest[] test = new ClassLoaderTest[1];
System.out.println(test.getClass().getClassLoader());
// 运行结果：sun.misc.Launcher$AppClassLoader@18b4aac2

int[] ints = new int[2];
System.out.println(ints.getClass().getClassLoader());
// 运行结果：null
```



# 4、ClassLoader源码剖析

> 面试题：深入分析ClassLoader

## 4.1、ClassLoader与现有类加载器的关系

ClassLoader是一个抽象类。如果我们给定了一个类的二进制名称，类加载器应尝试去定位或生成构成定义类的数据。一种典型的策略是将给定的二进制名称转换为文件名，然后去文件系统中读取这个文件名所对应的class文件。 

![image-20241212164134539](【尚硅谷】JVM-类的加载篇/675aa1beafe39.webp)

## 4.2、ClassLoader的主要方法

**抽象类ClassLoader的主要方法(内部没有抽象方法)：**

```java
public final ClassLoader getParent()
```

返回该类加载器的超类加载器



```java
public Class<?> loadClass(String name) throws ClassNotFoundException
```

加载名称为name的类，返回结果为java.lang.Class类的实例。如果找不到类，则返回 ClassNotFoundException 异常。**该方法中的逻辑就是双亲委派模式的实现**。



```java
protected Class<?> findClass(String name) throws ClassNotFoundException
```

查找二进制名称为name的类，返回结果为java.lang.Class类的实例。这是一个受保护的方法，JVM鼓励我们重写此方法，需要自定义加载器遵循双亲委托机制，该方法会在检查完父类加载器之后被loadClass()方法调用。

> 在JDK1.2之前，在自定义类加载时，总会去继承ClassLoader类并重写loadClass方法，从而实现自定义的类加载类。但是在JDK1.2之后已不再建议用户去覆盖loadClass()方法，而是建议把自定义的类加载逻辑写在findClass()方法中，从前面的分析可知，findClass()方法是在loadClass()方法中被调用的，当loadClass()方法中父加载器加载失败后，则会调用自己的findClass()方法来完成类加载，这样就可以保证自定义的类加载器也符合双亲委托模式。
> 需要注意的是ClassLoader类中并没有实现findClass()方法的具体代码逻辑，取而代之的是抛出ClassNotFoundException异常，同时应该知道的是findClass方法通常是和defineClass方法一起使用的。一般情况下，在自定义类加载器时，会直接覆盖ClassLoader的findClass()方法并编写加载规则，取得要加载类的字节码后转换成流，然后调用defineClass()方法生成类的Class对象。



```java
protected final Class<?> defineClass(String name, byte[] b, int off, int len)
```

根据给定的字节数组b转换为Class的实例，off和len参数表示实际Class信息在byte数组中的位置和长度，其中byte数组b是ClassLoader从外部获取的。这是受保护的方法，只有在自定义ClassLoader子类中可以使用。

> defineClass()方法是用来将byte字节流解析成JVM能够识别的Class对象(ClassLoader中已实现该方法逻辑)，通过这个方法不仅能够通过class文件实例化class对象，也可以通过其他方式实例化class对象，如通过网络接收一个类的字节码，然后转换为byte字节流创建对应的Class对象。
> defineClass()方法通常与findClass()方法一起使用，一般情况下，在自定义类加载器时，会直接覆盖ClassLoader的findClass()方法并编写加载规则，取得要加载类的字节码后转换成流，然后调用defineClass()方法生成类的Class对象

简单举例：

```java
protected Class<?> findClass(String name) throws ClassNotFoundException {
    // 获取类的字节数组
    byte[] classData = getClassData(name);
    if (classData == null) {
        throw new ClassNotFoundException();
    } else {
        //使用defineClass生成class对象
        return defineClass(name, classData, 0, classData.length);
    }
}
```



```java
protected final void resolveClass(Class<?> c)
```

链接指定的一个Java类。使用该方法可以使用类的Class对象创建完成的同时也被解析。前面我们说链接阶段主要是对字节码进行验证，为类变量分配内存并设置初始值同时将字节码文件中的符号引用转换为直接引用。



```java
protected final Class<?> findLoadedClass(String name)
```

查找名称为name的已经被加载过的类，返回结果为java.lang.Class类的实例。这个方法是final方法，无法被修改。



```java
private final ClassLoader parent;
```

它也是一个ClassLoader的实例，这个字段所表示的ClassLoader也称为这个ClassLoader的双亲。在类加载的过程中,ClassLoader可能会将某些请求交予自己的双亲处理。

## 4.3、SecureClassLoader 与 URLClassLoader

接着SecureClassLoader扩展了 ClassLoader，新增了几个与使用相关的代码源(对代码源的位置及其证书的验证)和权限定义类验证(主要指对class源码的访问权限)的方法，一般我们不会直接跟这个类打交道，更多是与它的子类URLClassLoader有所关联。

前面说过，ClassLoader是一个抽象类，很多方法是空的没有实现，比如 findClass()、findResource()等。而URLClassLoader这个实现类为这些方法提供了具体的实现。并新增了URLClassPath类协助取得Class字节码流等功能。**在编写自定义类加载器时，如果没有太过于复杂的需求，可以直接继承URLClassLoader类**，这样就可以避免自己去编写findClass()方法及其获取字节码流的方式，使自定义类加载器编写更加简洁。

![image-20241212165435007](【尚硅谷】JVM-类的加载篇/675aa4ca9a7a9.webp)

## 4.4、ExtClassLoader 与 AppClassLoader

了解完URLClassLoader后接着看看剩余的两个类加载器，即拓展类加载器ExtClassLoader和系统类加载器AppClassLoader，这两个类都继承自URLClassLoader，是sun.misc.Launcher的静态内部类。sun.misc.Launcher主要被系统用于启动主应用程序，ExtClassLoader和AppClassLoader都是由sun.misc.Launcher创建的，其类主要类结构如下：

![image-20241212165538172](【尚硅谷】JVM-类的加载篇/675aa5097e2b9.webp)

我们发现ExtClassLoader并没有重写loadClass()方法，这足矣说明其遵循双亲委派模式，而AppClassLoader重载了loadClass()方法，但最终调用的还是父类loadClass()方法，因此依然遵守双亲委派模式。



## 4.5、Class.forName()与ClassLoader.loadClass()对比

- **Class.forName()**：是一个静态方法,最常用的是Class.forName(String className);根据传入的类的全限定名返回一个 Class 对象。**该方法在将 Class 文件加载到内存的同时,会执行类的初始化**。如: 

  ```java
  Class.forName("com.atguigu.java.HelloWorld");
  ```

- **ClassLoader.loadClass()**：这是一个实例方法,需要一个 ClassLoader 对象来调用该方法。**该方法将 Class 文件加载到内存时,并不会执行类的初始化,直到这个类第一次使用时才进行初始化**。该方法因为需要得到一个 ClassLoader 对象,所以可以根据需要指定使用哪个类加载器。  如：

  ```java
  ClassLoader cl=.......;    
  cl.loadClass("com.atguigu.java.HelloWorld");
  ```

  

# 5、自定义类的加载器

## 5.1、为什么要自定义类的加载器？

1、隔离加载类

在某些框架内进行中间件与应用的模块隔离，把类加载到不同的环境。比如：阿里内某容器框架通过自定义类加载器确保应用中依赖的jar包不会影响到中间件运行时使用的jar包。再比如：Tomcat这类Web应用服务器，内部自定义了好几种类加载器，用于隔离同一个Web应用服务器上的不同应用程序。 （类的仲裁-->类冲突）

2、修改类加载的方式

类的加载模型并非强制，除Bootstrap外，其他的加载并非一定要引入，或者根据实际情况在某个时间点进行按需进行动态加载

3、扩展加载源
比如从数据库、网络、甚至是电视机机顶盒进行加载

4、防止源码泄漏
Java代码容易被编译和篡改，可以进行编译加密。那么类加载也需要自定义，还原加密的字节码。

## 5.2、应用场景有哪些？

**常见的应用场景**

实现类似进程内隔离，类加载器实际上用作不同的命名空间，以提供类似容器、模块化的效果。例如，两个模块依赖于某个类库的不同版本，**如果分别被不同的容器加载，就可以互不干扰**。这个方面的集大成者是Java EE和OSGI、JPMS等框架。

应用需要从不同的数据源获取类定义信息，例如网络数据源，而不是本地文件系统。或者是需要自己操纵字节码，动态修改或者生成类型。

注意：
在一般情况下，使用不同的类加载器去加载不同的功能模块，会提高应用程序的安全性。**但是，如果涉及Java类型转换，则加载器反而容易产生不美好的事情**。在做Java类型转换时，只有两个类型都是由同一个加载器所加载，才能进行类型转换，否则转换时会发生异常。

## 5.3、两种实现方式

> 面试题：手写一个类加载器Demo 

用户通过定制自己的类加载器，这样可以重新定义类的加载规则，以便实现一些自定义的处理逻辑。

Java提供了抽象类java.lang.ClassLoader，所有用户自定义的类加载器都应该继承ClassLoader类。

在自定义 ClassLoader 的子类时候，我们常见的会有两种做法：

- 方式一：重写loadClass()方法
- 方式二：重写findClass()方法 -->推荐



**两种实现方式对比**
这两种方法本质上差不多，毕竟loadClass()也会调用findClass()，但是从逻辑上讲我们最好不要直接修改loadClass()的内部逻辑。建议的做法是只在findClass()里重写自定义类的加载方法，根据参数指定类的名字，返回对应的Class对象的引用。 

- loadClass()这个方法是实现双亲委派模型逻辑的地方，擅自修改这个方法会导致模型被破坏，容易造成问题。**因此我们最好是在双亲委派模型框架内进行小范围的改动，不破坏原有的稳定结构**。同时，也避免了自己重写 loadClass()方法的过程中必须写双亲委托的重复代码，从代码的复用性来看，不直接修改这个方法始终是比较好的选择。
- 当编写好自定义类加载器后，便可以在程序中调用loadClass() 方法来实现类加载操作。

**说明**

其父类加载器是系统类加载器

JVM中的所有类加载都会使用java.lang.ClassLoader.loadClass(String)接口(自定义类加载器并重写java.lang.ClassLoader.loadClass(String)接口的除外)，连JDK的核心类库也不能例外。



# 6、相关机制

## 6.1、双亲委派机制

> 面试题：
>
> - 讲一下双亲委派模型，以及其优点 ？
> - 双亲委派机制及使用原因 
>

### 6.1.1、定义与本质？

类加载器用来把类加载到Java虚拟机中。从JDK1.2版本开始，类的加载过程采用双亲委派机制，这种机制能更好地保证Java平台的安全。

**定义**

如果一个类加载器在接到加载类的请求时，它首先不会自己尝试去加载这个类，而是把这个请求任务委托给父类加载器去完成，依次递归，如果父类加载器可以完成类加载任务，就成功返回。只有父类加载器无法完成此加载任务时，才自己去加载。

![image-20241212200412221](【尚硅谷】JVM-类的加载篇/675ad13c53d4f.webp)



![image-20241212200443658](【尚硅谷】JVM-类的加载篇/675ad15b06052.webp)



![image-20241212200800209](【尚硅谷】JVM-类的加载篇/675ad21f5ea85.webp)



### 6.1.2、源码分析

**源码分析**

双亲委派机制在java.lang.ClassLoader.loadClass(String,boolean)接口中体现。该接口的逻辑如下：
(1) 先在当前加载器的缓存中查找有无目标类，如果有，直接返回。
(2) 判断当前加载器的父加载器是否为空，如果不为空，则调用parent.loadClass(name, false)接口进行加载。
(3) 反之，如果当前加载器的父类加载器为空，则调用findBootstrapClassOrNull(name)接口，让引导类加载器进行加载。
(4) 如果通过以上3条路径都没能成功加载，则调用findClass(name)接口进行加载。该接口最终会调用java.lang.ClassLoader接口的defineClass系列的native接口加载目标Java类。
双亲委派的模型就隐藏在这第2和第3步中。

**举例**
假设当前加载的是java.lang.Object这个类，很显然，该类属于JDK中核心得不能再核心的一个类，因此一定只能由引导类加载器进行加载。当JVM准备加载javaJang.Object时，JVM默认会使用系统类加载器去加载，按照上面4步加载的逻辑，在第1步从系统类的缓存中肯定查找不到该类，于是进入第2步。由于从系统类加载器的父加载器是扩展类加载器，于是扩展类加载器继续从第1步开始重复。由于扩展类加载器的缓存中也一定査找不到该类，因此进入第2步。扩展类的父加载器是null,因此系统调用findClass(String), 最终通过引导类加载器进行加载。

**思考**
如果在自定义的类加载器中重写java.lang.ClassLoader.loadClass(String)或 java.lang.ClassLoader.loadClass(String, boolean)方法,抹去其中的双亲委派机制,仅保留上面这4步中的第1步与第4步，那么是不是就能够加载核心类库了呢？
这也不行！ 因为JDK还为核心类库提供了一层保护机制。不管是自定义的类加载器，还是系统类加载器抑或扩展类加载器，最终都必须调用 java.lang.ClassLoader.defineClass(String, byte[], int, int, ProtectionDomain)方法，而该方法会执行preDefineClass()接口，该接口中提供了对JDK核心类库的保护。



### 6.1.3、优势与劣势

**双亲委派机制优势**

- 避免类的重复加载，确保一个类的全局唯一性
  **Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关系可以避免类的重复加载**，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。
- 保护程序安全，防止核心API被随意篡改

**双亲委托模式的弊端**
检查类是否加载的委托过程是单向的，这个方式虽然从结构上说比较清晰，使各个ClassLoader的职责非常明确，但是同时会带来一个问题，即顶层的ClassLoader无法访问底层的ClassLoader所加载的类。

通常情况下，启动类加载器中的类为系统核心类，包括一些重要的系统接口，而在应用类加载器中，为应用类。按照这种模式，**应用类访问系统类自然是没有问题，但是系统类访问应用类就会出现问题**。比如在系统类中提供了一个接口，该接口需要在应用类中得以实现，该接口还绑定一个工厂方法，用于创建该接口的实例，而接口和工厂方法都在启动类加载器中。这时，就会出现该工厂方法无法创建由应用类加载器加载的应用实例的问题。

**结论**
**由于Java虚拟机规范并没有明确要求类加载器的加载机制一定要使用双亲委派模型，只是建议采用这种方式而已**。
**比如在Tomcat中，类加载器所采用的加载机制就和传统的双亲委派模型有一定区别，当缺省的类加载器接收到一个类的加载任务时，首先会由它自行加载，当它加载失败时，才会将类的加载任务委派给它的超类加载器去执行，这同时也是Servlet规范推荐的一种做法。**

### 6.1.4、破坏双亲委派机制及举例

> 面试题：可以打破双亲委派么，怎么打破。

这里，我们使用了“被破坏”这个词来形容上述不符合双亲委派模型原则的行为，但**这里“被破坏”并不一定是带有贬义的。只要有明确的目的和充分的理由，突破旧有原则无疑是一种创新**。

正如：破坏双亲委派机制3的OSGi中的类加载器的设计不符合传统的双亲委派的类加载器架构，且业界对其为了实现热部署而带来的额外的高复杂度还存在不少争议，但对这方面有了解的技术人员基本还是能达成一个共识，认为**OSGi中对类加载器的运用是值得学习的，完全弄懂了OSGi的实现，就算是掌握了类加载器的精粹**。

双亲委派模型并不是一个具有强制性约束的模型，而是Java设计者推荐给开发者们的类加载器实现方式。

在Java的世界中大部分的类加载器都遵循这个模型，但也有例外的情况，直到Java模块化出现为止，双亲委派模型主要出现过3次较大规模“被破坏”的情况。

#### 6.1.4.1、第一次破坏双亲委派机制

双亲委派模型的第一次“被破坏”其实发生在双亲委派模型出现之前——即JDK 1.2面世以前的“远古”时代。
由于双亲委派模型在JDK 1.2之后才被引入，但是类加载器的概念和抽象类java.lang.ClassLoader则在Java的第一个版本中就已经存在，面对已经存在的用户自定义类加载器的代码，Java设计者们引入双亲委派模型时不得不做出一些妥协，**为了兼容这些已有代码，无法再以技术手段避免loadClass()被子类覆盖的可能性**，只能在JDK1.2之后的java.lang.ClassLoader中添加一个新的protected方法findClass()，并引导用户编写的类加载逻辑时尽可能去重写这个方法，而不是在loadClass()中编写代码。上节我们已经分析过loadClass()方法，双亲委派的具体逻辑就实现在这里面，按照loadClass()方法的逻辑，如果父类加载失败，会自动调用自己的findClass()方法来完成加载，这样既不影响用户按照自己的意愿去加载类，又可以保证新写出来的类加载器是符合双亲委派规则的。



#### 6.1.4.2、第二次破坏双亲委派机制：线程上下文类加载器

双亲委派模型的第二次“被破坏”是由这个模型自身的缺陷导致的，双亲委派很好地解决了各个类加载器协作时基础类型的一致性问题**（越基础的类由越上层的加载器进行加载）**，基础类型之所以被称为“基础”，是因为它们总是作为被用户代码继承、调用的API存在，但程序设计往往没有绝对不变的完美规则，如果**有基础类型又要调用回用户的代码，那该怎么办呢？**

这并非是不可能出现的事情，一个典型的例子便是JNDI服务，JNDI现在已经是Java的标准服务，它的代码由启动类加载器来完成加载（在JDK 1.3时加入到rt.jar的），肯定属于Java中很基础的类型了。但JNDI存在的目的就是对资源进行查找和集中管理，它需要调用由其他厂商实现并部署在应用程序的ClassPath下的JNDI服务提供者接口（Service Provider Interface，SPI）的代码，现在问题来了，**启动类加载器是绝不可能认识、加载这些代码的，那该怎么办？**（SPI:在Java平台中，通常把核心类rt.jar中提供外部服务、可由应用层自行实现的接口称为SPI）

为了解决这个困境，Java的设计团队只好引入了一个不太优雅的设计：**线程上下文类加载器（Thread Context ClassLoader）**。这个类加载器可以通过java.lang.Thread类的setContextClassLoader()方法进行设置，如果创建线程时还未设置，它将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。

有了线程上下文类加载器，程序就可以做一些“舞弊”的事情了。JNDI服务使用这个线程上下文类加载器去加载所需的SPI服务代码，**这是一种父类加载器去请求子类加载器完成类加载的行为，这种行为实际上是打通了双亲委派模型的层次结构来逆向使用类加载器，已经违背了双亲委派模型的一般性原则**，但也是无可奈何的事情。Java中涉及SPI的加载基本上都采用这种方式来完成，例如JNDI、JDBC、JCE、JAXB和JBI等。不过，**当SPI的服务提供者多于一个的时候**，代码就只能根据具体提供者的类型来硬编码判断，为了消除这种极不优雅的实现方式，在JDK 6时，JDK提供了java.util.ServiceLoader类，以META-INF/services中的配置信息，辅以责任链模式，这才算是给SPI的加载提供了一种相对合理的解决方案。

![image-20241212203052903](【尚硅谷】JVM-类的加载篇/675ad77d11b58.webp)



#### 6.1.4.3、第三次破坏双亲委派机制

双亲委派模型的第三次“被破坏”是由于用户对程序动态性的追求而导致的。如：代码热替换（Hot Swap）、模块热部署（Hot Deployment）等

IBM公司主导的JSR-291（即OSGi R4.2）实现模块化热部署的关键是它自定义的类加载器机制的实现，每一个程序模块（OSGi中称为Bundle）都有一个自己的类加载器，当需要更换一个Bundle时，就把Bundle连同类加载器一起换掉以实现代码的热替换。在OSGi环境下，类加载器不再双亲委派模型推荐的树状结构，而是进一步发展为更加复杂的网状结构。

当收到类加载请求时，OSGi将按照下面的顺序进行类搜索：
**1）将以java.*开头的类，委派给父类加载器加载。**
**2）否则，将委派列表名单内的类，委派给父类加载器加载。**
3）否则，将Import列表中的类，委派给Export这个类的Bundle的类加载器加载。
4）否则，查找当前Bundle的ClassPath，使用自己的类加载器加载。
5）否则，查找类是否在自己的Fragment Bundle中，如果在，则委派给Fragment Bundle的类加载器加载。
6）否则，查找Dynamic Import列表的Bundle，委派给对应Bundle的类加载器加载。
7）否则，类查找失败。
说明：只有开头两点仍然符合双亲委派模型的原则，其余的类查找都是在平级的类加载器中进行的



**热替换的实现**

根据这个特点，可以用来模拟热替换的实现，基本思路如下图所示：

![image-20241212203456282](【尚硅谷】JVM-类的加载篇/675ad86f7394b.webp)



#### 6.1.4.4、Tomcat 类加载机制

> 面试题：什么是tomcat类加载机制？

Tomcat8 和 Tomcat6比较大的区别是 ：
Tomcat8可以通过配置 \<Loader delegate="true"/>表示遵循双亲委派机制。

Tomcat的类加载机制是违反了双亲委托原则的，对于一些未加载的非基础类，各个web应用自己的类加载器(WebAppClassLoader)会优先查看自己的仓库加载，加载不到时再交给commonClassLoader走双亲委托。

**Tomcat 如何实现自己独特的类加载机制？**

所以，Tomcat 是怎么实现的呢？牛逼的Tomcat团队已经设计好了。我们看看他们的设计图：

![image-20241212204033650](【尚硅谷】JVM-类的加载篇/675ad9c0b44b8.webp)

当tomcat启动时，会创建几种类加载器：

1、Bootstrap 引导类加载器 
加载JVM启动所需的类，以及标准扩展类（位于jre/lib/ext下）

2、System 系统类加载器 
加载tomcat启动的类，比如bootstrap.jar，通常在catalina.bat或者catalina.sh中指定。位于CATALINA_HOME/bin下。

![image-20241212204235393](【尚硅谷】JVM-类的加载篇/675ada3a645a3.webp)

3、CommonClassLoader、CatalinaClassLoader、SharedClassLoader和WebappClassLoader
这些是Tomcat自己定义的类加载器，它们分别加载/common/\*、/server/\*、/shared/\*（在tomcat 6之后已经合并到根目录下的lib目录下）和/WebApp/WEB-INF/*中的Java类库。其中WebApp类加载器和Jsp类加载器通常会存在多个实例，**每一个Web应用程序对应一个WebApp类加载器，每一个JSP文件对应一个Jsp类加载器**。

- commonLoader：Tomcat最基本的类加载器，加载路径中的class可以被Tomcat容器本身以及各个Webapp访问；（加载CATALINA_HOME/lib下的结构，比如servlet-api.jar）
- catalinaLoader：Tomcat容器私有的类加载器，加载路径中的class对于Webapp不可见；
- sharedLoader：各个Webapp共享的类加载器，加载路径中的class对于所有Webapp可见，但是对于Tomcat容器不可见；
- WebappClassLoader：单个Tomcat实例中各个Web应用程序私有的类加载器，加载路径中的class只对当前Webapp可见；(加载WEB-INF/lib和WEB-INF/classes下的结构）



从图中的委派关系中可以看出：
CommonClassLoader能加载的类都可以被Catalina ClassLoader和SharedClassLoader使用，从而实现了公有类库的共用，而CatalinaClassLoader和Shared ClassLoader自己能加载的类则与对方相互隔离。

WebAppClassLoader可以使用SharedClassLoader加载到的类，但各个WebAppClassLoader实例之间相互隔离。
而JasperLoader的加载范围仅仅是这个JSP文件所编译出来的那一个.class文件，**它出现的目的就是为了被丢弃：当Web容器检测到JSP文件被修改时，会替换掉目前的JasperLoader的实例，并通过再建立一个新的Jsp类加载器来实现JSP文件的HotSwap功能**。

**当应用需要到某个类时，则会按照下面的顺序进行类加载：**
1 使用bootstrap引导类加载器加载
2 使用system系统类加载器加载
3 使用应用类加载器在WEB-INF/classes中加载
4 使用应用类加载器在WEB-INF/lib中加载
5 使用common类加载器在CATALINA_HOME/lib中加载



![image-20241212204529640](【尚硅谷】JVM-类的加载篇/675adae8c96a7.webp)

好了，至此，我们已经知道了tomcat为什么要这么设计，以及是如何设计的，那么，tomcat 违背了java 推荐的双亲委派模型了吗？答案是：违背了。 我们前面说过：双亲委派模型要求除了顶层的启动类加载器之外，其余的类加载器都应当由自己的父类加载器加载。
很显然，tomcat 不是这样实现，tomcat 为了实现隔离性，没有遵守这个约定，每个webappClassLoader加载自己的目录下的class文件，不会传递给父类加载器。



**源码分析**

Tomcat中WebappClassLoader的源码分析：
首先findLoadedClass0()和findLoadedClass()分别从本地和父类加载器的缓存中查找当前要加载的类是否已经加载过了。之后为了避免上面提到的安全问题，Tomcat类加载器会将加载请求委派给系统类加载器。接下来根据delegate变量的设置，决定是先由自己加载，还是先由父类加载器去加载。

```java
public Class loadClass(String name, boolean resolve)
    throws ClassNotFoundException {
    Class clazz = null;
    // (0) 先从自己的缓存中查找，有则返回，无则继续
    clazz = findLoadedClass0(name);
    if (clazz != null) {
        if (resolve) resolveClass(clazz);
        return (clazz);
    }
    // (0.1) 再从parent的缓存中查找
    clazz = findLoadedClass(name);
    if (clazz != null) {
        if (resolve) resolveClass(clazz);
        return (clazz);
    }
    // (0.2) 缓存中没有，则首先使用system类加载器来加载
    clazz = system.loadClass(name);
    if (clazz != null) {
        if (resolve) resolveClass(clazz);
        return (clazz);

    }
    //判断是否需要先让parent代理
    boolean delegateLoad = delegate || filter(name);
    // (1) 先让parent加载，通常delegateLoad == false，即这一步不会执行

    if (delegateLoad) {
        ClassLoader loader = parent;
        if (loader == null)
            loader = system;
        clazz = loader.loadClass(name);
        if (clazz != null) {
            if (resolve) resolveClass(clazz);
            return (clazz);

        }

    }
    // (2) delegateLoad == false 或者 parent加载失败，调用自身的加载机制
    clazz = findClass(name);
    if (clazz != null) {
        if (resolve) resolveClass(clazz);
        return (clazz);

    }
    // (3) 自己加载失败，则请求parent代理加载

    if (!delegateLoad) {
        ClassLoader loader = parent;
        if (loader == null)
            loader = system;
        clazz = loader.loadClass(name);
        if (clazz != null) {
            return (clazz);

        }

    }
    throw new ClassNotFoundException(name);

}
```



**5个子问题**

**1、既然 Tomcat 不遵循双亲委派机制，那么如果我自己定义一个恶意的HashMap，会不会有风险呢？**
答： 显然不会有风险，如果有，Tomcat都运行这么多年了，那能不改进吗？ 
tomcat不遵循双亲委派机制，只是自定义的classLoader顺序不同，但顶层还是相同的，还是要去顶层请求classloader。

**2、我们思考一下：Tomcat是个web容器， 那么它要解决什么问题？**

1. 一个web容器可能需要部署两个应用程序，不同的应用程序可能会依赖同一个第三方类库的不同版本，不能要求同一个类库在同一个服务器只有一份，因此要保证每个应用程序的类库都是独立的，保证相互隔离。 
2. 部署在同一个web容器中相同的类库相同的版本可以共享。否则，如果服务器有10个应用程序，那么要有10份相同的类库加载进虚拟机，这是扯淡的。 
3. web容器也有自己依赖的类库，不能于应用程序的类库混淆。基于安全考虑，应该让容器的类库和程序的类库隔离开来。 
4. web容器要支持jsp的修改，我们知道，jsp 文件最终也是要编译成class文件才能在虚拟机中运行，但程序运行后修改jsp已经是司空见惯的事情，否则要你何用？ 所以，web容器需要支持 jsp 修改后不用重启。

**3、Tomcat 如果使用默认的类加载机制行不行？ **
答案是不行的。为什么？我们看:
第一个问题，如果使用默认的类加载器机制，那么是无法加载两个相同类库的不同版本的，默认的累加器是不管你是什么版本的，只在乎你的全限定类名，并且只有一份。
第二个问题，默认的类加载器是能够实现的，因为他的职责就是保证唯一性。
第三个问题和第一个问题一样。
我们再看第四个问题，我们想我们要怎么实现jsp文件的热替换，jsp 文件其实也就是class文件，那么如果修改了，但类名还是一样，类加载器会直接取方法区中已经存在的，修改后的jsp是不会重新加载的。那么怎么办呢？我们可以直接卸载掉这jsp文件的类加载器，所以你应该想到了，每个jsp文件对应一个唯一的类加载器，当一个jsp文件修改了，就直接卸载这个jsp类加载器。重新创建类加载器，重新加载jsp文件。

**4、如果tomcat 的 Common ClassLoader 想加载 WebApp ClassLoader 中的类，该怎么办？**
看了前面的关于破坏双亲委派模型的内容，我们心里有数了，我们可以使用线程上下文类加载器实现，使用线程上下文加载器，可以让父类加载器请求子类加载器去完成类加载的动作。

**5、为什么java文件放在Eclipse/IDEA中的src文件夹下会优先jar包中的class?**
tomcat类加载机制的理解，就不难明白。因为Eclipse/IDEA中的src文件夹中的文件java以及webContent中的JSP都会在tomcat启动时，被编译成class文件放在 WEB-INF/class 中。
而Eclipse/IDEA外部引用的jar包，则相当于放在 WEB-INF/lib 中。
因此肯定是 java文件或者JSP文件编译出的class优先加载。



## 6.2、沙箱安全机制

**Java安全模型的核心就是Java沙箱（sandbox）**。什么是沙箱？沙箱是一个限制程序运行的环境。

沙箱安全机制作用：

- 保证程序安全
- 保护Java原生的JDK代码

沙箱机制就是将Java代码限定在虚拟机（JVM）特定的运行范围中，并且严格限制代码对本地系统资源访问。通过这样的措施来保证对代码的有限隔离，防止对本地系统造成破坏。

沙箱主要限制系统资源访问，那系统资源包括什么？CPU、内存、文件系统、网络。不同级别的沙箱对这些资源访问的限制也可以不一样。

所有的Java程序运行都可以指定沙箱，可以定制安全策略。

### 6.2.1、JDK1.0时期

在Java中将执行程序分成本地代码和远程代码两种，本地代码默认视为可信任的，而远程代码则被看作是不受信的。对于授信的本地代码，可以访问一切本地资源。而对于非授信的远程代码在早期的Java实现中，安全依赖于沙箱（Sandbox）机制。如下图所示JDK1.0安全模型

![image-20241212205808056](【尚硅谷】JVM-类的加载篇/675adde05e887.webp)

### 6.2.2、JDK1.1时期

JDK1.0中如此严格的安全机制也给程序的功能扩展带来障碍，比如当用户希望远程代码访问本地系统的文件时候，就无法实现。
因此在后续的Java1.1版本中，针对安全机制做了改进，增加了安全策略。允许用户指定代码对本地资源的访问权限。如下图所示JDK1.1安全模型  

![image-20241212205914150](【尚硅谷】JVM-类的加载篇/675ade2272696.webp)

### 6.2.3、JDK1.2时期

在Java1.2版本中，再次改进了安全机制，增加了代码签名。不论本地代码或是远程代码，都会按照用户的安全策略设定，由类加载器加载到虚拟机中权限不同的运行空间，来实现差异化的代码执行权限控制。如下图所示JDK1.2安全模型：

![image-20241212210016383](【尚硅谷】JVM-类的加载篇/675ade60ac454.webp)

### 6.2.4、JDK1.6时期

当前最新的安全机制实现，则引入了域（Domain）的概念。
虚拟机会把所有代码加载到不同的系统域和应用域。系统域部分专门负责与关键资源进行交互，而各个应用域部分则通过系统域的部分代理来对各种需要的资源进行访问。虚拟机中不同的受保护域（Protected Domain），对应不一样的权限（Permission）。存在于不同域中的类文件就具有了当前域的全部权限，如下图所示，最新的安全模型（jdk1.6)

![image-20241212210148897](【尚硅谷】JVM-类的加载篇/675adebd36aba.webp)

## 7、JDK9中类加载结构的新变化

为了保证兼容性，JDK 9没有从根本上改变三层类加载器架构和双亲委派模型，但为了模块化系统的顺利运行，仍然发生了一些值得被注意的变动。

1、扩展机制被移除，扩展类加载器由于向后兼容性的原因被保留，不过被重命名为**平台类加载器（platform class loader）**。可以通过ClassLoader的新方法getPlatformClassLoader()来获取。

JDK 9 时基于模块化进行构建（原来的 rt.jar 和 tools.jar 被拆分成数十个 JMOD 文件），其中的 Java 类库就已天然地满足了可扩展的需求，那自然无须再保留 <JAVA_HOME>\lib\ext 目录，此前使用这个目录或者 java.ext.dirs 系统变量来扩展 JDK 功能的机制已经没有继续存在的价值了。

2、平台类加载器和应用程序类加载器都不再继承自 java.net.URLClassLoader。
现在启动类加载器、平台类加载器、应用程序类加载器全都继承于 jdk.internal.loader.BuiltinClassLoader。

![image-20241212210425530](【尚硅谷】JVM-类的加载篇/675adf59cd360.webp)

如果有程序直接依赖了这种继承关系，或者依赖了 URLClassLoader 类的特定方法，那代码很可能会在 JDK 9 及更高版本的 JDK 中崩溃。

3、在Java 9中，类加载器有了名称。该名称在构造方法中指定，可以通过getName()方法来获取。平台类加载器的名称是platform，应用类加载器的名称是app。类加载器的名称在调试与类加载器相关的问题时会非常有用。

4、启动类加载器现在是在jvm内部和java类库共同协作实现的类加载器（以前是 C++实现），但为了与之前代码兼容，在获取启动类加载器的场景中仍然会返回null，而不会得到BootClassLoader实例。

5、类加载的委派关系也发生了变动。
当平台及应用程序类加载器收到类加载请求，在委派给父加载器加载前，要先判断该类是否能够归属到某一个系统模块中，如果可以找到这样的归属关系，就要优先委派给负责那个模块的加载器完成加载。

双亲委派模式示意图

![image-20241212210607249](【尚硅谷】JVM-类的加载篇/675adfbfb209d.webp)

**附加：**
在 Java 模块化系统明确规定了三个类加载器负责各自加载的模块：
启动类加载器负责加载的模块

```bash
java.base                        java.security.sasl
java.datatransfer                java.xml
java.desktop                     jdk.httpserver
java.instrument                  jdk.internal.vm.ci
java.logging                     jdk.management
java.management                  jdk.management.agent
java.management.rmi              jdk.naming.rmi
java.naming                      jdk.net
java.prefs                       jdk.sctp
java.rmi                         jdk.unsupported
```

平台类加载器负责加载的模块

```
java.activation*                jdk.accessibility
java.compiler*                  jdk.charsets
java.corba*                     jdk.crypto.cryptoki
java.scripting                  jdk.crypto.ec
java.se                         jdk.dynalink
java.se.ee                      jdk.incubator.httpclient
java.security.jgss              jdk.internal.vm.compiler*
java.smartcardio                jdk.jsobject
java.sql                        jdk.localedata
java.sql.rowset                 jdk.naming.dns
java.transaction*               jdk.scripting.nashorn
java.xml.bind*                  jdk.security.auth
java.xml.crypto                 jdk.security.jgss
java.xml.ws*                    jdk.xml.dom
java.xml.ws.annotation*         jdk.zipfs
```

应用程序类加载器负责加载的模块

```
jdk.aot                         jdk.jdeps
jdk.attach                      jdk.jdi
jdk.compiler                    jdk.jdwp.agent
jdk.editpad                     jdk.jlink
jdk.hotspot.agent               jdk.jshell
jdk.internal.ed                 jdk.jstatd
jdk.internal.jvmstat            jdk.pack
jdk.internal.le                 jdk.policytool
jdk.internal.opt                jdk.rmic
jdk.jartool                     jdk.scripting.nashorn.shell
jdk.javadoc                     jdk.xml.bind*
jdk.jcmd                        jdk.xml.ws*
jdk.jconsole
```

