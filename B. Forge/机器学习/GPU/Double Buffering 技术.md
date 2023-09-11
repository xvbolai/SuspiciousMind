**前言：**

每个芯片推出来的时候都会有各自的理论算力。但实际上，软件应用程序多大程度能利用上这些算力，一方面要看硬件设计的考虑能否切合应用场景，另一方面是软件工程师在编写应用程序代码的时候，是否能充分利用硬件资源。良好的软件工程师应该心中有一杆”性能“秤。

这篇文章主要聊的是double-buffering技术，也叫ping-pong缓冲技术。同时讨论，如何决定最优的数据搬运粒度。

**假设：**

- 应用程序是一个相同的计算操作(computation/operation) 在大的数据上(data array)
- 数据搬运通过DMA来搬运的，抽象的DMA语句是 `DMA_transfer(src_addr, dst_addr, transfer_size)`

如果你的代码是这样的

```python3
for i in range(block):
    DMA_get(X[i])
    DMA_wait

    Comp:Y[i]=f(X[i])

    DMA_put(Y[i])
    DMA_wait
```

那么当DMA在搬运数据时，计算单元是闲置状态，在等待数据，没有做到最大化硬件资源利用。

![](https://pic3.zhimg.com/80/v2-99e16dca0e51d4731221a621fb7e5b2e_1440w.webp)

如果使用Double Buffering模式来写代码，就可以使得DMA在搬运数据的时候，计算单元也在并行运行。

![](https://pic4.zhimg.com/80/v2-54d24c51f0fe9424d9d64d3c3927b623_1440w.webp)

Double Buffering

Double Buffering的本质思想是：把buffer分成两份，提前搬运下一轮数据的同时，计算本轮数据。这样在计算下一轮数据的时候，就已经有ready的数据了，节省了等待数据搬运的时间。

Double Buffering的编程模板是

```c++
inp_buf[2];
out_buf[2];

# prologue
DMA_get(inp_buf[0],x[0], event_in[0]); DMA_wait(event_in[0])
DMA_get(inp_buf[1],x[1], event_in[1]); Comp:out_buff[0]=f(inp_buff[0]); DMA_wait(event_in[1])
# iteration
for i in range (1，block-1):
    DMA_get(inp_buff[(i+1)%2], X[i+1], event_in[(i+1)%2])
    Comp:out_buff[i%2]=f(inp_buff[i%2])
    DMA_put(out_buff[(i-1)%2),even_out[(i-1)%2])

    DMA_wait(event_in[(i+1)%2])
    DMA_wait(even_out[(i-1)%2])
# epilogue
Comp:out_buff[(block-1)%2]=f(inp_buff[(block-1)%2]);DMA_put(out_buff[(block-2)%2),even_out[(block-2)%2]);DMA_wait(even_out[(block-2)%2])
DMA_put(out_buff[(block-1)%2),even_out[(block-1)%2])
```

**如何决定每次DMA搬运多少数据？**

_一个有意思的问题是：如何决定每次搬运多少数据，使得整体性能是最优的？_

这里对DMA搬运数据做了一个简单的建模：假设搬运x Byte数据，数据搬运需要的cycle数为：

$Transfer(x)=I + \alpha\times x$

其中， $I$ 是初始化DMA的需要的cycle数，和搬运数据量无关； $\alpha$ 是每搬运1 Byte数据需要的cycle数。

计算涉及x Byte数据，需要的cycle数为

$Comp(x) = w\times x$

有些场景下是数据搬运占主导，有些则是计算占主导。

![](https://pic3.zhimg.com/80/v2-c52340b6954619c9d1c5841ebe8a9ce6_1440w.webp)

假设我们总数据规模为N Byte，每个block搬运x Byte数据，那么我们希望寻找一个x （每次DMA搬运x Byte), 使得总耗时最少。

- Transfer主导：总时间=$(N/x + 2)\times(I + \alpha x)$
- Computation主导：总时间= $wN + 2(I + \alpha x)$

我设置了一组参数（$w > \alpha$)，画出以下两图。可以看到有一个分界点，在分界点之前是，数据搬运时间更长，在分界点之后，计算时间更长，总时间在临界点出得到最优（总占用cycle数最少）。

![](https://pic2.zhimg.com/80/v2-5fd214a383b8fa9ee61c70cb999e585d_1440w.webp)

- 当然可能存在一种情况是compute(x)永远在tranfer(x)上方，两条线没有交点，那么此时是计算占主导，x可以设为最小的DMA搬运单元。
- 另外一个情况是总是transfer占主导，那么x的最优取值应该是 local sram的upper bound。

不过这个模型是超级简化的版本，主要是理解背后的原理。实际操作上，有条件可以不断增加搬运数据的大小，测试DMA搬运数据和计算需要的cycle数，从而拟合出 $\alpha,I,w$ 的值；也可以软件上实现double-buffering后，更改每次搬运大小，测试性能，找到最优的数据搬运粒度。

[Double Buffering 每次该搬运多少数据？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/569285046)
