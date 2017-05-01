---
title: 论文笔记Making the V in VQA matter
date: 2017-04-21
tags: paper_note
categories: paper_note
---
论文为Making the V in VQA matter
这篇文章平衡了之前的真实场景数据集，提出了一个可解释的模型。
总结自己对这篇文章的理解，可能不太到位，欢迎指教。
<!--more--> 
### Making the V in VQA matter: 
* 主要的贡献以及解决的问题：   
    * 平衡了VQA的数据集（真实场景下），希望消除模型中语言的倾向，同时提高图片的作用。
    * 评估了最好的几个VQA模型在平衡数据集上的表现
    * 提出了一个新的可解释的模型，能够给出反例（同一个问题下不同答案的与原图片相似的图片）
* 任务定义： 
    1. 回答问题： 
        * 输入： 图片，问题
        * 输出： 答案
    2. 解释：
        * 输入： 图片，问题，需要解释的答案
        * 图片： 与原图片相似的，但答案不一样的图片
* 数据集：   
    * 利用原VQA的数据集作为不平衡的数据集
    * 同时人工的改进原VQA的数据集，使得其更加平衡，其方法与Yin and Yang 那篇类似。
* 测试已有的模型以及baselines：
    * 模型： 
        * Deeper LSTM Question + norm Image：与原始论文VQA：visual question answering 中的方法一样
        * Hierarchical Co-attention（HieCoAtt)：对语言的编码采用分层的形式，分别在word-level，phrase-level，entire question-level 上进行编码，最后组合起来。**（待细读原论文）**        * Multimodal Compact Bilinear Pooling（MCB）：采用Multimodal Compact Bilinear Pooling的机制。 **（待细读原论文）**
    * baselines：（都与Yin and Yang 那篇一样，不赘述了）
        * prior
        * language-only
* 对比平衡和不平衡的数据集： 
    ![](/images/14920775194879.jpg)
    会发现确实可以体现语言的偏差，用不平衡的数据训练的效果在平衡数据集中测试能够发现明显不好。
* 反例的解释：   
    提出一个模型，不仅只回答问题，同时还给出一张与原图片相似的，但答案又不一样的图片。举例，给一张图片，问题：这个消防栓什么颜色？答：红色。同时输出一张图片与原图相似，但答案不是红色。
* 模型：   
    在之前构造数据集的时候，是通过给定图片$I$,问题$Q$，人工选择答案不同的图片$I'$来作为补充数据。那么此时就可以利用$I'$来进行有监督的学习，选取与$I$最相近的24个图片的集合$I_{NN}$（利用VGGNet的倒数第二层的激活值，用L2距离计算邻近程度），学习$I'$在其中的位置。$I'$是人工选择的，机器只负责提供$I_{24}这24张图片$
    * 回答的训练：
        * 输入： $I,Q,A$，表示图片，问题，答案
    * 训练的训练
        * 输入： $Q,A$，表示问题，答案。$I'$反例图片，$I_{NN}(I' \in I_{NN}）$
    * 模型的结构：
        * 共享的部分：  
            学习图片和问题的表达，使用CNN embedding 作为一个分支表达image，LSTM embedding 作为另一个分支表达问题，最后融合。（注意这里的图片需要传入25个，$I$和$I_{NN}$
        * 回答部分：   
            与原始的方法一样，接MLP，再用softmax预测，将cross-entropy loss 作为损失函数
        * 解释部分：  
            利用$QI$的embedding，和需要解释的$A$，来训练计算这两个的内积来获取每个图片的一个标量值，这K个标量值通过一个全连接层来获取K个$S(I_i)$，越小表示越不可能是好的反例。  
            利用hinge ranking losses来鼓励$S(I') - S(I_i) > M - \epsilon， I_i \in \{I_1,...,I_k\} \backslash \{I'\}$,等于最小化下面这个表达式：
            $$
            max(0,M-(S(I')-S(I_i)))
            $$
            将两个损失结合起来可以得到：
            $$
            L = -logP(A|I,Q) + \lambda \sum_imax(0,M - (S(I') - S(I_i)))
            $$
            其中，$P(A|I,Q) = \frac{e^{f_A}}{\sum_je^{f_j}}$，$f_A$表示softmax中$A$的那个神经元的输出
* 解释模型的baseline，以及对比：
    * random： 在$I_{NN}$中随机选一个
    * distance： 在$I_{NN}$中选最相似的一个
    * VQA model： 在$I_{NN}$中$P(A|Q,I_i)$最小的，也就是最不可能为A的一张图片  
    选用top 5 来作为评判
    ![](/images/14920799546282.jpg)


