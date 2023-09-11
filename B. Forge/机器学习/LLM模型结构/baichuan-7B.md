**baichuan-7B** 是由[百川智能](https://www.zhihu.com/search?q=%E7%99%BE%E5%B7%9D%E6%99%BA%E8%83%BD&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3103302935%7D)开发的一个开源可商用的大规模预训练[语言模型](https://www.zhihu.com/search?q=%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3103302935%7D)。基于 Transformer 结构，在大约 1.2 万亿 [tokens](https://www.zhihu.com/search?q=tokens&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3103302935%7D) 上训练的 70 亿参数模型，支持中英双语，上下文窗口长度为 4096。在标准的中文和英文权威 benchmark（C-EVAL/MMLU）上均取得同尺寸最好的效果。

[官方训练代码](https://link.zhihu.com/?target=https%3A//github.com/baichuan-inc/baichuan-7B.git)

[模型下载](https://link.zhihu.com/?target=https%3A//huggingface.co/baichuan-inc/baichuan-7B)

整体模型基于标准的 Transformer 结构，采用了和 LLaMA 一样的模型设计:

- **Position Embedding：** 采用 [rotary-embedding](https://www.zhihu.com/search?q=rotary-embedding&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3103302935%7D)，是现阶段被大多数模型采用的位置编码方案，具有很好的外推性。  
- **Feedforward Layer：** 采用 SwiGLU，Feedforward 变化为(8/3)倍的隐含层大小，即11008。  
- **Layer Normalization:** 基于 RMSNorm 的 [Pre-Normalization](https://www.zhihu.com/search?q=Pre-Normalization&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3103302935%7D)。
### 源码结构

![[Pasted image 20230815144408.png]]
### 旋转位置编码

**旋转位置编码** 是 **相对位置编码** 的一种实现，**相对位置编码** 没有完整建模每个输入的位置信息，而是在计算 **Attention** 的时候考虑当前位置与被 Attention 的位置的相对距离，由于 **自然语言一般更依赖于相对位置，所以相对位置编码通常也有着优秀的表现**。

苏剑林老师利用复数的形式，构思了一种巧妙的相对位置编码算法，可以将 **绝对位置编码与相对位置编码融于一体**，详情见 [博采众长的旋转式位置编码](https://link.zhihu.com/?target=https%3A//kexue.fm/archives/8265)。

计算 Attention 时，关键之处在于向量的内积，先假设 $q_n、k_m$ 为二维向量，那么可以将它们当成二维向量，内积可表示为：$<q_{n}, k_{m}>=Re[q_{n}(k_{m})^*]$

两个复向量的内积，等于一个复向量与另一个复向量的共轭的乘积的实部。

![[Pasted image 20230815144707.png]]
![[Pasted image 20230815144735.png]]

源代码（只含关键逻辑）：

```python
class RotaryEmbedding(torch.nn.Module):
    def __init__(self, dim, max_position_embeddings=2048, base=10000, device=None):
        # 初始化 theta, inv_freq.shape: [dim/2]
        inv_freq = 1. / (base ** (torch.arange(0, dim, 2).float() / dim))
        self.register_buffer("inv_freq", inv_freq)

        # Build here to make `torch.jit.trace` work.
        self.max_seq_len_cached = max_position_embeddings
        # t.shape: [max_position_embeddings]
        t = torch.arange(self.max_seq_len_cached, device=self.inv_freq.device, dtype=self.inv_freq.dtype)
       
        # freqs.shape: [max_position_embeddings, dim/2]
        freqs = torch.einsum("i,j->ij", t, self.inv_freq)
       
        # 根据 rotation公式，[max_position_embeddings, dim/2] -> [max_position_embeddings, dim] 
        emb = torch.cat((freqs, freqs), dim=-1)
        self.register_buffer("cos_cached", emb.cos()[None, None, :, :], persistent=False)
        self.register_buffer("sin_cached", emb.sin()[None, None, :, :], persistent=False)

    def forward(self, x, seq_dim=1, seq_len=None): 
         return (
            # [1, 1, seq_len, dim]
            self.cos_cached[:, :, :seq_len, ...].to(dtype=x.dtype),
            self.sin_cached[:, :, :seq_len, ...].to(dtype=x.dtype),
        )


def rotate_half(x):    
    x1 = x[..., : x.shape[-1] // 2]
    x2 = x[..., x.shape[-1] // 2:]
    # 根据 rotation公式， 前半部分与后半部分互换，且后半部分取负
    return torch.cat((-x2, x1), dim=-1)


def apply_rotary_pos_emb_index(q, k, cos, sin, position_id):
    cos = cos.squeeze(1).squeeze(0)  # [seq_len, dim]
    sin = sin.squeeze(1).squeeze(0)  # [seq_len, dim]
    # 融入 token 的位置信息
    cos = cos[position_ids].unsqueeze(1)  # [bs, 1, seq_len, dim]
    sin = sin[position_ids].unsqueeze(1)  # [bs, 1, seq_len, dim]  

    # 计算旋转位置编码，参考 rotation公式
    q_embed = (q * cos) + (rotate_half(q) * sin)
    k_embed = (k * cos) + (rotate_half(k) * sin)
    return q_embed, k_embed


class Attention(torch.nn.Module):
    def __init__(self, hidden_size, num_attention_heads, position_encoding_2d=True):
        self.hidden_size = hidden_size
        self.num_heads = config.num_attention_heads
        self.head_dim = self.hidden_size // self.num_heads
        self.max_position_embeddings = config.max_position_embeddings

        # 初始化 RotaryEmbedding
        self.rotary_emb = RotaryEmbedding(self.head_dim, max_position_embeddings=self.max_position_embeddings)
        )

    def forward(self, position_ids):
        proj = self.W_pack(hidden_states)
        proj = proj.unflatten(-1, (3, self.hidden_size)).unsqueeze(0).transpose(0, -2).squeeze(-2)
         # batch_size x source_len x hidden_size
        query_states = proj[0].view(bsz, q_len, self.num_heads, self.head_dim).transpose(1, 2) 
        # batch_size x target_len x head_size
        key_states = proj[1].view(bsz, q_len, self.num_heads, self.head_dim).transpose(1, 2)  
        # batch_size x source_len x hidden_size
        value_states = proj[2].view(bsz, q_len, self.num_heads, self.head_dim).transpose(1, 2)  

        kv_seq_len = key_states.shape[-2]

         # 获取 cos, sin
        cos, sin = self.rotary_emb(value_states, seq_len=kv_seq_len)

        # 计算旋转位置编码
        query_states, key_states = apply_rotary_pos_emb(query_states, key_states, cos, sin, position_ids)
```

### RMSNorm

**RMSNorm**（Root Mean Square Layer Normalization），是一般 LayerNorm 的一种变体，**可以在梯度下降时令损失更加平滑**。

与 LayerNorm 相比，**RMSNorm** 的主要区别在于去掉了减去均值的部分（[re-centering](https://www.zhihu.com/search?q=re-centering&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3103302935%7D)），只保留方差部分（[re-scaling](https://www.zhihu.com/search?q=re-scaling&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3103302935%7D)），从归一化的表达式上可以直观地看出。

![[Pasted image 20230815144945.png]]

源代码：

```python
class RMSNorm(nn.Module):
    def __init__(self, hidden_size, eps=1e-6):
        """
        RMSNorm is equivalent to T5LayerNorm
        """
        super().__init__()
        self.weight = nn.Parameter(torch.ones(hidden_size))
        self.variance_epsilon = eps

    def forward(self, hidden_states):
        variance = hidden_states.to(torch.float32).pow(2).mean(-1, keepdim=True)
        hidden_states = hidden_states * torch.rsqrt(variance + self.variance_epsilon)

        # convert into half-precision if necessary
        if self.weight.dtype in [torch.float16, torch.bfloat16]:
            hidden_states = hidden_states.to(self.weight.dtype)

        return self.weight * hidden_states
```

### FlashAttention

