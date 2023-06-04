# scala无符号类型的支持
# scala无符号类型的支持

## 起源
最开始的设计中scala的类型与jvm支持的类型是一一对应的，而jvm中没有无符号类型的支持。但是，当scala需要配合其他类型的语言来配合使用时，无符号类型导致的互操作性渐渐暴露出来。原文举例，使用了`scala.js`的例子，为了表达UByte不得不使用Short类型，而为了使用Uint类型不得不使用Double类型 **TODO. why?** 。同时，从java8开始jdk中也集成了操作无符号类型的方法，当然这里不是原生类型。scala由于通过继承AnyVal (**TODO**) 的方式来实现基础类型，所以可以方便的扩展类型，因此在2.12开始加入了无符号类型的支持。提供了四种类型的支持：UByte UShort UInt ULong，使用方式与其他类型保持一致。

## 优点
这里举例看看使用无符号类型可以解决什么问题：
```scala
// 不使用无符号类型
val array = new Uint32Array(js.Array(5, 7, 6, 98))
val x = array(3) / array(0)
println(x) // 19.6 (!)

array(2) = 6.4    // compiles, silent drop of precision
println(array(2)) // 6

// 使用无符号类型
val array = new Uint32Array(js.Array(5, 7, 6, 98).map(_.toUInt))
val x = array(3) / array(0)
println(x) // 19, as expected

array(2) = 6.4 // does not compile, yeah!
```
本例用来说明使用无符号类型的遍历性，通过使用无符号类型表达更明确的语义，避免了奇怪的类型问题。这个例子里因为无符号类型增强了类型检测。

```scala
// 不使用无符号类型，隐藏bug
def decodeISO88591(buffer: Array[Byte]): String = {
  val result = new StringBuilder(buffer.length)
  for (i <- 0 until buffer.length)
    result.append(buffer(i).toChar)
  result.toString()
}

// 使用无符号类型
def decodeISO88591(buffer: Array[UByte]): String = {
  val result = new StringBuilder(buffer.length)
  for (i <- 0 until buffer.length)
    result.append(buffer(i).toChar) // correct: UByte.toChar does not sign-extend
  result.toString()
}
```
Byte类型的toChar的处理过程是：先将Byte类型通过扩展符号位成为Int类型，之后通过截取高16bit之后生成结果，这种方式在处理超过0x80的数据时因为提前作了符号扩展导致生成的结果高8bit都是1。不使用无符号类型的解决方案是使用`(buffer(i)&0xff).toChar`提前将符号扩展的bit置为0。比较起来使用无符号类型更加优雅。

### 对比方案
在 [Unsigned int considered harmful for Java](unsign for java) 这篇文章中提出了怎么使用signed 来表示unsigned类型，存在的缺点是使用不太方便。

### 反例
我们没有实现大整数和浮点数的的无符号类型。
同时无符号类型的文本表示应该与带符号类型一致，然后使用toUint来转换类型。  
**TODO.** 怎么表示无符号类型呢？默认的文本如果识别为带符号类型，作转换时不会发生截断吗？除非toUInt作了特殊处理。

## 缺陷
除非编译器对无符号类型作了特殊处理，无符号类型的数组会有额外的开销，因为每个元素都需要作装箱。因此性能比较重要的场景下建议还是使用带符号类型操作。

当然，也可以保持现有的有符号类型系统，在必要的时候作bit处理。

## 设计
如果仅仅按照最直观的解释进行实现，当无符号类型与有符号类型混合使用会产生问题，这里讨论可用的操作：

设计上基本与signed类型保持相同的语义，不同的是严禁signed与unsigned类型之间的转换，同时显示转换是允许的，但是从signed到unsigned的widening conversion是不允许的，因为在widening的过程中会进行bit扩展。  
另外 unary_-操作与>>操作因为在unsigned类型下没有意义并且容易争议，所以没有提供实现。  
性能上其实基本与signed类型差不多，因为jvm中已经提供了对应的unsigned的实现，在BoxesRunTime中虽然可能会减慢==操作，但是jvm会智能判断是否需要使用unsigned类型来作对应的优化。这部分没看太懂。

## 实现
这里记录下疑惑：
### 无符号除法比有符号除法慢
### BoxesRunTime对unsigned的==操作作了特殊处理，所以unsigned的==操作比较慢
### unsigned的问题是解决算术运算吗，无符号类型的bit与有符号类型的bit操作应该是一样的，为什么需要unsigned类型？
### int 表示 unsigned short

实现unsigned的过程就是重新解释bit的过程，同一个bit模式的不同解释引出了signed unsigned类型的划分

## 参考链接
[SIP-26](https://docs.scala-lang.org/sips/unsigned-integers.html#previous-discussions-and-implementations)  
[unsign for java](https://www.nayuki.io/page/unsigned-int-considered-harmful-for-java)  
https://www.zhihu.com/question/65775397  
[java 无符号类型方案](https://blog.csdn.net/LANGZI7758521/article/details/51853298)  
https://stackoverflow.com/questions/21212993/unsigned-variables-in-scala

## 时间线
> 2023.05.28 yeon.guo ShangHai MinHang 迁移
