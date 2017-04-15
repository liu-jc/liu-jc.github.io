---
title: path ranking algorithm
date: 2017-04-05
tags: paper_note
categories: paper_note
---
对于path ranking algorithm进行简要的介绍，主要用在知识图谱中，做link prediction，以及检索相关实体的任务。  
原始论文为 Relational Retrieval Using a Combination of Path-Constrained Random Walks

<!--more--> 
### path ranking algorithm
* 定义：  
    * $R$为一种二元关系。如果$e,e'$之间存在$R$这样的关系，则表示为$R(e,e')$,同时$R(e) \equiv \{e':R(e,e')\}$
    * $dom(R)$表示R的domain, $range(R)$表示R的range,并且定义一条关系路径$P$的domain就是路径中第一个关系的domain，而这条路径的range就是最后一个关系的range。
    * 对于关系路径$P = R_1...R_l$,以及查询的实体集$E_q \subset dom(P)$，定义一个概率分布$h_{E_q,P}$，表示在当前路径是$P$的情况下，下一个节点是实体$e$的概率。  
      ![](images/14915566131080.jpg)
      这里解释一下：
      1. 如果$P$是一条空的路径的话，那么下一个节点为$e$的机会自然是均匀的。
      2. 如果$P = R_1...R_l$非空，那么下一个节点概率可以从去除$P$中末尾节点的情况转移过来，类似一个DP的思想。假设$P$中最后一个节点就是$e'$，那么如果存在这样的关系$R_l(e',e)$，则有概率转移到$e$这个实体上。
    * 对于每一种path都有一个参数$\theta$，同时限定路径的长度，很容易生成路径集合$P(q,l) = \{P\}$，通过以下的评分函数：
        ![](images/14915586880722.jpg)
      可以表示成矩阵的形式： 
      ![](images/14915590764786.jpg)
* 查找相关实体的任务，训练步骤，参数估计： 
    * 训练数据集$D = \{(q^{(m)},r^{(m)}\}$，其中$r^{(m)}$表示一个二值，如果$r_e^{(m)}=1$表示实体$e$与查询$q^{(m)}$有关，0则无关
    * 最大化目标函数并且更新参数： 
     ![](images/14915602350793.jpg)
    ![](images/14915604849035.jpg)
    此处解释一下：  
        * $A^{(m)}$为这个样本的特征矩阵，而每一行每一个元素都对应着一个实体$e$所有的$h_{E_q,P(e)}$
        * $N^{(m)}$为不相关的实体的下标集合
        * $R^{(m)}$为相关的实体的下标集合
    * 负样本的采集： 
        使用stratified random sampling
    * 测试： 
        * 评价标准选用MAP进行评价
        ![](images/14915618243282.jpg)
        * 在venue和gene推荐任务重只有少量的候选答案，所以相对简单

* 对于特定的关系进行link prediction，训练过程，参数估计： 
    * 对于特定的关系$r$生成正负节点对$\{(s_i,t_i)\}$
    * 训练数据集$D = \{(x_i,y_i\}$，$x_i$为这个实体对的特征向量，每个元素表示一个路径的特征，$y_i = \pm1$，表示标签，是否存在这样的关系
    * 分类器$f(x) = w \cdot x + b$
    * 利用logistic regression：   
        $l(x_i,y_i) = log(1+exp(-y_if(x_i)))$
    * 目标函数：  
        $min \sum_{i=1}^{N_k}l(x_i,y_i) + \lambda \left \lVert w \right \rVert_2^2$
    * 求导更新参数
    * 测试直接使用准确率进行测试


