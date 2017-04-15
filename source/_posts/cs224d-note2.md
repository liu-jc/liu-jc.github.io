---
title: cs224d_note2
date: 2016-12-21 11:24:07
tags: ml_dl
categories: ml_dl
---

作为总结与提炼，写的不好的地方还请大家指点：）
原版讲义地址：[http://cs224d.stanford.edu/lecture_notes/notes2.pdf](http://cs224d.stanford.edu/lecture_notes/notes2.pdf)
keyphrases：
* 固有的以及外在的评估
* 超参数对于类比评估任务的影响
* 人类判断与词向量距离的相关性
* 利用上下文处理歧义
* window classification  

<!--more-->
### 词向量的评估  
利用word2vec 和GloVe等方法来训练并且发现自然语言词汇在语义空间上的潜在向量表达。考虑如何量化的评估词向量的质量  

1. 内在的评估
对于生成的词向量，在特定的子任务上进行评估，子任务要求简单并且能够快速的计算，使得我们能够直观的得知生成的词向量在子任务上的效果。由于整个系统往往是有个深度神经网络的，训练时间太长。所以使用这样内在评估的方法来度量这个word2vec 系统的好坏。由于这样就需要这个评估对最后的整个任务有着正相关性。
    特点：
    * 在特定的中间任务上评估
    * 快速计算效果
    * 帮助理解子系统
    * 需要有与实际任务有正相关来确定是否有用  

2. 外部的评估
与之前相反，是在实际任务上进行的评估。
    特点：
    * 在实际任务上进行评估
    * 计算慢
    * 出现问题的时候不能确定是这个子系统还是其他子系统，或者是内在的关系导致的
    * 如果替换子系统提高了整个任务的效果，那么这个改变应该是好的  

3. 内在评估的例子：词向量类比
对于词向量的内在评估的选择有一种做法是看在完成词向量类比任务上的表现。在一个词向量类比中，给出如下形式的未完成的类比：$a : b :: c : ?$
这个词向量确定了能够最大化这个cosine像吸毒的词向量：
$$
d = \underset{i}{argmax}\frac{(x_b-x_a+x_c)^Tx_i}{\left \| x_b-x_a+x_c \right \|}
$$
这个度量方法很容易理解。在理性情况下，我们希望$x_b-x_a=x_d-x_c$（例如，queen - king = actress - actor）。希望$x_b-x_a+x_c=x_d$ ，就可以利用这个最大化两个词向量之间的归一化点乘来确定向量$x_d$
词向量类比主要有两种形式，一种是语义上的类比，一种是句法上的类比。   

4. 内在评估调整举例：类比评估
考虑在word embedding 中使用的一些超参数能够通过内在评估系统进行调整。
在内在评估任务中可以考虑为word embedding 调整的参数：
    * 词向量的维度
    * 语料库大小
    * 语料库的来源或者类型
    * 上下文窗口大小
    * 上下文的对称性  
运用不同的word embedding方法在同相同的参数的一个类比评估任务中，观察得到：
    * 性能严重取决于使用的word embedding 的模型
    * 更大的语料库可以使得性能提升
    * 在极度低或极度高的词向量维度下，性能都会更低。（低维度的话模型复杂度太低，high bias ，高维度的话可能获取过多噪声，high variance）  

5. 内在评估举例：相关性评估
评估词向量的质量还可以通过询问人来进行相似性的评估打分，然后再利用这个跟cosine相似度比较。 

6. 处理词的歧义
同一个词可能有多个不同的词向量来处理歧义的问题，例如run 是 动词也是名词，如何使用和解释是基于上下文的。
Improving Word Representations Via Global Context And Multiple Word Prototypes (Huang et al, 2012) 在这篇详细的说明了（还没看orz，抽时间看一下）
主要方法步骤： 
    * 在这个词汇出现的所有地方获取固定大小的上下文窗口
    * 每一个上下文都被表示成一个上下文词向量的权重平均
    * 应用k-means 将这些上下文表达聚类
    * 最后每个出现的地方都被重新标记成其相关的类，并用于训练对应类的次表达  

### 外部任务的训练
讨论如何处理外部任务  

1. 问题的指定
大多NLP的外部任务都可以被看做分类的任务，例如情感分类，以及命名实体识别问题（NER）上 。
在这样的任务中，训练集可以表示成如下形式： 
$$
\{x^{(i)},y^{(i)}\} ^N_1
$$
$x^{(i)}$是一个d维的词向量，$y^{(i)}$是一个C维的onehot 向量，表示榆次的分类。 

2. 词向量再训练
在nlp应用中，在对外部任务进行训练的时候可以对输入的词向量进行再训练。
利用外部任务对预训练过的词向量进行再训练，可以使任务表现的更好。但只有在大的训练数据集上才考虑使用再训练，在小的数据集上，只会变得更糟。  

3. softmax分类以及正则化
 softmax分类通俗来讲就是计算每一类的概率，从而选取最大的那一类。利用交叉熵来定义损失函数。这部分不了解的可以查看cs229的讲义。公式部分这里不再赘述了。
 考虑在同时更新模型权值以及词向量，需要的参数数量。在简单的线性分类中，更新权值需要$C*d$个参数，如果更新词典中每个词的词向量，需要$|V|*d$。 

4. Window classification 
我们之前是使用单个词向量x作为输入，但自然语言中，一个词的意思在不同的上下文中有着不同的意思。所以考虑将一个序列的词语作为输入。
窄的窗口大小对句法有更好的表现，宽的则对语义有着更好的表现。
基于这个想法，就将$x^{(i)}_{windows}$ 代替 $x^{(i)}$
![](/images/windows.png)
词向量的梯度则变为：
![](/images/gadient.png)  

5. 非线性分类器
线性分类器通常过于简单了，需要像神经网络这样的非线性分类器。note3中会介绍。



