---
title: "Java基础"
date: 2023-06-18T14:05:25+08:00
tags: ["Java"]
categories: []
draft: false
---
## 数据类型

### INFINITY和NaN

```java
// INFINITY定义
public static final double POSITIVE_INFINITY = 1.0 / 0.0;
public static final double NEGATIVE_INFINITY = -1.0 / 0.0;

public static final float POSITIVE_INFINITY = 1.0f / 0.0f;
public static final float NEGATIVE_INFINITY = -1.0f / 0.0f;

// 无穷大*0=NAN
System.out.println(Float.POSITIVE_INFINITY * 0); // output: NAN

// 无穷大
System.out.println((Float.POSITIVE_INFINITY / 0) == Float.POSITIVE_INFINITY); // output: true
System.out.println(Float.POSITIVE_INFINITY == (Float.POSITIVE_INFINITY + 10000)); // output: true
System.out.println(Float.POSITIVE_INFINITY == (Float.POSITIVE_INFINITY / 10000)); // output: true

// 判断是否为INFINITY
System.out.println(Double.isInfinite(Float.POSITIVE_INFINITY)); // output: true
```

```java
// NAN定义
public static final double NaN = 0.0d / 0.0;

// NAN表示非数字，它与任何值都不相等，甚至不等于它自己，所以要判断一个数是否为NAN要用isNAN方法：
System.out.println(Float.NaN == Float.NaN); // output: false
System.out.println(Double.isNaN(Float.NaN)); // output: true
```

> 需要精确计算时，使用`BigDecimal`，不要使用float和double

### 判断相等

`float`不能使用`==`判断相等，`Float`不能使用`equals()`和`flt.compareTo(another) == 0`比较大小。可以用使用`Math.abs(f1 - f2) < 1e-6f`

`Float f1`和`float f2`使用`==`比较大小时，会先把f1拆箱，然后相当于float比较。

## IO操作

IO流是一种流式的数据输入/输出模型：

- 二进制数据以`byte`为最小单位在`InputStream`/`OutputStream`中单向流动；
- 字符数据以`char`为最小单位在`Reader`/`Writer`中单向流动。

Java标准库的`java.io`包提供了同步IO功能：

- 字节流接口：`InputStream`/`OutputStream`；
- 字符流接口：`Reader`/`Writer`。

### File类

Java标准库的`java.io.File`对象表示一个文件或者目录：

- 创建`File`对象本身不涉及IO操作；
- 可以获取路径／绝对路径／规范路径：`getPath()`/`getAbsolutePath()`/`getCanonicalPath()`；
- 可以获取目录的文件和子目录：`list()`/`listFiles()`；
- 可以创建或删除文件和目录。

### InputStream/OutputStream

Java标准库的`java.io.InputStream`定义了所有输入流的超类：

- `FileInputStream`实现了**文件流**输入，==**需要关闭流**==；
- `ByteArrayInputStream`在内存中模拟一个**字节流**输入，==**不需要关闭流**==。

> 在解压图片的时候发现`ByteArrayOutputStream`不需要关闭，为啥呢？
>
> `ByteArrayOutputStream`或`ByteArrayInputStream`是内存读写流，不同于指向硬盘的流，它内部是使用字节数组读内存的，这个字节数组是它的成员变量，当这个数组不再使用变成垃圾的时候，Java的垃圾回收机制会将它回收。所以不需要关流。

#### 文件流 FileInputStream

1. try...finally

> **切记释放资源**，即`input.close();`

```java
public void readFile() throws IOException {
    InputStream input = null;
    try {
        input = new FileInputStream("src/readme.txt");
        int n;
        while ((n = input.read()) != -1) { // 利用while同时读取并判断
            System.out.println(n);
        }
    } finally {
        if (input != null) { input.close(); } // 释放资源
    }
}


public void writeFile() throws IOException {
    OutputStream output = new FileOutputStream("out/readme.txt");
    output.write("Hello".getBytes("UTF-8")); // Hello
    output.close();
}
```

2. try(resource)  Java7引入

> 实际上，编译器并不会特别地为`InputStream`加上自动关闭。编译器只看`try(resource = ...)`中的对象是否实现了`java.lang.AutoCloseable`接口，如果实现了，就自动加上`finally`语句并调用`close()`方法。`InputStream`和`OutputStream`都实现了这个接口，因此，都可以用在`try(resource)`中。

```java
public void readFile() throws IOException {
    try (InputStream input = new FileInputStream("src/readme.txt")) {
        int n;
        while ((n = input.read()) != -1) {
            System.out.println(n);
        }
    } // 编译器在此自动为我们写入finally并调用close()
}
```

3. 阻塞

在调用`InputStream`的`read()`方法读取数据时，我们说`read()`方法是阻塞（Blocking）的。它的意思是，对于下面的代码：
和`InputStream`一样，`OutputStream`的`write()`方法也是阻塞的。

```java
int n;
n = input.read(); // 必须等待read()方法返回才能执行下一行代码
int m = n;
```

#### 字节流 ByteArrayInputStream

```java
public void readBuffer throws IOException {
    byte[] data = { 72, 101, 108, 108, 111, 33 };
    try (InputStream input = new ByteArrayInputStream(data)) {
        int n;
        while ((n = input.read()) != -1) {
            System.out.println((char)n);
        }
    }
}
```

```java
// 读取input.txt，写入output.txt:
try (InputStream input = new FileInputStream("input.txt");
     OutputStream output = new FileOutputStream("output.txt"))
{
    input.transferTo(output); // transferTo的作用是?
}
```

### 序列化 ObjectOuputStream

> 序列化是指把一个Java对象变成二进制内容，本质上就是一个`byte[]`数组。
>
> 一个类的对象要想序列化成功，必须满足两个条件：
>
> 1. 该类必须实现`java.io.Serializable`接口。`Serializable`接口没有定义任何方法，它是一个空接口。我们把这样的空接口称为“标记接口”（Marker Interface），实现了标记接口的类仅仅是给自身贴了个“标记”，并没有增加任何方法。
> 2. 该类的所有属性必须是可序列化的。如果有一个属性不是可序列化的，则该属性必须注明是**短暂**(transient)的。
> 3. static修饰的属性将不被序列化。

```java
public class Employee implements java.io.Serializable
{
    public String name;
    public String address;
    public transient int SSN;
    public int number;
    public void mailCheck() {
        System.out.println("Mailing a check to " + name + " " + address);
    }
}
```

把一个Java对象变为`byte[]`数组，需要使用`ObjectOutputStream`。它负责把一个Java对象写入一个字节流：

```java
public void test(String[] args) throws IOException {
    ByteArrayOutputStream buffer = new ByteArrayOutputStream();
    // FileOutputStream stream = new FileOutputStream(new File("person.out")); // 也可以用文件流，但是记得关闭流
    try (ObjectOutputStream output = new ObjectOutputStream(buffer)) {
        // 写入int:
        output.writeInt(12345);
        // 写入String:
        output.writeUTF("Hello");
        // 写入实现了Serializable接口的Object:
        output.writeObject(Double.valueOf(123.456));
    }

    System.out.println(Arrays.toString(buffer.toByteArray()));
}
```

### 反序列化 ObjectInputStream

>除了能读取基本类型和`String`类型外，调用`readObject()`可以直接返回一个`Object`对象。要把它变成一个特定类型，必须强制转型。
>
>`readObject()`可能抛出的异常有：
>
>- `ClassNotFoundException`：没有找到对应的Class；
>- `InvalidClassException`：Class不匹配。

```java
try (ObjectInputStream input = new ObjectInputStream(...)) {
    int n = input.readInt();
    String s = input.readUTF();
    Double d = (Double) input.readObject();
}
```

对于`ClassNotFoundException`，这种情况常见于一台电脑上的Java程序把一个Java对象，例如，`Person`对象序列化以后，通过网络传给另一台电脑上的另一个Java程序，但是这台电脑的Java程序并没有定义`Person`类，所以无法反序列化。

对于`InvalidClassException`，为了避免class定义变动导致的不兼容，Java的序列化允许class定义一个特殊的`serialVersionUID`静态变量，用于标识Java类的序列化“版本”，通常可以由IDE自动生成。如果增加或修改了字段，可以改变`serialVersionUID`的值，不兼容时就会抛出此异常。

```java
public class Person implements Serializable {
    private static final long serialVersionUID = 2709425275741743919L;
}
```

## 垃圾回收

### finalize用法

finalize()是Object类里的protected类型的方法，子类（所有类都是Object的子类）可以通过覆盖这个方法来实现回收前的资源清理工作。

由于重写finalize不当，会导致该对象无法回收，所以在项目里，我们一般不重写该方法，而会采用Object类自带的空的finalize方法。

> 1. Java虚拟机一旦通过刚才提到的“根搜索算法”判断出某对象处于可回收状态时，会判断该对象是否重写了Object类的finalize方法，如果没，则直接回收。
> 2. 如重写过finalize方法，而且未执行过该方法，则把该对象其放入F-Queue队列，另个线程会定时遍历F-Queue队列，并执行该队列中各对象的finalize方法。
> 3. finalize方法执行完毕后，GC会再次判断该对象是否可被回收，如果可以，则进行回收，如果此时该对象上有强引用，则该对象“复活”，即处于“不可回收状态”。

```java
public class FinalizeDemo {
    static FinalizeDemo obj = null;
    //重写Object里的finalize方法
    protected void finalize() throws Throwable {
       System.out.println("In finalize()");
       obj = this; //给obj加个强引用
    }
    public static void main(String[] args) throws InterruptedException {
        obj = new FinalizeDemo();
        obj = null; //去掉强引用
        System.gc(); //垃圾回收
        // sleep 1秒，以便垃圾回收线程清理obj对象
        Thread.sleep(1000);
        if (null != obj) { // 在finalize方法复活
            System.out.println("Still alive.");
        } else {
            System.out.println("Not alive.");
        } 
    }  
}
```



### 强引用、软引用、弱引用、虚引用

>[理解Java的强引用、软引用、弱引用和虚引用 - 掘金 (juejin.cn)](https://juejin.cn/post/6844903665241686029)

| 引用类型 | 被垃圾回收时间 | 用途               | 生存时间          |
| -------- | -------------- | ------------------ | ----------------- |
| 强引用   | 从来不会       | 对象的一般状态     | JVM停止运行时终止 |
| 软引用   | 当内存不足时   | 对象缓存           | 内存不足时终止    |
| 弱引用   | 正常垃圾回收时 | 对象缓存           | 垃圾回收后终止    |
| 虚引用   | 正常垃圾回收时 | 跟踪对象的垃圾回收 | 垃圾回收后终止    |

#### 强引用

**强引用**是使用最普遍的引用。如果一个对象具有强引用，那**垃圾回收器**绝不会回收它。当**内存空间不足**时，`Java`虚拟机宁愿抛出`OutOfMemoryError`错误，使程序**异常终止**。如下：

```java
	Object strongReference = new Object();

	// 如果强引用对象不使用时，需要弱化从而使GC能够回收
	strongReference = null;
```

> 显式地设置`strongReference`对象为`null`，或让其**超出**对象的**生命周期**范围，则`gc`认为该对象**不存在引用**，这时就可以回收这个对象。具体什么时候收集这要取决于`GC`算法。
>
> 在一个**方法的内部**有一个**强引用**，这个引用保存在`Java`**栈**中，而真正的引用内容(`Object`)保存在`Java`**堆**中。 当这个**方法运行完成**后，就会退出**方法栈**，则引用对象的**引用数**为`0`，这个对象会被回收。但是如果这个`strongReference`是**全局变量**时，就需要在不用这个对象时赋值为`null`，因为**强引用**不会被垃圾回收。

#### 软引用

如果一个对象只具有**软引用**，则**内存空间充足**时，**垃圾回收器**就**不会**回收它；如果**内存空间不足**了，就会**回收**这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。

```java
    // 强引用
    String strongReference = new String("abc");
    // 软引用
    String str = new String("abc");
    SoftReference<String> softReference = new SoftReference<String>(str);
```

> 软引用可用来实现内存敏感的高速缓存。
>
> 注意：软引用对象是在jvm内存不够的时候才会被回收，我们调用System.gc()方法只是起通知作用，JVM什么时候扫描回收对象是JVM自己的状态决定的。就算扫描到软引用对象也不一定会回收它，只有内存不够的时候才会回收。

**应用场景：**

浏览器的后退按钮。按后退时，这个后退时显示的网页内容是重新进行请求还是从缓存中取出呢？这就要看具体的实现策略了。

1. 如果一个网页在浏览结束时就进行内容的回收，则按后退查看前面浏览过的页面时，需要重新构建；
2. 如果将浏览过的网页存储到内存中会造成内存的大量浪费，甚至会造成内存溢出。

```java
    // 获取浏览器对象进行浏览
    Browser browser = new Browser();
    // 从后台程序加载浏览页面
    BrowserPage page = browser.getPage();
    // 将浏览完毕的页面置为软引用
    SoftReference softReference = new SoftReference(page);

    // 回退或者再次浏览此页面时
    if (softReference.get() != null) {
        // 内存充足，还没有被回收器回收，直接获取缓存
        page = softReference.get();
    } else {
        // 内存不足，软引用的对象已经回收
        page = browser.getPage();
        // 重新构建软引用
        softReference = new SoftReference(page);
    }
```

#### 弱引用

**弱引用**与**软引用**的区别在于：只具有**弱引用**的对象拥有**更短暂**的**生命周期**。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有**弱引用**的对象，不管当前**内存空间足够与否**，都会**回收**它的内存。不过，由于垃圾回收器是一个**优先级很低的线程**，因此**不一定**会**很快**发现那些只具有**弱引用**的对象。

```java
    String str = new String("abc");
    WeakReference<String> weakReference = new WeakReference<>(str);
    str = null;

	// JVM首先将软引用中的对象引用置为null，然后通知垃圾回收器进行回收：
    str = null;
    System.gc();
```

> 如果一个对象是偶尔(很少)的使用，并且希望在使用时随时就能获取到，但又不想影响此对象的垃圾收集，那么你应该用Weak Reference来记住此对象。
>
> **弱引用**可以和一个**引用队列**(`ReferenceQueue`)联合使用，如果**弱引用**所引用的**对象**被**垃圾回收**，`Java`虚拟机就会把这个**弱引用**加入到与之关联的**引用队列**中。

#### 虚引用

**虚引用**顾名思义，就是**形同虚设**。与其他几种引用都不同，**虚引用**并**不会**决定对象的**生命周期**。如果一个对象**仅持有虚引用**，那么它就和**没有任何引用**一样，在任何时候都可能被垃圾回收器回收。

> **虚引用**主要用来**跟踪对象**被垃圾回收器**回收**的活动。
>
> **虚引用**与**软引用**和**弱引用**的一个区别在于：虚引用必须和引用队列(ReferenceQueue)联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

```java
    String str = new String("abc");
    ReferenceQueue queue = new ReferenceQueue();
    // 创建虚引用，要求必须与一个引用队列关联
    PhantomReference pr = new PhantomReference(str, queue);
```
