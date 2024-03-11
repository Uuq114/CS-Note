# Lecture
<!-- TOC -->

- [Lecture](#lecture)
  - [Lecture 1 - Search](#lecture-1---search)
  - [Lecture 2 - Knowledge](#lecture-2---knowledge)
  - [Lecture 3 - Uncertainty](#lecture-3---uncertainty)

<!-- /TOC -->
<!-- /TOC -->

## Lecture 1 - Search

搜索方法可以看成是一个图，state是node，action是directed edge，为了避免环造成搜索永远不会结束，搜索时需要维护一个explored node set

在迷宫问题中，一些解法：

- uninformed search：解决问题的方式从根本上是一样的，没有考虑probloem-specific knowledge
  - DFS/BFS
- informed search：用到了和问题相关的信息，更高效
  - greedy best-first search，GBFS，每次会选择离goal最近的节点，距离由启发式函数`h(n)`决定，曼哈顿距离是一个常用的启发式函数
  - A* algorithm

A* algorithm
每次移动时，考虑g(n)+h(n)最小的节点
g(n) = cost to reach node, h(n) = estimated cost to goal
移动到这个节点需要的步数+后续需要的步数

和GBFS相比，A*化的时间可能更多，但是最后的结果是更好的

在以下条件下，A*算法是最佳的算法：

- `h(n)` is admissable，即启发式函数不会高估true cost
- `h(n)` is consistent，即对每个节点n以及后继者n'，`h(n) <= h(n')+c`，c为cost of step

Adversarial Search
两边对抗

Minimax
对每种可能出现的情况赋值，便于计算机理解。比如井字棋最后X赢了是1，O赢了是-1，平局是0。
包含两个对抗的玩家，max player和min player，每个人的目的是为了得到更大/更小的比分。
在井字棋中minimax算法的逻辑：考虑自己的所有可能选项，同时换位思考对手可能选择的行动，以此来决定自己的行动，如此反复直至比赛结束。

```pesudo
Given state s:
  * MAX picks action a in ACTIONS(s) that produces highest value of MIN-VALUE(RESULT(s,a))
  * MIN picks action a in ACTIONS(s) that produces lowest value of MAX-VALUE(RESULT(s,a))
```

max player会先列出来所有可能的结果RESULT(s,a)，然后模拟让min player决策，再从所有min_value里面选出来最大的。（我感觉这种max决策更偏稳，比如某个min_value很小但是后续max player可能得高分，但是max会忽略这种情况）
换位思考是minimax算法的核心

Alpha-Beta Pruning
在minimax的决策推演中，存在一些可以剪枝的情况
alpha和beta代表两个要跟踪的值，目前最好的分数和目前最坏的分数

Depth-Limited Mimimax
因为有时无法评估所有可能的情况，minimax会限制计算的步数，比如10-12步
为了在游戏未结束时评分，需要一个评估函数，对某个状态的局面进行评分

## Lecture 2 - Knowledge

一些概念：

- knowledge-based agent：可以基于knowledge internal representation推理的agent
- sentence：定义了外部世界的规则，使用knowledge representaion语言
- 一些逻辑符号：非、与、或、$\rightarrow$（蕴含，implication）、$\leftrightarrow$（双向条件，biconditional，if and only if）
- model：对一些命题赋真值（指定T F？），例如P=xxx，Q=xxx，P=F Q=T
- knowledge base：knowledge-based agent知道的一系列sentence，简写为KB
- entailment：蕴含，但是这里指两个sentence的关系：如果在每一个model中A成立，那么B也成立，记为$\alpha \models \beta$。
- inference：推理。从旧sentence得到新sentence的过程

---

Implication真值表

| P | Q | P -> Q |
|---|---|--------|
| F | F | T |
| F | T | T |
| T | F | F |
| T | T | T |

其中P=F Q=T的情况，P->Q为T。这是因为implication P->Q表示P发生则Q发生，P不发生则Q不一定发生。因此P为F时，无论Q为什么，P->Q都是T

Biconditional真值表

| P | Q | P <-> Q |
|---|---|--------|
| F | F | T |
| F | T | F |
| T | F | F |
| T | T | T |

biconditional表示的是当且仅当，if and only if。因此只有P Q同真同假时，P->Q才是真

---

语义后承（semantic consequence），$\models$，一般连接一个命题集合和一个命题。
实质蕴涵（material implication / material conditional），$\to$，连接的是两个命题

---

在上一章的搜索问题中，也有一些定义，这些定义可以迁移到本章的Theorem Proving中：

- initial state：starting knowledge base
- action：inference rules
- transition model：new knowledge base after inference
- goal test：检查是否证明了命题
- path cost function：证明的步数

---

后面还有一些名词，First-Orde Logic，Universal Quantification等，完全看不懂，先放着吧

## Lecture 3 - Uncertainty

概率：$0 \le P(\omega) \le 1$，$\sum_{\omega \in \Omega}P(\omega) = 1$
非条件概率（unconditional probability）：没有其他条件下，某个事件发生的概率
条件概率（conditional probability）：给了一些条件，此时某个事件发生的概率

Bayesian Network：表示random variable之间的dependancy的数据结构

- 有向图，顶点表示random variable，x到y的有向边表示x是y的parent
- 每个x的概率为$P(x|parents(x))$

Inference（中文应译作“推理”？）

- query：对事件计算概率
- evidence：对某个事件观测得到的概率
- hidden variable：query和evidence之外的variable
- goal：计算$P(x|e)$

条件概率和联合概率成正比：$P(A \wedge B) = P(A | B)P(B)$，这里考虑$P(B)$是常数？反正就是推到了一个这样的结论。
在使用时，可以在计算条件概率时将其转化为联合概率：

$$
    P(Appointment | light, no) = \alpha P(Appointment, light, no) \\
    = \alpha [P(Appointment, light, no, on time) + P(Appointment, light, no, delayed)]
$$

得到一个公式，y代表hidden variable能取到的值：
$P(X|e) = \alpha P(X,e) = \alpha \sum_{y}{P(X,e,y)}$
Python有一些库，可以方便的计算条件概率，只需要自己定义贝叶斯网络中的节点就行了

Approximate
