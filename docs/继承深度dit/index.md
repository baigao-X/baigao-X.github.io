# 继承深度(Dit)<no value>

继承深度基于三个基本假设 [CK](https://learn.microsoft.com/zh-cn/visualstudio/code-quality/code-metrics-depth-of-inheritance?view=vs-2022#ck)：
1. 层次结构中的类越深，它可能继承的方法数量就越多，这使得预测其行为变得更加困难。
2. 由于涉及到更多的类和方法，因此更深的树具有更大的设计复杂性。
3. 树中更深的类更有可能重用继承方法。

DIT 的高值意味着错误率也更高，低值表示错误率也更低。 
DIT 的高值表示通过继承重用代码的可能性更大，低值表示通过继承重用代码的可能性更低。 
由于缺乏足够的数据，目前没有公认的 DIT 值标准。 即使是最近进行的研究，也没有找到足够的数据来确定可用作该度量标准数字的有效数字 [Shatnawi](https://learn.microsoft.com/zh-cn/visualstudio/code-quality/code-metrics-depth-of-inheritance?view=vs-2022#shatnawi)。 虽然没有实证为其佐证，但有一些资源表明 DIT 值的上限约为 5 或 6。