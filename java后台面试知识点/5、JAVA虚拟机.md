### Oom的问题原因

1）分配的少了：比如虚拟机本身可使用的内存（一般通过启动时的VM参数指定）太少。
 
2）应用用的太多，并且用完没释放，浪费了。此时就会造成内存泄露或者内存溢出。



### 虚拟机参数
- xms 初始占用内存大小
- xmx 最大占用内存大小
- xmn 年轻代大小，增加则老年代减少，一般为八分之三
- xss 每个线程的堆栈大小

- XX:NewRatio	年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代)

### 虚拟机工具

- jps
- jinfo：实时查看和调整虚拟机的配置参数
- jmap：查看当前堆中各个对象的数量，大小，生成堆dump
- -XX:+HeapDumpBeforeFullGC。
- jhat
- jconsole
- jvisualVM

### Gc

年轻代：新生代、2*复活代、
年老代：
永久代：方法区、储存类信息但是类的class对象在堆中
大的对象直接进入老年代

### full gc

### 什么时候OutOfMemoryException
- M98%的时间都花费在内存回收
- 每次回收的内存小于2%
  满足这两个条件将触发OutOfMemoryException，这将会留给系统一个微小的间隙以做一些Down之前的操作，比如手动打印Heap Dump。



### 内存泄漏及解决方法

 1.系统崩溃前的一些现象：

每次垃圾回收的时间越来越长，由之前的10ms延长到50ms左右，FullGC的时间也有之前的0.5s延长到4、5s
FullGC的次数越来越多，最频繁时隔不到1分钟就进行一次FullGC
年老代的内存越来越大并且每次FullGC后年老代没有内存被释放
 之后系统会无法响应新的请求，逐渐到达OutOfMemoryError的临界值。

 

 2.生成堆的dump文件

 通过JMX的MBean生成当前的Heap信息，大小为一个3G（整个堆的大小）的hprof文件，如果没有启动JMX可以通过Java的jmap命令来生成该文件。

 

 3.分析dump文件

Visual VM
IBM HeapAnalyzer
JDK 自带的Hprof工具
Mat。

 

 4.分析内存泄漏

 通过Mat我们能清楚地看到，哪些对象被怀疑为内存泄漏，哪些对象占的空间最大及对象的调用关系。针对本案，在ThreadLocal中有很多的JbpmContext实例，经过调查是JBPM的Context没有关闭所致。

 另，通过Mat或JMX我们还可以分析线程状态，可以观察到线程被阻塞在哪个对象上，从而判断系统的瓶颈。