# 辨析alignof与alignas
最初对于alignof自认为是清楚的，`复合类型结构中内存对齐方式，用来友好的进行内存读取，根据对齐方式会选择性的加入padding`。  
直到后来遇到了alignas，无论是用法还是含义都陷入了模糊的状态。  

*BTW. 写的有点啰嗦*
- [ ] [stackoverflow对alignof/alignas讨论](https://stackoverflow.com/questions/17091382/memory-alignment-how-to-use-alignof-alignas)

## alignof
对于编程语言来说，数据类型分为两种：原生数据类型和自定义数据类型。  
- 原生数据类型指语言内置的数据类型，比如bool, int, long, double
- 自定义数据类型指根据需求搭建的数据类型，由多种原生数据类型和自定义数据类型组合而成
PS. 而class类型又有着独立的继承体系构建自定义类型  
---
当开始执行程序，各种各样的数据类型都需要加载到内存中，而alignof则是用来询问操作系统各个类型在内存中排列的方式。  
对于特定实现的操作系统来说，每种数据类型都有唯一的大小，比如char为1、long为64，数据类型的不同决定了内存读取大小的不同。  

既然每种数据类型都有各自的大小，而sizeof可以计算内存的大小，那alignof又是什么呢？  
这里与操作系统的内存读取有关，当我们谈论操作系统 32bit/64bit的时候，其实是在说：指令集的位数/数据总线的宽度 **TODO**  
也决定了内存读取过程中一次读取的大小。  

对于64bit的操作系统来说一次内存读取可以读取64bit（8byte）的数据，当遇到了自定义数据类型往往会超过这个大小，  
不可避免的读取过程需要分批进行，这里引入的问题就是如何高效的进行这种数据的读取。答案是尽可能减少内存读取的次数。  
单次读取64bit已经是固定的，那么读取次数越少意味着读取的效率越高。  

这就是为什么需要引入padding到内存中的原因。如果没有padding不可避免的单块内存会出现跨64bit的情况，  
当需要读取该内存数据时，就需要进行多次内存读取来拼接处完整的数据，比如：
```bash
------------------------
| I1 |    L1    | I2 |
------------------------
I1/I2 为两个int类型数据，各占4字节
L1 为long类型数据，占8字节
假设I1、L1、I2的内存地址分别为0、4、12
此时需要读取L1的数据，则需要先读取0-8，再读取9-16，之后拼接4-8,9-12来读取完整的L1的数据

如果能够在I1、L1直接插入4byte的padding，那么I1、L1、I2的内存地址分别为0、8、16
这样就可以直接读取8-16来获取L1的数据了，效率更高
```

现实情况中不会出现这种情况，因为原生数据类型的对齐方式已经固定好了，比如alignof(int) == 4, alignof(long) == 8，当操作系统尝试为long数据分配内存时，  
会选择N*alignof(long)的内存地址。这样每次需要读取long数据时仅通过单独读取操作就能完整获取到数据。  

而对于自定义数据类型来说，要求是每个属性都满足对应的对齐方式，最初出现的效果往往是自定义数据类型的对齐方式与属性中对齐方式最大值保持一致。

## alignas
与获取对齐方式的alignof相对应的，alignas用来设置对齐方式。  
alignas的用法有两种场景：
- 希望某块数据完整的占用cache line **TODO**，不会因为和其他数据共用cache line，当其他数据发生更新后导致cache line失效
- 某些align比较小的数据容易把单块数据分割到不同的64bit 例如前一节的示例（比如说`char[64]`由于`alignof(char) = 1`有可能出现被分割到不同的64bit情况）

alignas标识可以为分配的内存提前管理好对齐方式以获取更好的性能表现

## 参考链接
[简洁清晰介绍了alignof/alignas](http://georgeflanagin.com/alignas.php)  
[C++标准alignof/alignas/align_storage/align的分析](https://szza.github.io/2022/01/01/C++/0_align/)  
[告诉我alignof/alignas其实是和内存的读取有关的](https://yangwang.hk/?p=773)  
[alignas的应用场景](https://stackoverflow.com/questions/45478824/c-alignment-when-to-use-alignas)  

## 时间线
> 2023.06.23 yeon.guo ShangHai XuHui 
