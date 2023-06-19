# ligra中压缩编码
# ligra中压缩编码

## 一、简介
ligra为输入的点边集合划分为两种表示方式分别是：sparse representation and dense representation. **sparse表示**是指输入的点集通过数组进行表示，数组中每个元素存储点的索引，元素无序且不保证唯一。与之相对，**dense表示**使用bool数组表示，进一步也可以使用bitmap，好处是节省空间且保证元素的唯一性。  
在ligra中使用数字对点进行索引，但由于图计算的数据量一般比较大，当点规模变大时数字表示占用空间也会飙升。所以，ligra+ 论文提出了对点存储使用压缩编码来节省存储空间。  
但是，在对点的表示中，ligra+并没有使用优化方式，在eaglegraph实现的BC算法中，正向迭代完成后内存中其实保存了**所有**点的数据，同时算法使用了sparse的表示方式所以会对内存造成比较大的压力。本文便是来探讨在点的表示中使用ligra+这套编码压缩方案，并给出实现逻辑。  

## 二、编码压缩原理
ligra+使用的编码方式是基于k-bit变长编码的优化版本，称为`run-length encoded byte codes`，下面分别对这两种编码作介绍：
### k-bit 变长编码
这种编码方式是protobuf中采用的编码方式：
- 除了最后一个字节，每个字节的第一个bit标识后续字节仍是该数据的编码
- 每个字节剩余7个bit表示原来的数据

ligra+使用的编码在对负值的处理与protobuf不一致，ligra将符号位放在第一个字节第二个bit，即第一个字节的数据为长度只有6位。protobuf会将负值转换为正值进行编码，此处不论。  
目前看到eaglegraph只实现了该编码格式。

### RLE 游程编码
ligra+ 基于变长编码应用了针对额外bit的优化，参考[`spMV`压缩优化][spMV]，称为`run-length encoded byte codes`，注意与`run-length encoded`是不同的。  

考虑到每次都要进行bit操作影响decode的效率，提出的优化方案将额外的bit保存到单独的字节中。这里使用了group的概念，每个group内存储的数据编码使用的字节数必须相同，此时单独使用一个字节来表示group的元数据。  
group信息分为两部分：编码使用的字节数，group中数据的个数。ligra中使用2bit保存使用的字节数，剩余6bit保存数据的个数，2bit可以表示的范围是1-4，至于为什么使用2bit暂未知道原因，毕竟ligra中是可以使用Long类型的。 **TODO.**  
此方案的缺点在于可能会造成空间的浪费。

### 压缩存储
ligra+使用了称为`difference encoding`的压缩方案，该方案对每个顶点的边作升序排列，然后仅存储当前顶点索引相对于前一个顶点的差值。这样除了第一条边的数据（存储与源端顶点的差值，可能为负值）外，其余边的数据都会比原先的存储数据要小，再通过变长编码格式达到压缩存储的目的，图规模越大边数越多效果越明显。  

## 三、Ligra与Ligra+中图表示
ligra与ligra+都是使用邻接表来表示图，图中的出边和入边分开存储。出边按照源端聚集存储，也就是每个顶点的出边放在一起，入边同理。因为所有顶点的边数据是存放在一起的，所以每个顶点为自己的边数据保留了offset信息。  
<br/>
PS：  
(ligra)非压缩的图结构 只存储了单边的数据，比如只存储出边的数据，解析时首先构建出边数据，构建完成后再依赖出边构建入边数据。参考ligra代码实现：ligra/IO.h Row.319 Func.readGraphFromBinary。  
![ligra storage](/home/transwarp/space/images/ligra存储格式.png)  

(ligra+)压缩后的图结构，在出边数据之后存储了入边的数据，包括offsets, degrees, edges。参考ligra代码实现：ligra/IO.h Row.476 Func.readCompressedGraph。   
![ligra storage](/home/transwarp/space/images/ligra+存储格式.png)   

## 四、代码实现
本节参考下ligra中的代码实现，这里参考串行化的RLE编码格式，主要关注RLE编码和差值编码两部分：

### 编码
<details>
<summary>compressFirstEdge</summary>

```c++
// 编码第一条边，额外注意
// 1. 这里对符号位作了特殊处理
long compressFirstEdge(uchar *start, long offset, uintE source, uintE target)
{
  ...
  intE preCompress = (intE)target - source;
  int bytesUsed = 0;
  uchar firstByte = 0;
  intE toCompress = abs(preCompress);
  firstByte = toCompress & 0x3f; // 0011|1111
  if (preCompress < 0)
  {
    // 符号位处理
    firstByte |= 0x40;
  }
  toCompress = toCompress >> 6;
  if (toCompress > 0)
  {
    // 变长编码标记位，表示编码未结束
    firstByte |= 0x80;
  }
  start[offset] = firstByte;
  offset++;

  // 下列循环重复上面的过程
  uchar curByte = toCompress & 0x7f;
  while ((curByte > 0) || (toCompress > 0))
  {
    bytesUsed++;
    uchar toWrite = curByte;
    toCompress = toCompress >> 7;
    // Check to see if there's any bits left to represent
    curByte = toCompress & 0x7f;
    if (toCompress > 0)
    {
      toWrite |= 0x80;
    }
    start[offset] = toWrite;
    offset++;
  }
  return offset;
}
```
</details>

<details>
<summary>compressEdges</summary>

```c++
// 编码一组边集，runlength个长度为numBytes的边
long compressEdges(uchar *start, long curOffset, uintE *savedEdges, uintE edgeI, int numBytes, uintT runlength)
{
  //header
  uchar header = numBytes - 1;
  header |= ((runlength - 1) << 2);
  // 将header存储到数据中，解码时使用
  start[curOffset++] = header;

  for (int i = 0; i < runlength; i++)
  {
    uintE e = savedEdges[edgeI + i] - savedEdges[edgeI + i - 1];
    int bytesUsed = 0;
    for (int j = 0; j < numBytes; j++)
    {
      // 数据位全8位都可以使用
      uchar curByte = e & 0xff;
      e = e >> 8;
      start[curOffset++] = curByte;
      bytesUsed++;
    }
  }
  return curOffset;
}
```
</details>

<details>
<summary>sequentialCompressEdgeSet</summary>

```c++
long sequentialCompressEdgeSet(uchar *edgeArray, long currentOffset, uintT degree, uintE vertexNum, uintE *savedEdges)
{
  if (degree > 0)
  {
    // Compress the first edge whole, which is signed difference coded
    currentOffset = compressFirstEdge(edgeArray, currentOffset,
                                      vertexNum, savedEdges[0]);
    if (degree == 1)
      return currentOffset;
    uintE edgeI = 1;
    uintT runlength = 0;
    int numBytes = 0;
    // 下面的循环每次写入一批编码长度相同的数据
    while (1)
    {
      uintE difference = savedEdges[edgeI + runlength] -
                         savedEdges[edgeI + runlength - 1];
      // 编码长度为1
      if (difference < ONE_BYTE)
      {
        if (!numBytes)
        {
          numBytes = 1;
          runlength++;
        }
        else if (numBytes == 1)
          runlength++;
        else
        {
          //发现与之前的编码长度不同，将之前的数据编码
          // 注意这里runlength没有加1，所以当前边不会被编码
          currentOffset = compressEdges(edgeArray, currentOffset, savedEdges, edgeI, numBytes, runlength);
          edgeI += runlength;
          runlength = numBytes = 0;
        }
      }
      else if (difference < TWO_BYTES)
      {
        ...
      }
      else if (difference < THREE_BYTES)
      {
        ...
      }
      else
      {
        // 剩余都是编码长度为4，TODO why？
        ...
      }

      // 如果本批数据长度达到64，也进行编码，应为header中6bit代表数据长度，最大为2^6
      if (runlength == 64)
      {
        currentOffset = compressEdges(edgeArray, currentOffset, savedEdges, edgeI, numBytes, runlength);
        edgeI += runlength;
        runlength = numBytes = 0;
      }
      // 最后一批数据编码后退出
      if (runlength + edgeI == degree)
      {
        currentOffset = compressEdges(edgeArray, currentOffset, savedEdges, edgeI, numBytes, runlength);
        break;
      }
    }
  }
  return currentOffset;
}
```
</details>

### 解码
<details>
<summary>eatFirstEdge</summary>

```c++
inline intE eatFirstEdge(uchar *&start, uintE source)
{
  uchar fb = *start++;
  //int sign = (fb & 0x40) ? -1 : 1;
  intE edgeRead = (fb & 0x3f);
  if (LAST_BIT_SET(fb))
  {
    // 变长编码标志位，代表还有后续编码
    int shiftAmount = 6;
    //shiftAmount += 6;
    while (1)
    {
      // 后续编码数据都是7bit
      uchar b = *start;
      edgeRead |= ((b & 0x7f) << shiftAmount);
      start++;
      if (LAST_BIT_SET(b))
        shiftAmount += EDGE_SIZE_PER_BYTE;
      else
        break;
    }
  }
  //edgeRead *= sign;
  return (fb & 0x40) ? source - edgeRead : source + edgeRead;
}
```
</details>

<details>
<summary>decode</summary>

```c++
inline void decode(T t, uchar *edgeStart, const uintE &source, const uintT &degree, const bool par = true)
{
  uintE edgesRead = 0;
  if (degree > 0)
  {
    // Eat first edge, which is compressed specially
    uintE startEdge = eatFirstEdge(edgeStart, source);
    if (!t.srcTarg(source, startEdge, edgesRead))
    {
      return;
    }
    uintT i = 0;
    edgesRead = 1;
    while (1)
    {
      if (edgesRead == degree)
        return;
      // 先取header
      uchar header = edgeStart[i++];
      uint numbytes = 1 + (header & 0x3);
      uint runlength = 1 + (header >> 2);
      // 每次解码一批数据
      switch (numbytes)
      {
      case 1:
        for (uint j = 0; j < runlength; j++)
        {
          uintE edge = (uintE)edgeStart[i++] + startEdge;
          startEdge = edge;
          // srcTarg用来回调update操作，也就是API中的F
          if (!t.srcTarg(source, edge, edgesRead++))
          {
            return;
          }
        }
        break;
      case 2:
        ...
      case 3:
        ...
      default:
        for (uint j = 0; j < runlength; j++)
        {
          uintE edge = (uintE)edgeStart[i] + (((uintE)edgeStart[i + 1]) << 8) + (((uintE)edgeStart[i + 2]) << 16) + (((uintE)edgeStart[i + 3]) << 24) + startEdge;
          i += 4;
          startEdge = edge;
          if (!t.srcTarg(source, edge, edgesRead++))
          {
            return;
          }
        }
      }
    }
  }
}
```
</details>

<details>
<summary></summary>

```c++
```
</details>

## 五、优劣分析  
参考上述，为了实现在sparse模式下节省内存的目的，这里考虑两方便的处理：
1. 使用变长编码以节省小数值的内存空间
2. 使用差分编码以减小需要存储的数值，来尽可能利用1的优化
3. 使用RLE编码来加速解码效率并进一步节省空间

带来的缺点就是额外的解码延迟，以及额外的空间开销。  
PS： 理论上当数值超过28bit时，需要的存储空间会超过32bit，不过在差分编码下这种几率比较小。

## 六、伪代码实现
实现SparseRLEVertexSubset数据类型，实现VertexSubset接口：
```scala
trait VertexSubset {
  def free(): Unit // free data
  def getVertex(vid: Int): Int // obtain ith vertex
  def isIn(vid: Int): Boolean // sparse mode dont need impl
  def size: Int // subset size
  def vertexNumber: Int // all vertex count 
  def isEmpty: Boolean
  def isDense: Boolean // false
}
```
底层数据存储不需要对外暴露，但是需要接受Array[Int]作为构造参数。

考虑到每次调用getVertex都会产生解码开销，并且编码数据对与并行化的支持不太好，所以需要为框架API：EdgeMap, VertexMap提供对该类型的支持以优化对getVertex的并行调用。
### 伪代码
目标：所有使用SparseVertexSubset的地方都可以替换为新的结构  
两种方式：使用前解码 or 将外围调用当作回调，由方法触发

问题：
1. 想要保留接口使用，那么每次getVertex效率会很低: 使用chunk方式可以一定程度减轻
2. 除了特定算法中的实现使用了getVertex接口的，其他三个地方用到了该接口：edgeMap vertexMap changeToDense，这三个接口都需要遍历VertexSubset中的所有点，所以可以使用回调的方式提供接口.

综上，新的结构提供了两种访问方式：
1. 提供getVertex接口，通过串行解码得到想要的vid
2. 提供traverse接口，通过指定回调来遍历subset中的所有点信息，可以并行化处理

```scala
class SparseRLEVertexSubset extends VertexSubset {
  private data Array[byte]
  def this(vn, active, arr_int) {
    encode(arr_int) => data
  }

  // impl1, abandon
  def getVertex(idx int) {
    decode(arr_int) => data
    return data[idx]
  }

  // impl2 better than impl1, but same design
  def getVertex(idx int) {
    return decodeIndex(idx int)
  }
}

// 定义静态函数encode decode
object SparseRLEVertexSubset {
  def EncodeFirst()
  def EncodeOther()

  def encode(arr_int) {
    sort(arr_int)
    first_vid = arr_int[0]
    EncodeFirst(first_vid) => data
    start_vid = first_vid
    
    for vid in arr_int[1:] {
      diff = vid-start_vid
      EncodeOther(diff) => data
    }
  }

  def DecodeFirst()
  def DecodeOther()

  def decode(index int) {
    first_vid = DecodeFirst(data)
    start_vid = first_vid
    for i, item in data {
      diff = DecodeOther(item)
      vid = start_vid + diff
      if i==index {return vid}
      i++
    }
    exception
  }

  // TODO. 并行化
  def traverse(f func(vid Int) {}) {
    first_vid = DecodeFirst(data)
    start_vid = first_vid
    for idx, item in data {
      diff = DecodeOther(item)
      f(idx, start_vid + diff)
    }
  }
}
```

## 参考链接
[spMV]:(https://www.kkourt.io/papers/taco10-spmv-kkourt.pdf)

## 时间线
> 2023.05.28 yeon.guo ShangHai MinHang 迁移
