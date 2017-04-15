---
title: 论文笔记CPRA_ACL2016
date: 2017-04-10
tags: paper_note
categories: paper_note
---
论文为Knowledge Base Completion via Coupled Path Ranking
总结自己对这篇文章的理解，可能不太到位，欢迎指教。
<!--more--> 
### Knowledge Base Completion via Coupled Path Ranking 
* 解决的问题：  
    knowledge Base Completion
* 任务的输入输出：   
    输入： 知识图谱，一个关系，两个实体  
    输出： True or False，判断是否应该有这样的关系
* 之前的方法以及缺陷：  
    * Path ranking algorithm，single-task 的学习，对于每个关系独立的利用自己的训练数据进行训练得到一个分类器，也就是每个关系都有一个分类器。
    * 忽视了特定关系之间的联系，并且对于出现频率比较小的关系没有办法得到足够的训练数据来训练。
* 文章的贡献以及改进的方法： 
    * 将mulit-task learning 运用到PRA上，叫做 coupled PRA(CPRA)
    * 考虑了关系之间的联系，并且使得隐含数据能够相关的关系中共享
    * 相比于PRA有更高的准确率以及更好的模型可解释性
* PRA 
    三个步骤
    1. Feature extraction  
        这里每一种path都被作为一种feature来预测特定的关系。  
        限制路径的最长长度，使用random walks 等方法来寻找路径。 好像很多用小于等于3的路径,大于3计算量可能太大？还可以用DFS或BFS等来枚举这样的路径。
    2. Feature computation  
        给定实体对$(h,t)$，和一个路径$\pi$，利用random walk probability $p(t|h,\pi)$来作为feature value。  
        由于计算量太大，所以也仅使用缺失和出现来表示feature value的，还有用出现的频率的。
    3. Relation-specific classification 
        对于每一个relation都训练一个独立的分类器，来判断两个实体是否应该被这个关系所连接。之前的工作通常使用逻辑回归来作为分类器训练模型。
* CPRA 
    1. 形式化定义问题
        $O = \{(h,r,t)\}$ 表示知识图谱是这样三元组的集合，每个三元组都表示两个实体$h,t \in \epsilon$,关系 $r \in R$。  
        $\overline{R} \subseteq R$，表示需要被预测的关系集合。  
        $\Pi_r$为关系$r$提取出来的路径特征集合                 
        $\tau_r = \{(x_{ir},y_{ir})\}$为关系$r$的训练集  
        $x_{ir}$为一个实体对的特征向量，并且每一个维度表示一个路径$\pi \in \Pi_r$，$y_{ir} = \pm1$为label
    2. Relation Clustering 
        * 开始有$|\overline{R}|$个clusters，每一个包含一个关系$r \in R$
        * 合并相似度最高的两个cluster
            ![](/images/14914647302008.jpg)
            $\Pi_C = \Pi _{C_m} \cup \Pi_{C_n}$  
            当最高的相似度都小于一个阈值的时候停止算法。  
            这样就能够保证在同一个簇里面的关系都共享大量的path
    3. Relation Coupling   
        * 将同一个cluster里面的不同关系的path ranking结合起来，从而同时学习这些关系的分类任务。  
        * 对于一个cluster包含了K个关系，$C = \{r_1,r_2,...,r_K\}$，可以保证$\Pi_C = \Pi_{r1} \cup ... \cup \Pi_{rK}$  
        * 改进将这K个关系的训练实例都是用共享的特征集合，所以所有的训练数据都可以表示在一个相同的空间中
        * 对于与第k个关系有关的训练数据表示成$\tau_k = \{(x_{ik},y_{ik})\}_{i=1}^{N_k}$
        * 目标在于联合学习K个分类器$f_1,...,f_K$使得$f_k(x_{ik}) \approx y_{ik}$
        * 假设每个关系都有$f_k(x) = w_k \cdot x + b_k$,其中$w_k \in R^{1*d},x\in R^{d*1}$。同时假设对于每个$k \in \{1,...,K\}$都有$w_k = w_0 + v_k$和$b_k = b_0$,其中$w_0$用来对这些不同关系间的共同之处建模，$v_k$用来确定每个关系自身独特的地方。如果关系高度相似，则$v_k \approx 0,w_t \approx w_0$。
        * 形式化定义优化目标： 
            ![](/images/14914670660001.jpg)

        * 转换成更统一的形式：
            ![](/images/14914670429447.jpg)
            其中$\widetilde{\lambda} = \frac{\lambda_1}{K}$,loss function 跟之前一样。  
            <font color = red> 这里我感觉符号有点疑惑，我的理解是:
             $\widetilde{x}_{ik} \in R^{d*k}$,对于每一个$R^{d*1}$的向量按列的方向排过去。
            $\widetilde{w} \in R^{k*d}$，对于每个$R^{1*d}$的权值向量按行的方向排下来。
            所以$\widetilde{w} \cdot \widetilde{x} \in R^{d*d}$
            $\widetilde{f}(\widetilde{x}) = tr(\widetilde{w} \cdot \widetilde{x}) + \widetilde{b}$ ，其中$tr()$表示矩阵的迹则为对角线的和。
            </font>
* 实验
    * 知识图谱的构建  
        去除reverse的关系，比如只要$(h,r_1,t)$出现，$(t,r_2,h)$都会出现的这种三元组，就随机丢弃一种关系。  
        然后同时在路径提取的时候还是需要增加这个关系的inverse 版本,应该是指假设丢弃$r_2$,需要添加$(t,r_1^{-1},h)$？   
    * labeled instance generation   
        对于每一个正样本都故意生成4个负样本，并且让他尽可能的难，并确保负样本不会覆盖正样本。


