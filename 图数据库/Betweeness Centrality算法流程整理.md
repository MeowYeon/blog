# Betweeness Centrality算法流程整理
# Betweeness Centrality算法流程整理

## 一、简介
BC算法也叫介质中心性，用来衡量图中某个节点在图中的重要性，相似的算法还有Degree Centrality, Clossness Centrality, 相关的算法描述请看参考链接。  
BC算法使用的描述中心性的方式是选择，经过该点的最短路径占全图所有最短路径的比例，或者说，图中经过该点的最短路径的数目。BC算法的意义在于找出图中的枢纽节点。  
现在，我们可以给个初步的计算方式，对单个节点计算任意两点最短路径中经过该节点的比例，此操作需要O(V^2)时间复杂度，然后每个节点都执行此操作，最后的时间复杂度是O(V^3)。

根据前面的结论，其实BC算法最大的问题在于计算复杂度太高，即使Brande BC算法也只是将复杂度降低到O(V^2)，所以在大图处理上无法直接应用BC算法。比较常用的做法是应用取样方式，在图中随机选择几个点应用BC算法，一定程度上与全局计算的结果是吻合的。

## 二、Brande BC 算法  
Brande给出了优化版本的BC算法可以将复杂度降到O(V^2)。该算法使用了与原生算法不同的指标，使用`δr•(v)`来描述在以r为源端的所有最短路径中，经过v的最短路径在其中所占的比例，虽然和原生的表述不太一致但一样可以描述出介质中心性。下面是ligra中给出的`δr•(v)`计算公式：
![BC-exp1](/home/transwarp/space/images/BC-exp1.png)

其中，`δrt(v)`描述的是在顶点r t的最短路径中，经过v的最短路径所占的比例。对于节点r来说，v的BC就是在r到所有其他节点的最短路径中v所占的比例之和。下面两个公式给出了公式1的计算过程：
![BC-exp23](/home/transwarp/space/images/BC-exp23.png)

其中的`Pr(v)`用来定义在以r为起点的BFS遍历树中，节点v的所有父节点的集合。

## 三、Ligra算法流程分析
根据前文给出的公式2和公式3，BC算法的计算过程分为两部分：
- 公式2用来计算以r为根的遍历树中，r到各个节点的最短路径数目，在伪代码中 32-36行描述该步骤。
- 公式3用来计算`δr•(v)`，根据公式`δr•(v)`依赖于`δr•(w)`的值，w是v节点的子结点，所以需要先计算子结点的BC值，所以该步骤的遍历是与第一步反向的过程，在伪代码42-46行描述该步骤。

下面给出ligra中Brande BC算法的伪代码：
![BC-alg1](/home/transwarp/space/images/BC-alg1.png)
![BC-alg2](/home/transwarp/space/images/BC-alg2.png)

## 四、Ligra代码分析
TODO.

## 五、关于使用EdgeMapDense-Forward简化BC算法的分析
EdgeMap 本身定义是基于出边的，如果希望不对图进行转置，无法进行BC算法。

## PS
eaglegraph 参考了networkx中BC算法实现，参见参考链接

## 参考链接
[图节点中心度分析](https://alphafan.github.io/posts/graph_centrality.html)  
[Brande BC算法论文](http://snap.stanford.edu/class/cs224w-readings/brandes01centrality.pdf)  
[networkx BC实现](https://github.com/networkx/networkx/blob/main/networkx/algorithms/centrality/betweenness.py)  

## 时间线
> 2023.05.28 yeon.guo ShangHai MinHang 迁移
