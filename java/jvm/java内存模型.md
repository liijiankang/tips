<h1 align=center> JVM 内存结构</h1>

<h2 align=left> 1. jvm内存空间 </h2>

* 程序计数器
* 堆
* 本地方法栈
* jvm栈
* 方法区

<div align=center>
    <img width="450" height="450" src="picture/1.png">
</div>

<font face="STCAIYUN" color=#5F9EA0>
&emsp;&emsp;JDK 1.8 同 JDK 1.7 比，元数据区取代了永久代。元空间的本质和永久代类似，都是对 JVM 规范中方法区的实现,但是元数据空间使用本地内存。</font>
<br></br>
<h3> &emsp;1.1.程序计数器</h3>

<h4> &emsp;&emsp;1.1.1.定义</h4>
<p>
    &emsp;&emsp;&emsp;程序计数器是一块较小的内存空间，是当前线程正在执行的那条字节码指令的地址。若当前线程正在执行的是一个本地方法，那么此时程序计数器为Undefined。
</p>

<h4> &emsp;&emsp;1.1.2.作用</h4>
    <p>
       &emsp;&emsp;&emsp;* 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制。
       </br>
       &emsp;&emsp;&emsp;* 在多线程情况下，程序计数器记录的是当前线程执行的位置，从而当线程切换回来时，就知道上次线程执行到哪了。
    </p>
<h4> &emsp;&emsp;1.1.3.特点</h4>
    <p>
    &emsp;&emsp;&emsp;* 线程私有，每条线程都有自己的程序计数器。
    </br>
    &emsp;&emsp;&emsp;* 随着线程的创建而创建，随着线程的结束而销毁。
    </br>
    &emsp;&emsp;&emsp;* 不会出现OutOfMemoryError.
    </p>
<h3> &emsp;1.2.堆</h3>
<h4> &emsp;&emsp;1.2.1.定义</h4>
<p>
&emsp;&emsp;&emsp;堆是用来存放对象的内存空间，几乎所有的对象都存储在堆中。
</p>
<h4> &emsp;&emsp;1.2.2.特点</h4>
<p>
&emsp;&emsp;&emsp;* 线程共享，整个 Java 虚拟机只有一个堆，所有的线程都访问同一个堆
</br>
&emsp;&emsp;&emsp;* 在虚拟机启动时创建
</br>
&emsp;&emsp;&emsp;* 是垃圾回收的主要场所
</br>
&emsp;&emsp;&emsp;* 新生代(Eden 区 From Survior To Survivor)、老年代
</p>
<h3> &emsp;1.3.本地方法栈</h3>
&emsp;&emsp; 本地方法栈是为 JVM 运行 Native 方法准备的空间，
<h3> &emsp;1.4.jvm栈</h3>
<h4> &emsp;&emsp;1.1.1.定义</h4>
    <p>
    &emsp;&emsp;&emsp;Java 虚拟机栈是描述 Java 方法运行过程的内存模型。为每一个即将运行的 Java 方法创建一块叫做“栈帧”的区域，用于存放该方法运行过程中的一些信息。如：
    &emsp;&emsp;&emsp;* 局部变量表
    </br>
    &emsp;&emsp;&emsp;* 操作数栈
    </br>
    &emsp;&emsp;&emsp;* 动态链接
    </br>
    &emsp;&emsp;&emsp;* 方法出口信息
    </br>
    &emsp;&emsp;&emsp;* ...
    </p>

<div align=center>
    <img width="250" height="350" src="picture/2.png">
</div>
    <p >
    <font face="STCAIYUN" color=#5F9EA0>
    &emsp;&emsp;&emsp;方法运行过程中需要创建局部变量时，就将局部变量的值存入栈帧中的局部变量表中。
    </font>
    </p>
<h4> &emsp;&emsp;1.1.2.特点</h4>
<p>
&emsp;&emsp;&emsp;* 局部变量表随着栈帧的创建而创建，它的大小在编译时确定，创建时只需分配事先规定的大小即可。在方法运行过程中，局部变量表的大小不会发生改变。
</br>
&emsp;&emsp;&emsp;* Java 虚拟机栈会出现两种异常：StackOverFlowError 和 OutOfMemoryError。
</br>
&emsp;&emsp;&emsp;&emsp;1.若 Java 虚拟机栈的大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度时，抛出 StackOverFlowError 异常
</br>
&emsp;&emsp;&emsp;&emsp;2.若允许动态扩展，那么当线程请求栈时内存用完了，无法再动态扩展时，抛出 OutOfMemoryError 异常。
</br>
&emsp;&emsp;&emsp;* Java 虚拟机栈也是线程私有，随着线程创建而创建，随着线程的结束而销毁。
</p>
<h3> &emsp;1.5.方法区</h3>
<h4> &emsp;&emsp;1.5.1.定义</h4>
<p>
&emsp;&emsp;&emsp;方法区是堆的一个逻辑部分,方法区存放以下信息：
</br>
&emsp;&emsp;&emsp;* 已经被虚拟机加载的类信息
</br>
&emsp;&emsp;&emsp;* 常量
</br>
&emsp;&emsp;&emsp;* 静态变量
</br>
&emsp;&emsp;&emsp;* 即时编译器编译后的代码
</p>
<h4> &emsp;&emsp;1.5.2.特点</h4>
<p>
&emsp;&emsp;&emsp;* 线程共享。 方法区是堆的一个逻辑部分，因此和堆一样，都是线程共享的。整个虚拟机中只有一个方法区。
</br>
&emsp;&emsp;&emsp;* 永久代。 方法区中的信息一般需要长期存在，而且它又是堆的逻辑分区，因此用堆的划分方法，把方法区称为“永久代”。
</br>
&emsp;&emsp;&emsp;* 内存回收效率低。 方法区中的信息一般需要长期存在，回收一遍之后可能只有少量信息无效。主要回收目标是：对常量池的回收；对类型的卸载。
</br>
&emsp;&emsp;&emsp;* 允许固定大小，也允许动态扩展，还允许不实现垃圾回收。
<h4> &emsp;&emsp;1.5.3.运行时常量池</h4>
<p>
&emsp;&emsp;&emsp; 1.方法区中存放：类信息、常量、静态变量、即时编译器编译后的代码。常量就存放在运行时常量池中。
</br>
&emsp;&emsp;&emsp; 2.当类被 Java 虚拟机加载后， .class 文件中的常量就存放在方法区的运行时常量池中。而且在运行期间，可以向常量池中添加新的常量。如 String 类的 intern() 方法就能在运行期间向常量池中添加字符串常量。
<h3> &emsp;1.6.直接内存</h3>
<p>
&emsp;&emsp;&emsp; 除 Java 虚拟机之外的内存，但也可能被 Java 使用。
</p>
<h4>&emsp;&emsp;1.6.1.操作直接内存</h4>
<p>
&emsp;&emsp;&emsp;在 NIO 中引入了一种基于通道和缓冲的 IO 方式。它可以通过调用本地方法直接分配 Java 虚拟机之外的内存，然后通过一个存储在堆中的DirectByteBuffer对象直接操作该内存，而无须先将外部内存中的数据复制到堆中再进行操作，从而提高了数据操作的效率。直接内存的大小不受 Java 虚拟机控制，但既然是内存，当内存不足时就会抛出 OutOfMemoryError 异常。
</p>
<h4>&emsp;&emsp;1.6.2.直接内存与堆内存比较</h4>
<p>
&emsp;&emsp;&emsp;* 直接内存申请空间耗费更高的性能
</br>
&emsp;&emsp;&emsp;* 直接内存读取 IO 的性能要优于普通的堆内存
</br>
&emsp;&emsp;&emsp;* 直接内存作用链： 本地 IO -> 直接内存 -> 本地 IO
</br>
&emsp;&emsp;&emsp;* 堆内存作用链：本地 IO -> 直接内存 -> 非直接内存 -> 直接内存 -> 本地 IO
</p>

