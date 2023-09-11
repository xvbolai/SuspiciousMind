
# vLLM

源自vLLM: Easy, Fast, and Cheap LLM Serving with PagedAttention这篇paper，关键的技术有两点PagedAttention和内存共享：

## PagedAttention

### KVCache

KV Cache是大模型推理优化的一个常用技术，该技术以空间换时间的思想，通过使用上次推理的KV缓存，可以在不影响任何计算精度的前提下，提高推理性能，降低端到端的时延。

以GPT为代表的Decoder-Only自回归语言模型在生成每一个新的 token 时，接受所有之前生成的 tokens 作为输入。然而，对于这些先前生成的 tokens，每次生成新的 token 时都需要重新计算他们的表示，这个过程造成了大量的计算浪费。KV Cache 的引入就是为了解决这个问题。

KV Cache实质上是存储了之前计算过的 key-value 对用于下一个Token的生成。在 Transformer 结构中，self-attention 中的k_proj, v_proj会将输入的每个 token 转化为一个 key 和一个 value，然后使用这些 key-value 以及当前的query对来计算下一个 token。引入 KV Cache，我们就可以将之前生成的 tokens 对应的 key-value 对存储起来，当生成新的 token 时，直接从 KV Cache 中取出这些已经计算好的 key-value 对，再把当前token的key-value做一个连结在进行计算，这样就避免了KV的重复计算，大大提高了计算效率。

到huggingface代码里看，例如Hugging Face的transformers库代码实现就比较清爽，在modeling_gpt2.py中Attention部分相关代码如下：

```c++
	query = self._split_heads(query, self.num_heads, self.head_dim)
	key = self._split_heads(key, self.num_heads, self.head_dim)
	value = self._split_heads(value, self.num_heads, self.head_dim)

	if layer_past is not None: # 当输出第一个token后，layer_past就是非None了
		past_key, past_value = layer_past # 取出之前计算好的 key, value
		key = torch.cat((past_key, key), dim=-2) # past_key 与当前 token 对应的 key 拼接
		value = torch.cat((past_value, value), dim=-2) # past_value 与当前 token 对应的 value 拼接

	if use_cache is True:
		present = (key, value)
	else:
		present = None
```

整体来说，使用KV Cache包含以下两个步骤：

**预填充阶段**：在计算第一个输出token过程中，此时Cache是空的，计算时需要为每个 transformer layer 计算并保存key cache和value cache，在输出token时Cache完成填充；FLOPs同KV Cache关闭一致，存在大量gemm操作，推理速度慢，这时属于Compute-bound类型计算。

**KV Cache阶段**：在计算第二个输出token至最后一个token过程中，此时Cache是有值的，每轮推理只需读取Cache，同时将当前轮计算出的新的Key、Value追加写入至Cache；FLOPs降低，gemm变为gemv操作，推理速度相对第一阶段变快，这时属于Memory-bound类型计算。

### PagedAttention

通过KV Cache的技术，我们已经可以极大地提升LLM地推理速度，但是现有的Cache仍存在一些问题，

![[Pasted image 20230825150111.png]]

[一些改cuda加速的思路：FlashAttention、PagedAttention、LightSeq、ByteTransformer_taoqick的博客-CSDN博客](https://blog.csdn.net/taoqick/article/details/131382360)


