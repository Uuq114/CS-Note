# Deep Reinforcement Learning Note

王树森 Youtube 的《深度强化学习》视频笔记

## 基本概念

- 随机变量（random variable）。一般用大写字母 $X$ 表示随机变量，用小写字母 $x$ 表示观测值
- 概率密度函数（probability density function，PDF）。$p(x)$ 表示观测值为 $x$ 的概率。类型有连续概率分布、离散概率分布
- 期望（expectation）。对于连续概率分布，$E(f(X))=\int_x{p(x)f(x)}dx$。对于离散概率分布，$E(f(X))=\sum_{x \in X}{p(x)f(x)}$
- 随机抽样（random sampling）

RL 术语：

- state
- action，agent 的动作
- policy，策略，记为 $\pi$ 函数。$\pi$ 是个概率密度函数，$\pi(a|s)=P(A=a|S=s)$，表示给定状态 s 下执行动作 a 的概率
- reward，奖励，一般需要自己定义。
- state transitition。执行 action 之后，old state 变成 new state。状态转移可以是随机的（受环境随机因素影响）。
- return，回报、未来累计奖励。$U_t=R_t+R_{t+1}+...$。未来的奖励有不确定性
- disconted return，未来累计折扣奖励。$U_t=R_t+\gamma R_{t+1}+\gamma^2 R_{t+2}+...$。$\gamma$ 是折扣率，是超参数需要自己调

RL 中的随机性来源：

- action 的随机性
- 状态转移的随机性

Return & Value 术语：

- action-value function，动作价值函数。$Q_{\pi}(s_t,a_t)=E[U_t|S_t=s_t,A_t=a_t]$。discounted return $U_t$ 依赖当前和后续的 action/state，没法求具体值，只能求期望。另外，动作价值函数 $Q_{\pi}$ 和 $\pi$ 函数也有关
- optimal action-value function，最优动作价值函数。所有 $Q_{\pi}$ 中最大的，$Q^\star(s_t,a_t)=\max_{\pi}Q_{\pi}(s_t,a_t)$。这个函数可以判断哪个动作价值最高
- state-value function，状态价值函数。$V_{\pi}$ 是 $Q_{\pi}$ 的期望，$V_{\pi}(s_t)=E_A[Q_{\pi}(s_t,A)]$。这里关于随机变量 A 求期望。$V_{\pi}$ 可以用来判断局势

总结：
- action-value function $Q_\pi(s,a)$。如果使用 policy $\pi$，$Q_\pi$ 可以判断状态 s 下执行动作 a 是否明智
- state-value function $V_\pi(s)$。可以评价状态 s 下情况的好坏，快赢了还是快输了

让 AI 控制 agent 的两种方法：
- 已知一个足够好的 policy $\pi(a|s)$。那么，可以根据 $s_t$，从 $\pi(s)$ 中随机抽样 $a_t$
- 已知最佳动作价值函数 $Q^\star(s,a)$。那么，可以根据 $s_t$，直接选能让 $Q^\star$ 最大的 action

** 因此，强化学习的目标就是学到 policy 函数 $\pi(a|s)$ 或者 $Q^\star$ 函数 $Q^\star(s,a)$**。
