要想编写高效的程序，那么一定要对内存结构有比较深刻的认识，就像C/C++里面的堆内存，栈内存，全局存储区，静态存储区，常量区等。Cuda是并行计算框架，而GPU的内存有限，那么如果想编写高效的Cuda程序，首先要对其内存结构有一个简单的认识。

![[Pasted image 20230824205458.png]]
**寄存器（Register）**：对于GPU来说，寄存器是最快速的存储单元，它们用于存储线程的私有数据。但因为寄存器的数量有限，过多的使用可能会限制并发线程的数量。  

**共享内存（Shared Memory）**：共享内存是一个可以被同一个线程块中的所有线程共享的存储空间，它比全局内存要快得多，但容量较小。适用于需要线程间通讯的场景。

**本地内存（Local Memory）**：本地内存主要用于存储单个线程的数据，但其实质上是全局内存的一部分，因此访问速度较慢。

**纹理内存（Texture Memory）**：纹理内存是一种只读内存，具有特殊的缓存机制，适用于特定的数据访问模式。它对于处理图像和其他形式的网格数据非常有用。    

**常量内存（Constant Memory）**：常量内存是一种只读内存，用于存放不会在kernel执行过程中改变的数据。它被所有线程共享，并具有特殊的缓存机制。

**全局内存（Global Memory）**：全局内存是CUDA设备上的主要存储空间，所有的线程都可以读写，但访问速度最慢。

**主机内存（Host Memory）**：主机内存是CPU的RAM，访问速度比GPU内存更慢，一般在进行GPU和CPU间的数据传输时使用。

![[Pasted image 20230824205607.png]]

- 第一梯队：A100, MI100 (AMD), MI210 (AMD)  
- 第二梯队：A10, A5000/A6000, V100, T4 
- 第三梯度：国产卡

### A100 架构

![[Pasted image 20230824210705.png]]

a. 7 GPCs, 7 or 8 TPCs/GPC, 2 SMs/TPC, up to 16 SMs/GPC, **108 SMs** 
b. **64 FP32 CUDA Cores/SM**, **6912 FP32 CUDA Cores per GPU** 
c. 4 third-generation Tensor Cores/SM, 432 third-generation Tensor Cores per GPU
d. 5 HBM2 stacks, 10 512-bit memory controllers


GPU中每个SM都设计成支持数以百计的Thread并行执行，并且每个GPU都包含了很多的SM，所以GPU支持成百上千的Thread并行执行。当一个Kernel启动后，Thread会被分配到这些SM中执行。大量的Thread可能会分配到不同的SM，同一个block中的Threads必然在同一个SM中并行（SIMT）执行。每个Thread拥有它自己的程序计数器和状态寄存器，并且用Thread自己的数据执行指令， 这就是所谓的 SIMT( single instruction multiple thread)。

一个SP可以执行一个thread，但是实际上并不是所有的thread能够在同一时刻执行，Nvidia把32个threads组成一个warp，warp是调度和运行的基本单元。warp中所有thread并行的执行相同的指令。一个warp需要占用一个SM运行，多个warp需要轮流进入SM。由SM的硬件调度器warp schedule负责调度。目前每个warp包含32个threads。所以一个GPU上的resident thread最多只有 SM\*warp 个。

### registers：寄存器

它是GPU片上高速缓存器，执行单元可以以极低的延迟访问寄存器。寄存器的基本单元是寄存器文件（register file），每个寄存器文件大小为32bit。寄存器文件数量虽然客观，但是平均分给并行执行的线程，每个线程拥有的数量就非常有限了。**编程时，不要为每个线程分配过多的私有变量。下面程序中，aBegin,aEnd,aStep,a等变量都是寄存器变量，每个线程都会维护这些变量。** 
```c++
__global__ void registerDemo(float *B,float *A ,int wA)
{
    int aBegin = wA*BLOCK_SIZE * blockIdx.y;
	int aEnd = aBegin + wA - 1;
	int aStep = BLOCK_SIZE;
	
	for(int a=aBegin;a<=aEnd;a+=aStep)
	{
	    //...
	}
}
```

### local memory：局部存储器

- 位于堆栈中，不在寄存器中的所有内容
- 作用域为特定线程
- 存储在global内存空间中，速度比寄存器慢很多

对于每个线程，局部存储器也是**私有的。如果寄存器被消耗完，数据将被存储在局部存储器中。如果每个线程用了过多的寄存器，或声明了大型结构体或数组，或者编译期无法确定数组的大小，线程的私有数据就有可能被分配到local memory中。一个线程的输入和中间变量将被保存在寄存器或者局部存储器中。局部存储器中的数据将被保存在显存中，而不是片上的寄存器或者缓存中，因此对local memory的访问速度比较慢。**

### shared memory：共享存储器

- SM（SM = streaming multiprocessor）中的内存空间
- 最大48KB（现在A100是192KB）
- 作用域是线程块

共享存储器也是GPU片内的告诉存储器。它是一个块可以被同一block中的所有线程访问的可读写存储器。访问共享存储器的速度几乎和访问寄存器一样快。是实现线程间通信的延迟最小的方法。共享存储器可用于实现多种功能，如用于保存共用的计数器（例如计算循环迭代次数）或者block的公共结果（例如规约的结果）。
我们可以动态或者静态的分配shared Memory，其声明即可以在kernel内部也可以作为全局变量。其标识符为：__ shared __。
下面这句话静态的声明了一个2D的浮点型数组：
__shared\_\_ float tile\[size_y\]\[size_x\];
如果在kernel中声明的话，其作用域就是kernel内，否则是对所有kernel有效。
如果shared Memory的大小在编译器未知的话，可以使用extern关键字修饰，例如下面声明一个未知大小的1D数组：
extern __shared__ int tile\[\];
由于其大小在编译器未知，我们需要在每个kernel调用时，动态的分配其shared memory，也就是最开始提及的第三个参数：
kernel<<<grid, block, isize * sizeof(int)>>>(...)
应该注意到，只有1D数组才能这样动态使用。

static variable使用shared memory：

```c++
#include<iostream>
#include<stdio.h>
 
#if 1
__global__ void example(float *u)
{
	int i = threadIdx.x;
	__shared__ int tmp[4];
	tmp[i] = u[i]; 
	u[i] = tmp[i] * tmp[i] + tmp[3-i] ;
}
#endif
 
#if 1
 
int main()
{
	float host_u[4] = {1,2,3,4};
	float * dev_u ; 
	size_t size = 4*sizeof(float);
 
	cudaMalloc(&dev_u , size);
	cudaMemcpy(dev_u,host_u,size,cudaMemcpyHostToDevice);
 
	example<<<1,4>>> (dev_u);
 
	cudaMemcpy(host_u , dev_u , size , cudaMemcpyDeviceToHost);
 
	cudaFree(dev_u);
 
	for(int i=0;i<4;i++)
		printf("%f\n",host_u[i]);
	return 0;
}
 
#endif
```
dynamic variable使用shared memory：
```c++
#include<iostream>
#include<stdio.h>
 
__global__ void example(float *u)
{
	int i = threadIdx.x;
  extern __shared__ int tmp[];
	tmp[i] = u[i];
	u[i] = tmp[i] * tmp[i] + tmp[3-i];
}
 
int main()
{
	float host_u[4] = {1,2,3,4};
	float * dev_u;
	size_t size = 4*sizeof(float);
 
	cudaMalloc(&dev_u,size);
	cudaMemcpy(dev_u , host_u ,size , cudaMemcpyHostToDevice);
	example<<<1,4,  size>>>>(dev_u);
 
	cudaMemcpy(host_u, dev_u,size,cudaMemcpyDeviceToHost);
	cudaFree(dev_u);
	for(int i=0;i<4;i++)
		printf("%f ",host_u[i]);
	return 0;
}
```

#### 共享内存-Bank Conflict

为了获得较高的内存带宽，共享存储器被划分为多个大小相等的存储器模块，称为bank，可以被同时访问。因此任何跨越b个不同的内存bank的对n个地址进行读取和写入的操作可以被同时进行，这样就大大提高了整体带宽 ——可达到单独一个bank带宽的b倍。但是很多情况下，我们无法充分发挥bank的功能，以致于shared memory的带宽非常的小，这可能是因为我们遇到了bank冲突。

当一个warp中的不同线程访问一个bank中的不同的字地址时，就会发生bank冲突。
如果没有bank冲突的话，共享内存的访存速度将会非常的快，大约比全局内存的访问延迟低100多倍，但是速度没有寄存器快。然而，如果在使用共享内存时发生了bank冲突的话，性能将会降低很多很多。在最坏的情况下，即一个warp中的所有线程访问了相同bank的32个不同字地址的话，那么这32个访问操作将会全部被序列化，大大降低了内存带宽。

共享内存的地址映射方式

要解决bank冲突，首先我们要了解一下共享内存的地址映射方式。
在共享内存中，连续的32-bits字被分配到连续的32个bank中，这就像电影院的座位一样：一列的座位就相当于一个bank，所以每行有32个座位，在每个座位上可以“坐”一个32-bits的数据(或者多个小于32-bits的数据，如4个char型的数据，2个short型的数据)；而正常情况下，我们是按照先坐完一行再坐下一行的顺序来坐座位的，在shared memory中地址映射的方式也是这样的。下图中内存地址是按照箭头的方向依次映射的：
![[Pasted image 20230824232633.png]]
上图中数字为bank编号。这样的话，如果你将申请一个共享内存数组(假设是int类型)的话，那么你的每个元素所对应的bank编号就是地址偏移量(也就是数组下标)对32取余所得的结果，比如大小为1024的一维数组myShMem：

- myShMem\[4\]: 对应的bank id为#4 (相应的行偏移量为0)
- myShMem\[31\]: 对应的bank id为#31 (相应的行偏移量为0)
- myShMem\[50\]: 对应的bank id为#18 (相应的行偏移量为1)
- myShMem\[128\]: 对应的bank id为#0 (相应的行偏移量为4)
- myShMem\[178\]: 对应的bank id为#18 (相应的行偏移量为5)
下面我介绍几种典型的bank访问的形式。
下面这这种访问方式是典型的线性访问方式(访问步长(stride)为1)，由于每个warp中的线程ID与每个bank的ID一一对应，因此不会产生bank冲突。
![[Pasted image 20230901220623.png]]
下面这种访问虽然是交叉的访问，每个线程并没有与bank一一对应，但每个线程都会对应一个唯一的bank，所以也不会产生bank冲突。
![[Pasted image 20230901220652.png]]
下面这种虽然也是线性的访问bank，但这种访问方式与第一种的区别在于访问的步长(stride)变为2，这就造成了线程0与线程28都访问到了bank 0，线程1与线程29都访问到了bank 2…，于是就造成了2路的bank冲突。我在后面会对以不同的步长(stride)访问bank的情况做进一步讨论。
![[Pasted image 20230901220734.png]]
下面这种访问造成了8路的bank冲突。

![[Pasted image 20230901220815.png]]

_这里我们需要注意，下面这两种情况是两种特殊情况：_

![[Pasted image 20230901220827.png]]

上图中，所有的线程都访问了同一个bank，貌似产生了32路的bank冲突，但是由于广播(broadcast)机制(当一个warp中的所有线程访问一个bank中的同一个字(word)地址时，就会向所有的线程广播这个字(word))，这种情况并不会发生bank冲突。

![[Pasted image 20230901220857.png]]

这就是所谓的多播机制(multicast)——当一个warp中的几个线程访问同一个bank中的**相同字地址**时，会将该字广播给这些线程。

NOTE:这里的多播机制(multicast)只适用于计算能力2.0及以上的设备.

详细请见 [共享内存之bank冲突](https://segmentfault.com/a/1190000007533157?utm_source=tag-newest)
### constant memory：常数存储器

属于全局内存，大小64KB
线程请求同一个数据时很快，请求不同的数据时性能下降
在运行中不变，所有constant变量的值必须在kernel启动之前从host设置
__ global \_\_函数参数通过 constant memory穿的到device端， 限定4 KB，即kernel参数通过常量内存传递

它是只读的地址空间。常数存储器中的数据位于显存，但拥有缓存加速。常数存储器的空间较小，在Cuda程序中用于存储需要频繁访问的只读参数。当来自同一half-warp的线程访问常数存储器中的同一数据时，如果发生缓存命中，那么只需要一个周期就可以获得数据。常数存储器有缓存机制，用以节约带宽，加快访问速度。每个SM拥有8KB的常数存储器缓存。常数存储器是只读的，因此不存在缓存一致性问题。

constant memory的使用：

```c++
#include<iostream>
 
using namespace std;
 
__constant__ int devVar = 100;
 
__global__ void xminus(int *a)
{
	int i = threadIdx.x;
	a[i] = devVar+i;
}
 
int main()
{
	int *h_a = (int*)malloc(4*10) ;
	int *d_a ;
	cudaMalloc(&d_a, 4*10) ;
	cudaMemset(d_a, 0, 40) ;
 
	xminus<<<1,4>>>(d_a);
	
	cudaMemcpy(h_a, d_a, 4*10, cudaMemcpyDeviceToHost) ;
 
	for(int i = 0; i < 4 ; i++)
		cout << h_a[i] << " " ;
	cout << endl ;
}
```

### texture memory：纹理存储器

类似constant memory，是只读内存，以某种形式访问的时候可以提升性能。原本是用在OpenGL和DirectX渲染管线中的。  
有用的特点：

- 不需考虑要聚合coalescing访问的问题
- 通过“CUDA Array”进行缓存的2D或3D空间的数据位置
- 在1D，2D或3D数组上进行快速插值
- 将整数转换为“unitized”浮点数

### [global](https://so.csdn.net/so/search?q=global&spm=1001.2101.3001.7020) memory：全局存储器

- 独立于GPU核心的硬件RAM
- GPU绝大多数内存空间都是全局内存
- 全局内存的IO是GPU上最慢的IO形式（除了访问host端内存）

全局存储器位于显存（占据了显存的绝大部分），CPU、GPU都可以进行读写访问。整个网格中的任意线程都能读写全局存储器的任意位置由于全局存储器是可写的。全局存储器能够提供很高的带宽，但同时也具有较高的访存延迟。显存中的全局存储器也称为线性内存。线性内存通常使用cudaMalloc()函数分配，cudaFree()函数释放，并由cudaMemcpy()进行主机端与设备端的数据传输。

此外，也可以使用__device__关键字定义的变量分配全局存储器，这个变量应该在所有函数外定义，必须对使用这个变量的host端和device端函数都可见才能成功编译。在定义__device__变量的同时可以对其赋值。

```c++
#include<stdio.h>
#include<iostream>
 
__device__ float devU[4];
__device__ float devV[4];
 
//__global__ function 
__global__ void addUV()
{
	int i = threadIdx.x;
	devU[i] += devV[i];
}
 
int main()
{
	float hostU[4] = {1,2,3,4};
	float hostV[4] = {5,6,7,8};
	
	int size = 4* sizeof(float);
 
	//cudaMemcpyToSymbol:将数据复制到__constant__或者__device__变量中
	//cudaMemcpyFromSymbol:同上相反
	//cudaMalloc:在设备端分配内存
	//cudaMemcpy:数据拷贝
	//cudaFree():内存释放
	//cudaMemset():内存初始化
	cudaMemcpyToSymbol(devU,hostU,size,0,cudaMemcpyHostToDevice);
	cudaMemcpyToSymbol(devV,hostV,size,0,cudaMemcpyHostToDevice);
 
	addUV<<<1,4>>>();
 
	cudaMemcpyFromSymbol( hostU,devU,size,0,cudaMemcpyDeviceToHost );
 
	for(int i=0;i<4;i++)
		printf("hostU[%d] = %f\n",i,hostU[i]);
	return 0;
}
```


dynamic variable使用global memory：

global_mem_dynamic.cu:

```c++
#include<iostream>
#include<stdio.h>
 
__global__ void add4f(float *u , float *v)
{
	int i = threadIdx.x;
	u[i] += v[i];
}
 
void print(float * U ,int size)
{
	for(int i=0;i<4;i++)
	{
		printf("U[%d] = %f\n",i,U[i]);
	}
}
 
int main()
{
	float hostU[4] = {1,2,3,4};
	float hostV[4] = {5,6,7,8};
 
	float * devU ;
	float * devV ; 
	int size = sizeof(float) * 4;
	
	//在设备内存上分配空间
	cudaMalloc( &devU,size );
	cudaMalloc( &devV,size );
	//数据拷贝
	cudaMemcpy( devU ,hostU ,size ,cudaMemcpyHostToDevice );
	cudaMemcpy( devV ,hostV ,size ,cudaMemcpyHostToDevice );
 
	add4f<<<1,4>>> (devU,devV);
	//数据返回
	cudaMemcpy(hostU,devU,size,cudaMemcpyDeviceToHost);
 
	print(hostU,size);
	//释放空间
	cudaFree(devV);
	cudaFree(devU);
	return 0;
}
```

[cuda 学习之内存层次结构_请说明 register,shared,global 以及 constant 四类 cuda 内 存_xukang95的博客-CSDN博客](https://blog.csdn.net/xukang95/article/details/102855750)