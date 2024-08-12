# Reflection

学完CS50 AI Intro纪念。

总的体验还不错，课程的内容比较广，涉及了搜索、知识表示、概率、最优化、机器学习、神经网络、NLP。难度也是入门水平，前面的lecture我还跟了一下，后面都是直接看slides直接做lab了。

先说一下lecture的体验。对于AI这门课，我觉得比较有挑战的是定义一个问题，将知识/规则/目标等通过机器能够理解的形式表示，比如loss函数，或者是NLP中的context-free grammar（忘记叫啥了，就是一个用各种词性的词组成句子的规则树）。还有最经典的搜索问题中的state/operation/goal这些，都很好地将问题表示为能够直接计算的模式。

下面说一下lab的体验吧。每个lab基本都是两个文件，一个是主程序，包括整个应用的初始化，定义一些类的交互等。另一个就是一个主要类的实现，而lab的部分就是实现这个类中的一些方法，让程序具有“智能”。难度还好，可以接受，前面的lab基本都是手搓，后面的各种调库，什么`scikit-learn`、`nltk`都是直接用的，确实实现很快。

最后是一些杂谈。之前在李沐的动态看到“随机梯度下降”一词，还不知道是什么意思（话说《动手学深度学习》这本也是一直todo状态，惭愧）。现在大概了解了SGD，是一种常用的loss函数优化方法。“梯度”是函数变化最快的点，“随机”是SGD和“梯度下降”的区别所在，“梯度下降”每次会向全局最优的方向迭代，且步长比较大。而SGD每次会随机抽取一部分样本，向梯度更新一次，“随机”会导致它走的也许不是正确的方向，步长也比较小。但是因为随机性，SGD并不容易陷入局部最优解。

最后的最后，用李沐的《用随机梯度下降来优化人生》中的一段做总结吧：
> 如果你的目标很复杂，简单的随机梯度下降反而效果最好。深度学习里大家都用它。关注当前，每次抬头瞄一眼世界，快速做个决定，然后迈一小步。小步快跑。只要你有目标，不要停，就能到达。

继续加油！