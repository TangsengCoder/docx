![](images/20170610165140237.png)

- jvm被分为三个主要的子系统
  - 类加载器子系统
  - 运行时数据区
  - 执行引擎

# jvm体系结构概述

## 类的加载 

- 类的加载指的是将类的.class文件中的二进制字节码读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个**java.lang.Class**对象，用来封装类在方法区内的数据结构。类的加载的最终产品是位于堆区中的**Class** 对象，**Class** 对象封装了类在方法区内的数据结构，并且向java程序员提供了访问方法区内的数据结构的接口。

  - 将类的.class文件中的二进制字节码加载至jvm中，jvm通过类名，类所在包名，ClassLoader完成类的加载。因此，标识一个被加载了的类：类名+包名+ClassLoader实例ID
  - 加载过程：
    - 通过一个类的全限定名称来获取此类的二进制流
    - 将这个字节流所代表的静态存储结构转换为方法区的运行时数据结构
    - 在内存中生成一个代表这个类的Java.lang.Class对象，作为方法区这个类的各种数据访问的入口

- 加载**.class** 文件的方式：

  - 从本地系统中直接加载

  - 通过网络下载.class文件

  - 从zip、jar等归档文件中加载.class文件

  - 从专有数据库中提取.class文件

  - 将java源文件动态编译为.class文件

  - > 类加载有三种方式：
    >
    > - 命令行启动应用时候由jvm初始化加载
    > - 通过Class.foName()方法动态加载
    > - 通过ClassLoader.loadClass()方法动态加载

- 类加载器

  https://blog.csdn.net/csdnliuxin123524/article/details/81303711

  （jvm启动，程序开始执行时，负责将class字节码加载到jvm内存区域中（运行时数据区中））

  - 启动类加载器（Bootstrap ClassLoader）
    - jar/lib/rt.jar
  - 扩展类加载器（ExtClassLoader）
    - jar/lib/ext/
  - 应用类加载器（Application ClassLoader）
    - classpath
  - 自定义加载器（User ClassLoader）

- 类加载机制
  - 全盘负责
    - 当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其它Class也将由该类加载器负责载入，除非显示使用另一个类加载器来载入
  - 父类委托（双亲委派）
    - 先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类（啃老族）
  - 缓存机制
    - 将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启jvm，程序的修改才会生效。

- 双亲委派模型

  - 

## 类的生命周期 

- 加载阶段

- 验证阶段

  - 负责对二进制字节码的格式进行校验

- 准备阶段：

  - **为类的静态变量分配内存，并将其初始化为默认值**

  - 是正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：

    - 这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在java堆中。

    - 这里所设置的初始值通常情况下是数据类型默认的值（如0，0L，null、false等），而不是被在java代码中被显式地赋予的值。

    - > 假设一个类变量的定义为：**public static final int value =3；** ，那么变量value在准备阶段过后的初始值为0，而不是3，因为这时候尚未开始执行任何java方法，而把value赋值为3的**public static** 指令是在程序编译后，存放于类构造器<clinit()>方法之中的，所以把value赋值为3的动作将在初始化阶段才会执行（可以理解为static final常量在编译期就将其结果放入了调用它的类的常量池中）

- 解析阶段：
  - **把类中的符号引用转换为直接引用**
  - 

- 初始化
  - **为类的静态变量赋予正确的初始值，并且静态代码块将被执行**
  - jvm负责对类进行初始化，主要对类变量进行初始化。在java中对类变量进行初始值设定有两种方式：
    - 声明类变量是指定初始值
    - 使用静态代码块为类变量指定初始值
  - 类初始化时机：只有当对类的主动使用的时候才会导致类的初始化，类的主动使用包括以下6种：
    - 创建类的实例，也就是new的方式
    - 访问某个类或接口的静态变量，或者对该静态变量赋值
    - 调用类的静态方法
    - 反射（如Class.forName("com.test.Test")）
    - 初始化某个类的子类，则其父类也会被初始化
    - java虚拟机启动时被注明为启动类的类（java Test，有main方法的类），直接使用java.exe命令来运行某个主类

- 结束阶段
  - 以下情况将结束生命周期
    - 执行了System.exit()方法
    - 程序正常执行结束
    - 程序在执行过程中遇到了异常或错误而异常终止
    - 由于操作系统出现错误而导致java虚拟机进程终止

## 一个类何时被实例化 

> 当初始化完成后，一个类的**静态变量**被正确赋值。如果这个对象是被new出来的，那么在初始化完成之后会进入实例化阶段。
>
> **初始化完成以后，类被存放在方法区，此时并没有存放在堆内存中。**
>
> **只有当对象实例化进入堆内存中以后才会对非静态变量进行初始化赋值** 

- 实例化的具体步骤为：

  - 父类非静态成员初始化语句（包括代码块，按照在类定义中的顺序执行）-->父类构造函数-->子类非静态成员初始化语句（包括代码块，按照在类定义中的顺序执行）-->子类构造方法()

    - > 如果这个类有父类不光是先实例化父类，整体流程如下：
      >
      > - 加载父类
      >   - 为静态属性分配存储空间并赋初始值
      >   - 执行静态初始化块和静态初始化语句（从上至下）
      > - 加载子类
      >   - 为静态属性分配存储空间
      >   - 执行静态初始化块和静态初始化语句（从上至下）
      > - 加载父类构造器
      >   - 为实例属性分配存储空间并赋初始值
      >   - 执行实例初始化块和实例初始化语句
      >   - 执行构造器内容
      > - 加载子类构造器
      >   - 为实例属性分配存储空间并赋初始值
      >   - 执行实例初始化块和实例初始化语句
      >   - 执行构造器内容

## JVM内存结构 

![](images/331425-20160623115840235-1252768148.png)

- jvm内存结构主要有三大块：堆内存、方法区、栈。

  - 堆内存：是jvm中最大的一块由年轻代和老年代组成

    - 年轻代：就是创建和使用完之后立马就要被回收的对象放在里面
      - 对象在被实例化之后，都是属于新生代。（例如，在一个方法中创建的对象会随着方法执行完毕，栈空间的栈帧出栈后而失去引用）
    - 老年代：就是把一些会长期存活的对象放在里面。（比如被static引用的对象，这种对象不会轻易的被垃圾回收器）

    - 为什么区分新生代和老年代？
      - 因为这个垃圾回收有关，对于年轻代的对象，他们的特点是很快就会被回收，所以需要使用一种垃圾回收算法。
      - 而对于老年代而言，里面的大部分对象可能都会长期存活，那么使用新生代的回收算法放在这里就可能并不是那么的合适，需要有着自己的一套回收算法。
    - 新生代对象是如何变成老年代对象的？
      - 长期存活的对象会多次躲过垃圾回收
        - 默认，当一个对象躲过10次Minor GC而没有被回收掉，就需要被转移到老年代去。
      - 特别大的超大对象直接不经过新生代就进入老年代
      - 动态年龄判断机制
        - 年轻代内存又被分成三部分，Eden空间、From Survivor空间、To Survivor空间，默认情况下年轻代按照8:1:1的比例来分配。
        - 生成的对象默认在Eden区域。当发生一次Minor GC后，会将存活的对象复制到其中一个Survivor区域。当下一次GC后又会将存活的对象复制到另一块survivor。那么做的好处是减少内存碎片。当发现其中的1岁，2岁，3岁的对象年龄加起来内存超过survivor区域的一半，就会把4岁及4岁以上的对象转移到老年代。
      - GC后survivor区域放不下
        - ？
      - 空间担保机制
        - 在发生minor gc之前，虚拟机会检测：老年代最大可用的连续空间>新生代all对象总空间
          - 满足，minor gc是安全的，可以进行minor gc
          - 不满足，虚拟机查看HandlerPromotionFailure参数：
            - 为true，允许担保失败，会继续检测老年代最大可用的连续空间>历次晋升到老年代对象的平均大小。若大于，将尝试进行一次minor gc，若失败，则重新进行一次full gc。
            - 为false，则不允许冒险，要进行full gc（对老年代进行gc）

  - 方法区：存储类信息、常量、静态变量等数据，是线程共享的区域，为与java堆区分，方法区还有一个别名：Non-Heap（非堆），jdk1.8后，这块区域叫做“Matespace”（元数据空间）。

    - JVM里的永久区其实就是方法区
      - 所谓的永久代，可认为是存放一些类的信息，例如，生成的.class就是存放在这个区的。一般情况下，我们对jvm调优都是对新生代和老年代进行调优。一般而言永久代保持沉默配置就可以了。但因为要存储类的相关信息，所以对于动态生成类的情况比较容易出现永久代的内存溢出。最典型的场景就是，thymeleaf模板比较多的情况，容易出现永久代内存溢出。

  - 栈：分为java虚拟机栈和本地方法栈，主要用于方法的执行

    https://www.cnblogs.com/zhxiansheng/p/11185840.html

    - java虚拟机栈：（java代码在执行的时候，一定是某个线程来执行某个方法中的代码）当线程执行到某个方法的时候，如果这个方法有局部变量，那么就需要一块区域来存放局部变量的数据信息。这个区域就叫做**java虚拟机栈**。

      - 每个线程都有一个自己的java虚拟机栈，比如说当执行main方法的时候就会有一个main线程，用来存放main方法中定义的局部变量

      - ```java
        public static void main(String[] args){
            People people = new People();
            int i = 9;
        }
        /**
        比如上面的main()方法中，其实就有一个“people”的局部变量，他是引用一个People的实例对象的，然后还有一个“i”的局部变量
        */
        ```

      - ![](images/QQ截图20190823112836.png)

      - 栈的数据结构：后进先出

        - 为什么要用后进先出的数据结构？
          - 假设a方法当中同步调用b方法，此时a方法的栈帧先入栈，然后再是b方法的栈帧入栈。b方法执行完毕后，b方法的栈帧出栈，继续执行a方法，所以使用一个后进先出的栈结构是非常完美的。

    - 本地方法栈：保存本地方法信息。对每个线程，将创建一个单独的本地方法栈

![](images/331425-20160623115841781-223449019.png)

- 控制参数：

  - -Xms设置堆的最小空间大小

  - -Xmx设置堆的最大空间大小

  - -XX:NewSize设置新生代最小空间大小

  - -XX:MaxNewSize设置新生代最大空间大小

  - -Xss设置每个线程的堆栈大小

  - > 没有直接设置老年代的参数，但是可以设置堆空间大小和新生代空间大小两个参数来间接控制：
    >
    > **老年代空间大小 = 堆空间大小-年轻代空间大小** 

  - -Xms、-Xmx是对堆的性能调优参数，一般两个设置是一样的，如果不一样，当Heap不够用，会发生内存抖动。一般都调大这两个参数，并且两个大小一样。

  - -Xss是对每一个线程栈的性能调优参数，影响堆栈调用的深度

![](images/331425-20160623115846235-947282498.png)

![](images/20180617161343935.png)

```java
package com.spark.jvm;
/**
 * 从JVM调用的角度分析java程序堆内存空间的使用：
 * 当JVM进程启动的时候，会从类加载路径中找到包含main方法的入口类HelloJVM
 * 找到HelloJVM会直接读取该文件中的二进制数据，并且把该类的信息放到运行时的Method内存区域中。
 * 然后会定位到HelloJVM中的main方法的字节码中，并开始执行Main方法中的指令
 * 此时会创建Student实例对象，并且使用student来引用该对象（或者说给该对象命名），其内幕如下：
 * 第一步：JVM会直接到Method区域中去查找Student类的信息，此时发现没有Student类，就通过类加载器加载该Student类文件；
 * 第二步：在JVM的Method区域中加载并找到了Student类之后会在Heap区域中为Student实例对象分配内存，
 * 并且在Student的实例对象中持有指向方法区域中的Student类的引用（内存地址）；
 * 第三步：JVM实例化完成后会在当前线程中为Stack中的reference建立实际的应用关系，此时会赋值给student
 * 接下来就是调用方法
 * 在JVM中方法的调用一定是属于线程的行为，也就是说方法调用本身会发生在线程的方法调用栈：
 * 线程的方法调用栈（Method Stack Frames），每一个方法的调用就是方法调用栈中的一个Frame，
 * 该Frame包含了方法的参数，局部变量，临时数据等 student.sayHello();
 */
public class HelloJVM {
	//在JVM运行的时候会通过反射的方式到Method区域找到入口方法main
	public static void main(String[] args) {//main方法也是放在Method方法区域中的
		/**
		 * student(小写的)是放在主线程中的Stack区域中的
		 * Student对象实例是放在所有线程共享的Heap区域中的
		 */
		Student student = new Student("spark");
		/**
		 * 首先会通过student指针（或句柄）（指针就直接指向堆中的对象，句柄表明有一个中间的,student指向句柄，句柄指向对象）
		 * 找Student对象，当找到该对象后会通过对象内部指向方法区域中的指针来调用具体的方法去执行任务
		 */
		student.sayHello();
	}
}
 
class Student {
	// name本身作为成员是放在stack区域的但是name指向的String对象是放在Heap中
	private String name;
	public Student(String name) {
		this.name = name;
	}
	//sayHello这个方法是放在方法区中的
	public void sayHello() {
	System.out.println("Hello, this is " + this.name);
	}
}

```

## JVM执行引擎 

- 类加载器将字节码载入内存后，执行引擎以java字节码为单元，读取java字节码。java字节码机器读不懂，必须将字节码转化为平台相关的机器码。这个过程就是由执行引擎完成的。



# GC算法 

- 垃圾回收（Garbage Collection）：GC，垃圾回收算法主要采用的是分代收集算法。
- jvm中，程序计数器、虚拟机栈、本地方法栈都是随着线程而生随着线程而灭，栈帧随着方法的进入和退出做入栈和出栈的操作，实现了自动的内存清理，因此，内存垃圾回收主要集中于java堆（新生代和老年代）和方法区（永久代）中，在程序运行期间，这部分内存的分配和使用都是动态的。
  - 新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要复制少量存活对象即可完成收集。
  - 老年代中，因为对象存活率高、没有额外的空间对它进行分配担保，就必须使用“标记-清理”或者“标记-整理”算法来回收。

## 如何判断一个对象是否是垃圾 

- 引用计数：每个对象都有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可回收。（此方法无法解决对象相互循环引用的问题）

> 例如：当一个对象被创建出来的时候，比如说在一个方法中创建一个对象，当该方法执行完毕后，就没有引用指向这个对象了，这个对象就会变成垃圾对象。（这仅仅是一种情况）

- jvm使用了一种可达性分析算法来判断哪些对象是可以被回收的。这个算法的核心就是看这个对象有谁在引用它，然后一层一层的往上判断，看是否被GC roots所引用。当一个对象到GC roots没有任何引用链相连时，则证明此对象是不可用的，不可达对象。
  - 在java中，可作为GC roots的对象有：
    - 虚拟机栈（栈帧中的本地变量表）中引用的对象
    - 方法区中的类静态属性引用的对象
    - 方法区中常量引用的对象
    - 本地方法栈JNI（即一般说的Native方法）中引用的对象

## jvm内存泄漏和内存溢出 

https://blog.csdn.net/shengmingqijiquan/article/details/77508471

- 内存泄漏

  - 指程序中动态分配内存给一些临时对象，但是对象不会被GC所回收，它始终占用内存。即被分配的对象可达但已无用。

- 内存溢出

  - 指程序运行过程中无法申请到足够的内存而导致的一种错误。内存溢出通常发生于OLD段或Perm段垃圾回收后，仍然无内存空间容纳新的java对象的情况。

  （从定义上可看出内存泄漏是内存溢出的一种诱因）

- 常见内存泄漏的几种场景？

  - 机器的连接数和关闭时间设置 （例如，数据库连接、IO连接）
  - 长生命周期的对象持有短生命周期对象的引用
    - 例如：在全局静态map中缓存局部变量，且没有清空操作，随着时间的推移，这个map会越来越大，造成内存泄露

- 常见内存溢出的几种情况？

  - 堆内存溢出（outOfMemoryError：java heap space）
    - 在jvm规范中，堆中的内存是用来生成对象实例和数组的。 
      如果细分，堆内存还可以分为年轻代和年老代，年轻代包括一个eden区和两个survivor区。 
      当生成新对象时，内存的申请过程如下： 
      a、jvm先尝试在eden区分配新建对象所需的内存； 
      b、如果内存大小足够，申请结束，否则下一步； 
      c、jvm启动youngGC，试图将eden区中不活跃的对象释放掉，释放后若Eden空间仍然不足以放入新对象，则试图将部分Eden中活跃对象放入Survivor区； 
      d、Survivor区被用来作为Eden及old的中间交换区域，当OLD区空间足够时，Survivor区的对象会被移到Old区，否则会被保留在Survivor区； 
      e、 当OLD区空间不够时，JVM会在OLD区进行full GC； 
      f、full GC后，若Survivor及OLD区仍然无法存放从Eden复制过来的部分对象，导致JVM无法在Eden区为新对象创建内存区域，则出现”out of memory错误”： outOfMemoryError：java heap space
  - 方法区内存溢出（outOfMemoryError：permgem space）
    - 方法区主要存放的是类信息、常量、静态变量等
    - 所以如果程序加载的类过多，或者使用反射、gclib等这种动态代理生成类的技术，就可能导致该区发生内存溢出，一般该区发生内存溢出的错误信息为：outOfMemoryError：permgem space
  - 线程栈溢出（java.lang.StackOverflowError）
    - 线程栈时线程独有的一块内存结构，所以线程栈发生问题必定是某个线程运行时产生的错误。一般线程溢出是由于递归太深或方法调用层级过多导致的。错误信息为：java.lang.StackOverflowError
  - 发生了内存泄漏或溢出怎么办？
    - 要解决这个区域的异常，一般的手段首先是通过“内存映像分析工具”对dump出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是先分清楚到底是出现了内存泄漏or内存溢出
    - 内存泄漏：可通过工具查看泄漏对象的GC roots的引用链。然后找出导致垃圾收集器无法自动回收的原因，分析泄漏对象的类型信息，GC roots引用链的信息，准确定位泄漏代码的位置
    - 如果不是内存泄漏，那说明内存中的对象确实都是必须存活着，那就应当检查虚拟机的堆参数（-Xmx与-Xms），与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。



https://blog.csdn.net/ityouknow/article/details/78037470

> - 编码细节：
>   - 使用StringBuilder或StringBuffer来代替String
>   - 尽量少输出日志
> - GC优化的两个目的：
>   - 将进入老年代的对象数量降到最低
>   - 减少Full GC的执行时间

> GC 过程：
>
> - 监控GC状态
> - 分析监控结果后决定是否需要优化GC
>   - 如果分析结果显示运行GC的时间只有0.1-0.3秒，那么就不需要把时间浪费在GC优化上，但如果运行GC的时间达到1-3秒，甚至大于10秒，那么GC优化就很有必要的。
>   - 此外，如果GC执行时间满足下列所有条件，就没有必要进行GC优化了：
>     - Minor GC执行非常迅速（50ms以内）
>     - Minor GC没有频繁执行（大约10s执行一次）
>     - Full GC执行非常迅速（1s以内）
>     - Full GC没有频繁执行（大约10min执行一次）
> - 设置GC类型/内存大小
> - 分析结果
>   - 应考虑的最重要的因素：
>     - 单次Full GC运行时间
>     - 单次Minor GC运行时间
>     - Full GC运行间隔
>     - Minor GC运行间隔
>     - 整个Full GC的时间
>     - 整个Minor GC的运行时间
>     - 整个GC的运行时间
>     - Full GC的执行次数
>     - Minor GC的执行次数
> - 如果结果满意，将参数应用到所有服务器上并结束GC优化

- - -

# 垃圾收集器 

> 如果说收集算法是内存回收的方法论，垃圾收集器就是内存回收的具体实现

- Serial收集器（串行收集器）
- ParNew收集器
- Parallel收集器
- Parallel Old收集器
- CMS收集器
- G1收集器



# Hotspot内存管理 

# Hotspot垃圾回收器

# 调优

# 监控工具

# 实战

```shell
//https://blog.csdn.net/fuyuwei2015/article/details/73256425
top
printf "%x \n" 线程ID  //把线程十进制的线程ID转换为十六进制
jstack -F 线程ID > dump.txt
more dump.txt    //查看线程ID对应的位置，或nid=0xtid的位置
//或者：
jstack pid |grep '0xtid' -C5 --color


```

```shell
 jmap -dump:live,format=b,file=/dump201612271310.dat 384
```

## CPU飙升、内存不足解决办法

- 限制每个进程的内存大小、CPU使用率（cpulimit）(docker 可在启动时-m 带参数)

### 排除步骤

- 先用top命令找出CPU占比最高的
- ps -ef或者jps进一步定位，得知是一个怎么样的一个后台程序
- 定位到具体线程或者代码
  - ps -mp 进程 -o THREAD,tid,time
  - -m：显示所有的线程
  - -p pid 进程使用CPU的时间
  - -o 该参数后是用户自定义格式
- 将需要的线程ID转换为16进制格式
  - printf "%x\n" 有问题的线程ID
- jstack 进程ID |grep tid（16进制线程ID小写英文）-A60