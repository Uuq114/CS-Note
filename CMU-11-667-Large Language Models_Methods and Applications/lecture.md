# CMU 11-617: Large Language Models: Methods and Applications

<!-- TOC -->

- [CMU 11-617: Large Language Models: Methods and Applications](#cmu-11-617-large-language-models-methods-and-applications)
  - [Language Models Basics](#language-models-basics)

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

以一个序列生成为例，给了三个词 I、eat、the。第一个部分是 tokenization

![alt text](img/image-5.png)

tokenization 将 text 变成 token sequence。vocabulary 是所有 token 的 list，根据级别不同（character/subword/word），生成 seq 的 length 会变化

![alt text](img/image-6.png)

第二部分，embedding。为了让 NN 能处理，需要将离散的 token 变成连续的 vector。这一步构建一个 embedding matrix，大小是 vocabulary size * embedding dim

![alt text](img/image-7.png)

第三部分，NN。

NN input：input seq 的每个 token 的 vector