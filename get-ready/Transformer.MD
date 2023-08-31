# Transformer[2017]

- 视频：[Transformer论文逐段精读](https://www.bilibili.com/video/BV1pu411o7BE)
- 论文：[Attention Is All You Need](https://arxiv.org/pdf/1706.03762.pdf)

## 摘要
 
**现状：** 

目前主流的序列转录模型（sequence transduction models）通常依赖于复杂的循环或卷积神经网络，并且包括一个 encoder 和一个 decoder；表现最好的模型通常还会使用一个注意力机制来连接 encoder 和 decoder。

**本文工作：** 

本文提出了一个新的简单的架构，Transformer，（第一次听说 Transformer 的中文名叫变形金刚），这个模型仅依赖于注意力机制，完全没有使用到循环和卷积网络。

**效果：** 

在两个机器翻译的实验上，Transformer 展现了更好的并行度，并且使用更少的时间来训练，并且达到了不逊于复杂模型的效果。
- WMT 2014 English-to-German: BLEU 分数达到 28.4，高于目前最好结果 2 个 BLEU
- WMT 2014 English-to-French: 在 8 个 GPU 上训练了 3.5 天之后，达到了单个模型的最高 BLEU 分数 41.8

作者成功将 Transformer 模型应用在了 English constituency parsing 任务上也证明了模型有好的泛化性。

> 可以看出，Transformer 一开始提出时主要是针对**机器翻译**这个小任务的，但是随着 BERT、GPT 都应用了 Transformer 架构，使它火出圈了之后，Transformer 也开始被用于图像，视频等任务。

## 结论

1. 本文提出一个新的序列转录模型，Transformer，仅使用 attention，把之前 encoder-decoder 架构通常会用到的循环层全部换成了 multi-headed self-attention。
2. 对机器翻译的任务来说，Transformer 训练的更快，并且能得到更好的效果。
3. 我们对 attention-based 模型的未来感到兴奋，并且计划将它应用到更多任务上，包括文本以外的输入和输出模式，以及研究局部的有限的的注意力机制来有效地处理大型的输入输出，例如图像，音频和视频。另外一个研究方向是使得生成不那么时序化。

访问[代码](https://github.com/tensorflow/tensor2tensor)。

## 导言

这里的导言写的比较短，基本上可以认为是摘要前面一半的扩充。

**第一段**讲在时序模型中，当前（2017年）最常用的是 RNN，包括 LSTM（long short-term memory），GRU（gated recurrent neural）。在这里有两个比较主流的模型，一是循环语言模型（recurrent language model），第二个是当输出结构化信息比较多的时候会用的 encoder-decoder 结构。

**第二段**讲的是 RNN 的特点，同时也是 RNN 的缺点。在 RNN 里，给你一个序列的话，它的计算是把这个序列从左往右一步一步的向前做。假设你的序列是一个句子，它就是一个词一个词的看。对第 $t$ 个词，计算一个 $h_t$，

## 相关工作