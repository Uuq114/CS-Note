# CMU 11-617: Large Language Models: Methods and Applications

<!-- TOC -->

- [CMU 11-617: Large Language Models: Methods and Applications](#cmu-11-617-large-language-models-methods-and-applications)
  - [Language Models Basics](#language-models-basics)
  - [Neural Language Model Architectures](#neural-language-model-architectures)

<!-- /TOC -->

## Language Models Basics

语言模型：从 n-gram model 进化为 neural language model

**n-gram model**

链式计算概率：

![alt text](img/image.png)

以 2-gram 为例，每个窗口两个词

![alt text](img/image-1.png)

**neural language model**

计算下一个 token 的条件概率时，和 n-gram 的方法不同：

![alt text](img/image-2.png)

分类，基于计算方式区别：

- 根据前缀，预测下一个词。计算的序列概率是非条件概率。例如 GPT、LLaMA
- 根据前缀和额外的条件序列，预测下一个词。也叫 seq2seq 模型。计算的序列概率是条件概率。例如 T5、机器翻译模型。

![alt text](img/image-3.png)

理论上，conditioned 和 unconditioned 可以相互转换。

![alt text](img/image-4.png)

**language models terms**

以一个序列生成为例，给了三个词 I、eat、the。
第一个部分是 tokenization

![alt text](img/image-5.png)

tokenization 将 text 变成 token sequence。vocabulary 是所有 token 的 list，根据级别不同（character/subword/word），生成 seq 的 length 会变化

![alt text](img/image-6.png)

第二部分，embedding。为了让 NN 能处理，需要将离散的 token 变成连续的 vector。这一步构建一个 embedding matrix，大小是 vocabulary size * embedding dim

![alt text](img/image-7.png)

第三部分，NN。

NN 的输入：input seq 每个 token 的 vector

![alt text](img/image-8.png)

NN 的中间：每一层产生并向前传递 hidden state vector

![alt text](img/image-9.png)

NN 的输出：表示生成 token 的 embedding

![alt text](img/image-10.png)

第四部分，Logits。

predicted embedding => logits（未归一化得分）===softmax===> probability

NN 预测的 enmedding 乘以 vocabulary embedding matrix，可以为 vocabulary 的每个 word 计算一个分数，这个分数称为 logits

第五部分，使用 softmax，将 logits 变成概率

以序列生成任务为例，使用 softmax 可以让我们知道下一个词的概率分布

![alt text](img/image-11.png)

损失函数衡量该分布和真实的匹配程度。以负对数似然的损失函数为例，我们需要最大化真实序列的对数似然，因此需要最小化负对数似然。让模型学会什么是合理的语言

第六部分，Decoding。

我们需要根据下一个词的概率分布、取样算法，决定生成的下一个词。

- 选择概率最大的词。
- 根据 model 返回的概率分布，直接随机抽样。直接采样的后果是可能选到 softmax 的长尾噪声，这些词是低质量的。由于语言模型的 softmax 分布通常更平坦，因此尾部词的总概率不低，如 10~30%
- 根据温度随机采样。温度为 0 时接近 argmax 采样，温度越高时尾部的采样概率越高
- top-k 采样。只从 k 个高概率词采样，排除低质量 / 离群词，引入稀疏性。k 一般取值 10~50
- nucleus sampling，核采样，也称 top-p 采样。和 top k 采样类似，都是从高概率词取样。区别在于，top p 的 k 是不确定的，取决于概率和大于（例如）0.9 的 token 个数
- beam search，束搜索。一种在序列生成中平衡搜索质量和计算开销的搜索算法，假设质量最高的序列是概率最大的序列，并搜索这个序列。

例如，beam size=2 时，每次会搜索概率最大的两个分支

![alt text](img/image-12.png)

beam search 的问题：
对于开放问题（对话、故事生成），概率最高的生成序列不一定是最好的序列。即 beam search 生成的是更像人类写的文本，但是不够 surprising

decoding 策略的 trade-off：
质量、多样性、效率。

质量依赖高概率路径 → 需抑制尾部噪声 → 牺牲多样性；
多样性依赖探索低概率但合理路径 → 需保留尾部 → 风险引入错误。

![alt text](img/image-14.png)

![alt text](img/image-13.png)

其他的 generation parameter：

- frequency penalty。一个 token 已经出现的次数越多，减少后续输出的概率越小
- presense penalty。减少一个 token 出现的概率。
- stopping criteria。在输出 k 个 token / 特定 token 后，结束输出

## Neural Language Model Architectures

给定一个前缀的 token 序列，如何计算下一个 token 的条件概率？

neural language model 的方法：整个序列的联合概率等于每个 token 的条件概率乘积。使用 NN 可以让每个条件概率项可计算、可泛化、非零，从而让概率的积变成真实联合概率的有效近似。

![alt text](img/image-15.png)

Encoder-Decoder 架构

将输入序列压缩成一个或一组 context vector，再由 decoder 基于该表示逐步生成输出序列

![alt text](img/image-16.png)

Encoder: input seq -> hidden states

- 输入序列是变长的
- 输出 vector summary

Decoder: hidden states + target seq -> next token prediction

- target seq 是变长的
- decoder 逐个处理 target seq 的 token 并预测下一个 token。（为什么要重复生成 target seq 的 token？因为要让模型学会每一步的分布）

Recurrent Neural Network (RNN)

- 输入：embedding seq
- recurrent unit（循环单元）接收前一个 hidden state 和需要处理的 token embedding
- 输出：predicted embedding + updated hidden state

![alt text](img/image-17.png)

