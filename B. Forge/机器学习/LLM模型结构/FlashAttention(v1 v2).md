![[Pasted image 20230813140514.png]]

## **前向传播**

![[Pasted image 20230813140254.png]]
![[Pasted image 20230813200246.png]]
![[Pasted image 20230813200312.png]]


## Softmax Tiling


![[Pasted image 20230813140320.png]]
## 图解内部结构

![[Pasted image 20230813140331.png]]

## V2 vs V1

![[1691919486096.png]]

**简单示例的FlashAttention完整计算步骤（红色部分表示V1和V2区别）：**

![[Pasted image 20230813173913.png]]

**FlashAttention-2的完整计算步骤（红色部分表示V1和V2区别）：**

![[Pasted image 20230813173927.png]]
![[Pasted image 20230813173934.png]]

下边是flashattention的伪代码

![[Pasted image 20230813174035.png]]

## Parallelism

FlashAttention在**batch和heads**两个维度上进行了并行化：使用一个thread block来处理一个attention head，总共需要thread block的数量等于**batch size × number of heads**。每个block被调到到一个SM上运行，例如A100 GPU上有108个SMs。当block数量很大时（例如≥80），这种调度方式是高效的，因为几乎可以有效利用GPU上所有计算资源。

但是在处理长序列输入时，由于内存限制，通常会减小batch size和head数量，这样并行化成都就降低了。因此，FlashAttention-2还在序列长度这一维度上进行并行化，显著提升了计算速度。此外，当batch size和head数量较小时，在**序列长度**上增加并行性有助于提高GPU占用率。

![[Pasted image 20230813194643.png]]
## **Work Partitioning Between Warps**

上一节讨论了如何分配thread block，然而在每个thread block内部，我们也需要决定如何在不同的warp之间分配工作。我们通常在每个thread block中使用4或8个warp，如下图所示:

![[Pasted image 20230813194724.png]]
![[Pasted image 20230813194756.png]]

###  reference

[FlashAttention2详解（性能比FlashAttention提升200%） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/645376942)
[FlashAttention图解（如何加速Attention） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/626079753?utm_id=0)
[一些改cuda加速的思路：FlashAttention、PagedAttention、LightSeq、ByteTransformer_taoqick的博客-CSDN博客](https://blog.csdn.net/taoqick/article/details/131382360)

![[Pasted image 20230824210705.png]]
