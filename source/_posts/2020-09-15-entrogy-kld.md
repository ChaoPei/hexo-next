---
title: 机器学习中熵、交叉熵和 KL 散度的介绍
date: 2020-09-15 10:38:59
update: 2020-09-15 10:38:59
categories: MachineLearning
tags: [熵, KLD, 交叉熵, KL 散度]
mathjax: true
---


我在之前的文章：[机器学习中关于熵的一些概念](https://murphypei.github.io/blog/2019/12/entropy.html) 已经记录了机器学习中一些常见的关于熵的概念，最近看到一篇关于交叉熵和 KL 散度的英文文章，觉得不错，就大概翻译记录一下。

<!-- more -->

熵这一术语起源于统计热力学，是物理学的一个子领域。然而对于机器学习，我们更感兴趣的是信息论或香农熵定义的熵。这种熵的表述与信息的相关概念密切相关。熵是从某个随机过程产生的信息的平均速率。因此，我们首先需要解释“信息”一词在信息理论上下文中的含义。

### 信息和概率

信息论中的信息通常是以比特为单位来衡量的，它可以被宽泛地定义为从一个给定事件中产生的“惊奇”的数量。举一个简单的例子——想象一下，我们有一个极其不公平的硬币，当抛出时，它有 99% 的机会正面着陆，而反面着陆的机会只有 1% 。我们可以使用集合符号 ${0.99, 0.01}$ 来表示。

如果我们掷硬币一次，硬币正面朝上落地，我们不会感到很惊讶，因此这种事件“传递”的信息很低。或者，如果它落在反面，我们会非常惊讶（考虑到我们对硬币的了解），因此这样一个事件的信息会很高。这在数学上可以用以下公式表示:

$$
I(E) = -log[Pr(E)] = -log(P)
$$

这个方程给出了随机事件 E 中包含的信息：由**事件概率的负对数**给出的。这可以更简单地表示为 $-log(p)$。需要注意的一点是，如果我们处理的是以比特表示的信息，即每个比特不是 0 就是 1，对数的底是 2，所以 $I(E) = -log_{2}(p)$。机器学习中经常使用的一个替代单元是 **nats**，它适用于使用自然对数的地方。

对于我们不公平硬币的掷硬币事件，正面所包含的信息为 $-log_{2}(0.99) = 0.0144$ 位，这个值相当低，负面的信息等于 6.64 位。因此，这与上面讨论过的对信息的解释很好地吻合。

现在，回想一下熵被定义为从一个随机过程中产生的平均信息率。抛硬币过程中产生的平均或预期信息率是多少？

### 熵和信息

我们如何计算事物的期望值或平均值？回想一下，变量 $x$ 的期望值是:

$$
E[X] = \sum_{i=1}^{n} x_{i}p_{i}
$$

其中 $x_{i}$ 是 $x$ 的某个可能值，$p_{i}$ 是该可能值出现的概率。**熵是信息的期望**，因此定义为：

$$
H(X) = E[I(X)] = E[-log(P(X))] = -\sum_{i=1}^{n}P(x_{i})logP(x_{i})
$$

同样是上述的抛硬币问题，可以很容易求解抛硬币这件事的熵：

$$
H(X) = -(0.99log(0.99) + 0.01log(0.01)) = 0.08bit
$$

可以说，不公平硬币是一个平均信息传递率为 0.08bit 的随机信息发生器。这是相当小的，因为它被正面结果的高概率所支配。然而，应该指出的是，一枚公平的硬币会产生 1bit 的熵。以上这个例子应该能让你清楚地知道熵是什么，它测量的是什么，以及如何计算它。

### 机器学习和熵

你可能想知道熵在机器学习中的应用，比如交叉熵的核心概念，你们可能已经很熟悉了，我稍后也将详细介绍。然而，熵本身也被用于机器学习。一个值得注意的和有指导意义的例子是它在强化学习的政策梯度优化中的应用。在这种情况下，训练一个神经网络来控制某个 agent，其输出由一个 softmax 层组成。这个 softmax 层是 agent 的 action 的一个概率分布，根据这个概率分布可以选择最佳 action。

比如对于一个 action 空间为 4 的强化网络，其输出结果可能为：${0.9, 0.05, 0.025, 0.025}$。在这个例子中，代理最有可能选择第一个操作（即概率为 0.9）。但熵是怎么产生的呢？强化学习中需要解决的一个关键问题是确保 agent 不会过快地学会在一组行动或策略上收敛，也就是所谓的鼓励性探索。在强化学习的政策梯度版本中，可以通过将输出层熵的负值放入损失函数来鼓励探索。因此，随着损失最小化，行为概率趋于缩小和稳定，以抵消负熵的增加。

> 这一段比较难翻译，大概意思就是如果 softmax 收敛到某个结果（概率很大），则熵比较小（事件很确定，参考上述 0.99 的硬币）。而强化学习中鼓励探索性搜索，所以希望熵大一点，也就是概率分布在每个 action 上都比较平均。所以将熵的负值加入到损失函数中。这样相当于最小化损失函数时，鼓励熵增。

比如上述的例子，如果 softmax 输出是 ${0.9, 0.05, 0.025, 0.025}$，则其在损失函数中的负熵为：-0.61。如果输出是 ${0.3, 0.2, 0.25, 0.25}$，则其在损失函数中的负熵为：-1.98，损失函数的值也因此变小，因此这也是损失函数优化的一种方向趋势。

在机器学习的某些贝叶斯方法中也使用了熵，但这里不讨论熵。现在是考虑常用交叉熵损失函数的时候了。

### 交叉熵和 KL 散度

交叉熵的核心是测量两个概率分布 P 和 Q 之间的“距离”。正如你所观察到的，**熵本身只是一个概率分布的度量**。因此如果我们试图找到一种方法来建模一个真正的概率分布 P，比如说使用一个神经网络产生一个近似的概率分布 Q，那么就需要某种可以最小化的距离或差异度量。交叉熵的差异度量来源于 Kullback-Leibler(KL) 散度。这可以从交叉熵函数的定义中看出:

$$
H(p, q) = H(p) + D_{KL}(p \parallel q)
$$

第一个项，在优化过程中，**真实概率分布 P 的熵是固定的**，因此它在优化过程中是一个附加常数。在优化过程中，只有第二种近似分布的参数 Q 可以改变，因此度量两种分布距离的交叉熵的核心就是 KL 散度函数。

> 这个是转换的核心，特别是对于机器学习来说，训练集的数据和标签集合就是真实概率分布，对于确定的训练集，这个概率分布是确定的，也就是在训练过程中是一个常数。所以多分类训练的时候，交叉熵损失可以用 KL 散度代替。

从信息论的角度来看，两个分布之间的 KL 散度有许多不同的解释。简而言之，这也是一种“惊讶”的表达方式。假设 P 和 Q 很接近，如果事实证明它们并非如此，那就令人惊讶了，因此在这些情况下，KL 散度将会很大。如果它们靠得很近，那么 KL 散度就会很低。

> 惊讶本质就是描述事件包含的信息。散度本质就是距离。

从贝叶斯观点来看，KL 散度的另一种解释也是非常直观的。这种解释认为 KL 分歧是当我们从先验概率分布 Q 到后验概率分布 P 时所获得的信息。KL 散度的表达式也可以用似然比的方法推导出来。

> 关于先验概率和后验概率，以及最大后验概率估计，可以参考[最大似然估计和最大后验概率估计](https://murphypei.github.io/blog/2020/03/mle-map.html)。

#### 贝叶斯方法推导 KL 散度

似然比函数定义：

$$
LR = \frac{p(x)}{q(x)}
$$

这可以解释为：如果一个值 $x$ 是从某个未知的分布中取样，似然比表示样本来自分布 P 的可能性比来自分布 Q 的可能性大多少。如果它更有可能来自 P，LR > 1，否则如果它更有可能来自 Q，LR < 1。

假设我们有很多独立的样本，我们想要在考虑到所有这些证据的情况下，估计出可能性函数，然后变成：

$$
LR = \prod_{i=0}^{n}\frac{p(x_{i})}{q(x_{i})}
$$

如果我们把这个比率转换成对数，就可以把上面定义中的乘积变成一个总和：

$$
LR = \sum_{i=0}^{n}log\left(\frac{p(x_{i})}{q(x_{i})}\right)
$$

现在我们把似然比作为总和。假设我们想要回答这样一个问题: 平均每个样本可能性, $p(x)$ 比 $q(x)$ 大多少？要做到这一点，我们可以采取的期望值的似然比，并得到：

$$
D_{KL}(P\parallel Q) = \sum_{i=0}^{n}p(x_{i})log\left(\frac{p(x_{i})}{q(x_{i})}\right)
$$

上面的表达式就是 KL 散度的定义。它基本上是似然比的期望值。其中似然比表示的是样本数据来自分布 P 而不是分布 Q 的可能性有多大。另一种表示上述定义的方法如下(使用对数规则) :

$$
D_{KL}(P\parallel Q) = \sum_{i=0}^{n}p(x_{i})log (p(x_{i})) – \sum_{i=0}^{n}p(x_{i})log (q(x_{i}))
$$

上述方程中的第一项是分布 P 的熵。正如你所记得的，它是 P 的信息含量的期望值。第二项是 Q 的信息量，但是通过 P 的分布加权的（而不是 Q 的分布）。如果 P 是“真实”分布，那么 KL 散度就是该分布通过 Q 表达（或称为**编码**）时“丢失”的信息量。

不管你怎么解释 KL 散度，它显然是概率分布 P 和 Q 之间的差值。然而它只是一个“准”距离测度，因为 $D_{KL}(P \parallel Q) \neq D_{KL}(Q \parallel P)$。

现在我们需要说明 KL 散度是如何产生交叉熵函数的。

### 交叉熵

如前所述，交叉熵是“真实”分布 P 的熵和 P 与 Q 之间 KL 散度的组合：

$$
H(p, q) = H(p) + D_{KL}(p \parallel q)
$$

利用熵和 KL 散度的定义，以及对数规则，我们可以得到以下交叉熵的定义:

$$
H(p, q) = – \sum_{i=0}^{n}p(x_{i})log (q(x_{i}))
$$

在神经网络的分类任务中使用这个函数看起来像什么？在这样的任务中，我们通常要处理的真实分布 P 是一个 one-hot 编码。例如在 MNIST 手写数字分类任务中，如果图像表示手写数字“2” ，P 就是：${0, 0, 1, 0, 0, 0, 0, 0, 0, 0}$。

我们的神经网络在这样一个任务的输出层将是一个 softmax层，其中所有的输出已被规范化，所以他们的和为 1，代表一个准概率分布。这张图片的输出层 Q 可以是：${0.01, 0.02, 0.75, 0.05, 0.02, 0.1, 0.001, 0.02, 0.009, 0.02}$。

为了得到预测的类，我们将在输出上运行 $argmax$，在这个例子中，我们将得到正确的预测。然而，观察交叉熵损失函数在这种情况下是如何工作的。对于除 $i=2$ 以外的所有值，$p(x_{i})=0$，因此这些索引的总和中的值为 0。唯一没有零值的索引是 $i=2$。因此，对于 one-hot 编码向量，交叉熵折叠为:

$$
H(p,q) = -log(q(x_{i}))
$$

在本例中，交叉熵损失为 $-log(0.75)=0.287$（使用 nats 作为信息单位）。对于 $i=2$ 指数，Q 值越接近 1，损失就越小。这是因为对于这个指标，P 和 Q 之间的 KL 散度减小了。

有人可能会问，如果交叉熵损失的分类任务降低到一个单一的输出节点计算，神经网络如何学习既增加真实索引的值，并减少所有其他节点的值？它通过节点之间通过权重进行交叉相互作用来实现这一点，通过 softmax 激活函数指数本身的性质（各项相加等于 1），如果一个单一的指数被鼓励增加，那么所有其他的指数/输出类别将被鼓励在 softmax 激活函数中减少。