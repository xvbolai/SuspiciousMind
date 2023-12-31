## Day Planner
- [ ] 单词

| 单词        | 意思 |
| ----------- | ---- |
| granularity | 粒度 |
|             |      |


举一个简单的例子来说明混合精度量化在计算目标函数梯度时可能会出现的问题。

假设有一个简单的神经网络模型，包含一个全连接层，其输入为 $x$，权重为 $w$，激活函数为 sigmoid 函数 $\sigma(x)=\frac{1}{1+e^{-x}}$。目标函数为模型的输出值，即 $y = \sigma(wx)$，我们需要计算目标函数对权重 $w$ 的二阶导数。

根据链式法则，目标函数对权重 $w$ 的一阶导数可以表示为：

$$\frac{\partial y}{\partial w} = \sigma'(wx) \cdot x$$

其中 $\sigma'(x) = \sigma(x)(1 - \sigma(x))$ 是 sigmoid 函数的一阶导数。

目标函数对权重 $w$ 的二阶导数可以表示为：

$$\frac{\partial^2 y}{\partial w^2} = \sigma''(wx) \cdot x^2 + \sigma'(wx) \cdot \frac{\partial x}{\partial w}$$

其中 $\sigma''(x) = \sigma(x)(1-\sigma(x))(1-2\sigma(x))$ 是 sigmoid 函数的二阶导数。根据全连接层的定义，我们有 $\frac{\partial x}{\partial w} = x$。

现在假设我们使用混合精度量化，将权重 $w$ 量化为 8 位整数，将输入 $x$ 量化为 16 位浮点数。在计算目标函数的二阶导数时，如果直接使用上面的公式进行计算，由于权重和输入的精度不同，会导致计算结果的误差放大，从而影响梯度的计算精度。因此，为了避免这种误差放大的影响，我们需要使用其他更加适合混合精度量化的计算方法，例如使用高精度的计算方法来计算梯度，或者使用更加精细的精度设置来尽量减小误差放大的影响。