---
title: 论文笔记DPP_2015
date: 2017-04-07
tags: paper_note
categories: paper_note
---
论文为A Discriminative Predicate Path Mining Approach 
总结自己对这篇文章的理解，可能不太到位，并且有的点自己也存在困惑，由于时间关系等，暂时无法深究。欢迎指教。
<!--more--> 
### A Discriminative Predicate Path Mining Approach 
 
* 解决的问题：  
    fact checking, 判断这样的三元组是否成立，(subject, predicate, object)，其中subject,object 是实体，predicate是一种关系。例如： “Springfield is the capital of Illinois“，可以表示成（Springfield, capitalOf, Illionis).
* 之前的方法以及缺陷： 
    * association rule mining:  
        找全局的规则和同一侧，而不是去生成一种健壮的对于依赖上下文的断言的理解。
    * meta path mining works:   
        人类必须理解这个领域并且在分析开始之前写下相关的meta paths
* 文章的贡献：   
    找出一种方法自动的确定一个路径集合，这个集合可以唯一的概括两个实体之间的关系。
    1. discriminative path mining algorithm，能够发现RDF-style 三元组的定义，例如a statement of fact。
    2. 可解释的 fact checking framework，利用之前的discriminative paths 来预测真假
    3. 将fact checking 建模成一个link prediction的问题 同时在两个大型的知识图谱上验证。
* 问题的定义
    1. knowledge graph
    ![](/images/14912142937359.jpg) 
    节点的标签类型可以不唯一，如果唯一的话则为heterogeneous information network (HIN) 
    2. meta path
    ![](/images/14912143192104.jpg)
    3. Anchored Predicate Path
    ![](/images/14912143584163.jpg)
    4. Discriminative Paths
    ![](/images/14912144282286.jpg)
    
    5. fact checking
    ![](/images/14912146707486.jpg)
    $T^+,T^-$表示正负样本点对
* DISCRIMINATIVE PATH ANALYSIS
    三个阶段1）提取， 2）选择， 3）验证
    1. 路径提取
        ![](/images/14912158017090.jpg)
        <font color=red>有点没理解，用DFS求$P^-$?怎么求出来的？在图中$u,v$可能相互可达但不存在p的关系，但是有可能不可达吧？ </font>  
        定义路径集合：
        ![](/images/14912158155574.jpg)
    2. meta path versus Predicate Path
        meta path 在谓词是没有歧义的情况下是多余的。
        ![](/images/14912162745624.jpg)
在有歧义的情况下利用这个来去除多余的meta path。
    3. 路径的选择
        ![](/images/14912163763921.jpg)
        利用这样来减少特征只保留一些最有判别力的特征，应该也就是不同的路径吧
    4. Fact Interpretation  
        找出路径集合的一个子集，保留一些最有代表性的路径，去除无用的路径。
        ![](/images/14912168882944.jpg)

* 实验
    在实验中使用anchored predicate paths （只有端节点有标签,特征更少？为什么？因为有歧义的情况下，meta path 会更多？例如(lyricist, authorOf, song),(writer, authorOf, book),在meta path 中就会表示成两种，而这个只会有一种）而不用meta paths（所有节点都有标签）
![](/images/14912174451881.jpg)


