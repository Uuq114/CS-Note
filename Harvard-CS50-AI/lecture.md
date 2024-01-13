# Lecture
<!-- TOC -->

- [Lecture](#lecture)
  - [Lecture 1 - Search](#lecture-1---search)

<!-- /TOC -->
## Lecture 1 - Search

搜索问题的关键术语：

- state
- action
- transition model
- goal test
- path cost

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
