### JIT编译器

#### 定义

JIT(just-in-time)，即时编译器，是一种执行计算机代码的方法，这种方法涉及在程序执行过程中而不是执行前进行编译。通常程序的执行分为两种：静态编译和动态解释。静态编译的代码在之前前全部被编译为机器码，而动态解释执行则是边运行编译。

#### JIT在java上的应用

为了优化java的性能，jvm在解释器之外引入了即时编译器：当程序运行时，解释器首先发挥作用，代码可以直接执行。随着时间的推移，即时编译器发挥作用，把越来越多的代码编译优化为本地代码，来获取更高的执行效率。在这时解释器作为编译运行的降级手段，在一些不可靠编译优化出现问题时，再切换回解释执行保证程序正常运行。

即使编译器极大地提高了java程序运行效率，跟静态编译相比，即时编译器可以选择行的编译热点代码，节省了编译时间和空间。

#### 编译模式

JIT编译器在运行程序中可以选择两种编译模式：client模式和server模式，根据运行环境选择两者中的一个达到效果最优。client模式通过-client参数指定，server模式通过-server参数指定。两种模式的主要区别是：

- client模式：程序启动快，编译效率快，但生成的机器代码执行效率比server模式低。
- server模式：程序启动慢，编译时间长，但是生成的机器代码质量高适用于执行时间长，对峰值有性能要求的系统

##### 分层编译

java7将client的启动性能优势和server的峰值性能优势结合起来。

分层编译将jvm执行状态分为5个层次，分别是：

1. 解释执行
2. 执行不带profiling的C1代码
3. 执行仅带调用次数以及循环回边执行次数profiling的C1代码
4. 执行所有profiling的C1代码
5. 执行C2代码

说明：C1代码表示client编译的代码，C2表示server编译的代码，profiling会在程序执行过程中收集能够反映程序状态的数据，会损失一些性能。

分层编译可以在短时间内满足部分不那么热的代码编译完成，在早期就可以提升部分系统性能。等到长时间的运行收集到足够多的数据后，能让热带码得到最好的优化。JDK8中分层编译默认开启。

#### 代码缓存（Codecache ）

```
The Java Virtual Machine (JVM) generates native code and stores it in a memory area called the codecache. The JVM generates native code for a variety of reasons, including for the dynamically generated interpreter loop, Java Native Interface (JNI) stubs, and for Java methods that are compiled into native code by the just-in-time (JIT) compiler. The JIT is by far the biggest user of the codecache. 
```

jvm产生本地代码将其存放在codecache中。jvm产生本地代码的方式有：动态产生解释器循环，JNI方法调用，由JIT编译器编译过的java方法。

##### codecache调优

如果codecache用满了，就会停止JIT编译。并且会产生 “CodeCache is full… The compiler has been disabled”  之类的警告信息。而JIT的停止工作会导致系统性能急剧下降。为了避免这种情况，我们可以使用如下参数对CodeCache进行调优：

- InitialCodeCacheSize ：codecache初始空间，jvm8默认为160K 
- ReservedCodeCacheSize ：codecache最大内存空间，jvm8默认32M（client模式）/48M （server模式）
- CodeCacheExpansionSize ：codecache增长速率，jvm8默认为32K（client模式）/64K （server模式）

##### 查看codecache的使用情况

在jvm启动时添加参数：-XX:+PrintCodeCache ，可以打印CodeCache的使用情况。

```
CodeCache: size=245760Kb used=1331Kb max_used=1343Kb free=244428Kb
 bounds [0x00000000030d0000, 0x0000000003340000, 0x00000000120d0000]
 total_blobs=370 nmethods=100 adapters=183
 compilation: enabled
```

#### 参考文献

- [jit编译器](https://zh.wikipedia.org/wiki/%E5%8D%B3%E6%99%82%E7%B7%A8%E8%AD%AF)
- [jit编译原理-美团技术团队](https://tech.meituan.com/2020/10/22/java-jit-practice-in-meituan.html)
- [codecache](https://docs.oracle.com/javase/8/embedded/develop-apps-platforms/codecache.htm)



