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

**第二段**讲的是 RNN 的特点，同时也是 RNN 的缺点。

在 RNN 里，给你一个序列的话，它的计算是把这个序列从左往右一步一步的向前做。假设你的序列是一个句子，它就是一个词一个词的看。对第 $t$ 个词，计算一个 $h_t$，叫做它的隐藏状态（hidden state），然后 $h_t$ 是由前面一个词的隐藏状态 $h_{t-1}$ 和当前第 $t$ 个词本身的输入决定的。这样子它就可以把前面学到的历史信息通过 $h_{t-1}$ 放到当下，然后和当前的词做一些计算，得到输出。这也是 RNN 如何能够有效处理时序信息的一个关键之所在，它把历史信息全部放在隐藏状态中，然后一个一个放下去。

但 RNN 的问题也来自于这里：
1. 并行度低。它是一个时序，也就是一步一步计算的过程，难以并行。因为你在计算第 $t$ 个词的隐藏状态 $h_{t}$ 的时候，必须保证已经完成了 $h_{t-1}$ 的计算。这样就无法利用 GPU，或主流加速器，比如 TPU，成千上万线程带来的计算性能的提升。
2. 处理长序列的能力差。也是因为一步一步计算的原因，历史信息是一步一步往后传递的，如果你的时序比较长的话，那么很早期那些时序信息可能会丢失。如果你不想丢掉这些历史信息，可能需要做一个比较大的 $h_t$，并在每个时间步都得把它存下来，导致内存开销变得很大。

当然，作者也指出过去这些年大家针对这些缺点做了很多改进，包括并行的改进，以及做一些分解的方法来提高并行度，但没有从本质上解决问题。

**第三段**讲 attention 在 RNN 中的应用。在本篇文章之前，attention 已经被成功地用在 encoder 和 decoder 里面了，主要用于怎么样吧 encoder 的信息有效地传给 decoder，就是说 attention 和 RNN 是一起使用的。

**第四段**提出 Transformer 模型，这是一个全新的模型，不再使用循环网络层，而是纯基于注意力机制。所以 Transformer 模型可以并行计算，这样可以在比较短的时间内做到比之前的模型更好的结果。

## 相关工作
使用卷积神经网络（Convolutional Neural Network）来替换循环神经网络（Recurrent Neural Network），使得减少时序的计算。CNN 的主要问题是对长序列难以建模。这是因为 CNN 做计算的时候，每一次它去看一个比较小的窗口，比如一个 `3x3` 的像素块；那么，如果两个像素隔得比较远，就需要做很多层卷积，一层一层叠上去，最后才能把这两个隔得远的像素给融合起来。使用 Transformer 里的注意力机制可以解决这个问题，每一次可以看到所有的像素，也就是说我一层就能看到整个序列，相对来说就没有这个问题。另外，卷积的优势是可以做多个输出通道，一个输出通道可以认为是它可以去识别不一样的模式，所以为了做到这种多输出通道的效果，作者提出了 Multi-Headed Attention，及多头的注意力机制，可以模拟 CNN 多输出通道的效果。

Self-attention，自注意力机制，Transformer 中一个关键性的技术点，这个工作其实之前已经有人提出来了，并不是本文的创新。

基于循环注意力机制的 Memory Networks，在 2017 年也是一个研究重点，证明了比 sequence-aligned recurrence 在简单语言 QA 和语言建模任务上表现得好。

Transformer 是第一个只依赖于自注意力来做 encoder-decoder 架构的模型。

## 模型
