---
title: Glove和fastText
date: 2018-06-18 13:08:41
tags:
- NLP
- Glove
- fastText
categories: 笔记
---

## 词向量：Glove和fastText
在word2vec被提出后，很多其他词嵌入模型也陆续发表。其中比较有代表性的两个模型是斯坦福团队发表的Glove和Facebook团队发表的fastText

## Glove
Glove使用了词与词之间的共现（co-occurence）信息。我们定义X为共现词频矩阵，其中元素$x_{ij}$为词j出现在词i的背景的次数。这里的“背景”有多种可能的定义。举个例子，在一段文本序列中，如果词j出现在词i左边或右边不超过10个词的距离，我们可以认为词j出现在词i的背景一次。令 $x_i = \sum_{k}x_{ij}$为任意词出现在词i的背景的次数。那么
$$P_{ij} = P(j|i) = x_{ij}/x_{i}$$
为词j在词i的背景中出现的条件概率。这一条件概率也称为词i和词j的共现概率。

### 共现概率的比值
共现概率的比值比较直观地表达词与词之间的关系。Glove试图用有关词向量的函数来表达共现概率的比值。

### 用词向量表达共现概率比值
Glove的核心思想在于使用词向量表达共现概率比值。而任意一个这样的比值需要三个词i,j和k的词向量。对于共现概率$P_{ij} = P(j|i)$，我们称词i和词j分别为中心词和背景词。我们使用$v_i$和$v_i^~$分别表示词i作为中心词和背景词的词向量
我们可以用有关词向量的函数\\(f\\)来表达共现概率比值：
$$f(v_i, v_j, v_k^~) = P_{ik} / P_{jk}$$
需要注意的是，函数\\(f\\)可能的设计并不唯一。下面我们考虑一种较为合理的可能性。首先，用向量之差来表达共现概率的比值，并将上式改为
$$f(v_i - v_j, v_k^~) = P_{ik} / P_{jk}$$
由于共现概率比值是一个标量，我们可以使用向量之间的内积把函数\\(f\\)的自变量进一步改写，得到
$$f((v_i - v_j)^T * v_k^~) = P_{ik} / P_{jk}$$
由于任意一对词共现的对称性，我们希望以下两个性质可以同时被满足：
 - 任意词作为中心词和背景词的词向量应该相等：对任意词i，$v_i = v_i^~$
 - 词与词之间共现词频矩阵X应该对称：对任意词i和j，$x_{ij} = x_{ji}$

为了满足以上两个性质，一方面，我们令
$$f((v_i - v_j)^T * v_k^~) = f(v_i^T*V_k^~) / f(v_j^T*V_k^~)$$
并得到$f(x) = exp(x)$以上两式的右边联立
$$exp(v_i^T* v_k^~) = P_{ik} = x_{ik} / x_{i}$$
有上式可得
$$v_i^T*v_k^~ = log(x_{ik}) - log(x_i)$$
另一方面，我们可以把上式中$log(x_i)$替换成两个偏差项之和$b_i + b_k$，得到
$$v_i^T*v_k^~ = log(x_{ik}) - b_i - b_k$$
将i和k互换，我们可验证一对词共现的两个对称性质可以同时被上式满足。因此，对于任意一对词i和j，我们可以用它们词向量表达共现词频的对数：
$$v_i^T*v_j^~ + b_i + b_j = log(x_{ij})$$

## 损失函数
Glove中的共现词频是直接在训练数据上统计得到的。为了学习词向量和相应的偏差项，我们希望上式中的左边和右边尽可能接近。给定词典V和权重函数$h(x_ij)$，Glove的损失函数为
$$\sum_{i,j}h(x_{i,j})(v_i^T*v_j^~ + b_i + b_j - log(x_{ij}))^2$$
其中权重函数$h(x)$的一个建议选择是，当x<C(例如C = 100)，令$h(x) = (x/c)^a$（例如a = 0.75），反之$h(x) = 1$。由于权重函数的存在，损失函数的计算复杂度与共现词频矩阵X中非零元素的数目呈正相关。我们可以从X中随机采样小批量非零元素，并使用优化算法迭代共现词频相关词的向量和偏差项。

需要注意的是，对于任意一对i,j，损失函数中存在以下两项之和
$$h(x_{i,j})(v_i^T*v_j^~ + b_i + b_j - log(x_{ij}))^2 + h(x_{j,i})(v_j^T*v_i^~ + b_j + b_i - log(x_{ji}))^2$$

由于上式中$x_ij=x_ji$，对调$v$和$v^~$并不改变损失函数中这两项之和。也就是说，在损失函数所有项上对调$v$和$v^~$也不改变整个损失函数的值。因此，任意词的中心词向量和背景词向量是等价的。只是由于初始化值的不同，同一个词最终学习到的两组词向量可能不同。当学习得到所有词向量以后，GloVe使用一个词的中心词向量与背景词向量之和作为该词的最终词向量。
