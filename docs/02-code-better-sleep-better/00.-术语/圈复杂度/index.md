# 圈复杂度<no value>

圈复杂度定义为测量“源代码函数中的决策逻辑数量”[NIST235](https://learn.microsoft.com/zh-cn/visualstudio/code-quality/code-metrics-cyclomatic-complexity?view=vs-2022#nist235)。 简而言之，在代码中要做出的决策就多，就越复杂。

每增加一个嵌套或每增加一个分支条件，复杂度+1。

与此行业中的许多度量一样，圈复杂度对所有组织均不存在限制。 但 [NIST235](https://learn.microsoft.com/zh-cn/visualstudio/code-quality/code-metrics-cyclomatic-complexity?view=vs-2022#nist235) 确实表示限制为 10 是理想起点：