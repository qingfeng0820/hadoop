# JIT Compiler
## C1 C2 Compiler
* hotspot虚拟机内嵌有两个JIT编译器，分别是Client Compiler 和Server Compiler，不过大多数情况下简称为C1和C2编译器
* client模式是一种轻量级编译器，也叫C1编译器，占用内存小，启动快，耗时短，它会进行简单并可靠的优化，更注重效率。
* server模式是一种重量级编译器，也叫C2编译器，启动慢，占用内存大，耗时长，但编译的代码执行效率更高，甚至会根据性能监控信息进行一些不可靠的激进优化，更注重质量。
## Mode
* 自动选择: 通常虚拟机会根据机器的硬件和操作系统自动选择运行C1还是C2
* 手动选择: 
  ```
  -Xint 采用解释器模式执行
  -Xcomp 采用即使编译器模式执行
  -Xmixed 采用解释器+即使编译器的混合模式共同执行
  ```
  * 解释模式（Interpreted Mode）： 只使用解释器，禁用JIT
  * 编译模式（Compiled Mode）： 只使用编译器(-client/-server, 不指定则自动选择)，优先使用编译模式将所有字节码编译成本地机器码，解释模式作为备用。
  * 混合模式（Mixed Mode）：这是默认模式，使用解析器 + 其中一个JIT编译器 (-client/-server, 不指定则自动选择)。
## Compiler number
* c1、c2编译器线程的默认number
  ```
  c1、c2编译器线程的默认数量是根据运行应用程序的容器/设备上可用的CPU数量决定的:
  CPUs    c1 threads    c2 threads
  1    1    1
  2    1    1
  4    1    2
  8    1    2
  16    2    6
  32    3    7
  64    4    8
  128    4    10
  ```
* -XX:CICompilerCount=N JVM参数来改变编译器线程数: N指定的数量的三分之一将被分配给c1编译器线程。剩余的线程数将被分配给c2编译器线程。
  ```
  c1, c2编译器线程高CPU消耗 - 潜在的解决方案

  有时你可能会看到c1、c2编译器线程消耗了大量的CPU。当这种类型的问题出现时，下面是解决它的潜在方案。
  1. 什么都不做（如果是间歇性的）
     在你的案例中，如果你的C2编译器线程的CPU消耗只是间歇性的高，而不是持续的高，并且它没有损害你的应用程序的性能，那么你可以考虑忽略这个问题。
  2. -XX:-TieredCompilation
     将这个'-XX:-TieredCompilation'JVM参数传递给你的应用程序。这个参数将禁用JIT热点编译。因此，CPU的消耗将下降。然而，作为副作用，你的应用程序的性能会下降。
  3. -XX:TieredStopAtLevel=N
     如果CPU峰值是由c2编译器线程单独造成的，你可以单独关闭c2编译。你可以通过'-XX:TieredStopAtLevel=3'。当你传递这个'-XX:TieredStopAtLevel'参数的值为3时，那么只有c1编译会被启用，c2编译会被禁用。
     有四个层次的编译:
       Compilation Level    Description
       0    Interpreted Code
       1    Simple c1 compiled code
       2    Limited c1 compiled code
       3    Full c1 compiled code
       4    C2 compiled code
     当'-XX:TieredStopAtLevel=3'时，代码将只编译到'Full c1 compiled code'级别。C2的编译将被停止。
  4. -XX:+PrintCompilation
     你可以向你的应用程序传递'-XX:+PrintCompilation'JVM参数。它将打印关于你的应用程序的编译过程的细节。这将有助于你进一步调整编译过程。
  5. -XX:ReservedCodeCacheSize=N
     Hotspot JIT编译器编译/优化的代码被存储在JVM内存的代码缓存区。该代码缓存区的默认大小为240MB。你可以通过向你的应用程序传递'-XX:ReservedCodeCacheSize=N'来增加它。例如，如果你想让它变成512MB，你可以这样指定。'-XX:ReservedCodeCacheSize=512m'。增加代码缓存大小有可能减少编译器线程的CPU消耗。
  6. -XX:CICompilerCount
     你可以考虑通过使用参数'-XX:CICompilerCount'增加C2编译器线程。你可以捕获线程转储并将其上传到fastThread等工具，在那里你可以看到C2编译器线程的数量。如果你看到较少的C2编译器线程数，而你有更多的CPU处理器/核，你可以通过指定'-XX:CICompilerCount=8'参数增加C2编译器线程数。
  ```