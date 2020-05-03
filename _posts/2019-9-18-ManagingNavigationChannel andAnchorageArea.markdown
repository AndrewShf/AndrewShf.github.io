---
layout: math
title: 优化航道和停泊地的管理 (运筹)
date: 2019-09-18 22:10
post-link:
---



## [Managing Navigation Channel Traffic and Anchorage Area Utilization of a Container Port][paper]

这是在大四上学期读的一篇论文，是***Transportation Science***上面的。我连续读了5天，运筹学可真是难啊... 可惜没能和他人充分的讨论，这里只是我个人的理解。

### 我的理解:

**Section 1**: 论文通过考虑了tidal windows, safty clearance, berthing position , capacity of anchorage area 进行了建立了一个MILP模型（并做了简化和假设），目标函数是三个penalty的加和。

**Section 2**: 证明该问题是一个strong NP hard问题：通过得到一个P问题的实例(s+2r只船 s > r，s个buffer)。分别证明了3DM有yes answer 是instance of P问题有zero total cost的充分必要条件。3DM是一个strong NP hard问题，所以P问题是一个strong NP hard问题。

**Section 3**: 采用拉格朗日松弛启发算法的目的是解决原问题$P$的对偶问题$P_{LD}$ （企图用对偶问题与原问题的关系来解得原问题的最优解。）$P_{LD}$的最优值通过这个heuristic算法来估计，公式为$Z^* = min(\underline{Z}(\lambda),  2*\overline{Z}(\lambda))$. $\overline{Z}(\lambda)$通过解决问题$P_{ub}$可得，$\underline{Z}(\lambda)$通过解决拉格朗日函数$P(\lambda)$。而$P(\lambda)$为了易于得到它的optimal feasible solution，又拆分为了2个问题$P_{in}(\lambda)$, $P_{out}(\lambda)$, 引入Dummy Time point 重构为$P_{in'}(\lambda)$, $P_{out'}(\lambda)$（附录中证明了P 和P’的optimal value是相同的：证明思路：已知$P$(或者$P’$)的feasible solution, 构造一个$P’$(或者$P$)的feasible solution，分成四个情况分别讨论）。$P_{in}(\lambda)$, $P_{out}(\lambda)$ 是一个二分图问题，可以用匈牙利算法解决。

[paper]:<https://pubsonline.informs.org/doi/abs/10.1287/trsc.2018.0879?journalCode=trsc>

