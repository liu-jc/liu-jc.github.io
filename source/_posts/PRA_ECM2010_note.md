---
title: 论文笔记PRA_ECM2010
date: 2017-04-05
tags: paper_note
categories: paper_note
---
论文为Relational Retrieval Using a Combination of Path-Constrained Random Walks
总结自己对这篇文章的理解，可能不太到位，欢迎指正。
<!--more--> 
### Relational Retrieval Using a Combination of Path-Constrained Random Walks

* 解决的问题：
    typed proximity queries , 例如ad hoc retrieval（如基于web的搜索引擎），named entity recognition  
* 任务的输入输出：  
    输入： 查询节点集合，以及一个结果类型  
    输出： 跟结果类型一致的节点，按照跟查询的节点邻近度排序。
* 之前的做法以及缺陷： 
    random walk with restart(RWR)，都基于实体关系图，导致可能有一种边标签的出现的上下文被忽略了。例如下图的情况：
  ![](/images/14911178236277.jpg)

* 文章的贡献以及改进的方法： 
    1. 一种新的方法用于邻近度的查询，在一个有标记的图中，对于边来说有不同种标签，个人理解是等于是把不同的path experts 进行一个加权组合，每一个path expert 都是一种特定的标签的路径？
    2. 扩展方法使得支持另外两种experts:  
        query-independent experts：  
        类似于pagerank的方法，与查询无关，可以离线计算出来   
        popular entity experts：  
        允许排名根据那些特别重要的entities进行调整。
* 数据集以及任务定义： 
    1. 四个任务： 
        * venue recommendation：  
            根据一系列异构的节点，例如，title 中的术语，当前年份，以及和这个文章有关的genes 或 protein 来推荐期刊。
        * reference recommendation：  
            推荐需要引用的paper
        * expert finding：  
            推荐特定领域的专家
        * gene recommendation：  
            作者和年份来预测之后的研究兴趣
    2. 数据集
        * yeast
        * fly
    3. 评估指标：  
        MAP(Mean Average Precision)：
        ![](/images/14911210728041.jpg)

* The Path Ranking Algorithm
    定义$R$是一个二元关系，如果$e$和$e'$有这个关系，记为$R(e,e')$，并定义$R(e) \equiv \{e':R(e,e')\}$，$dom(R)$记为R的领域，$range(R)$记为R的范围，定义path：
    ![](/images/14911203280890.jpg)
定义
![](/images/14911207994315.jpg)
评分函数：
![](/images/14911211786020.jpg)
* 参数的估计以及训练： 
    目标函数：
    ![](/images/14911213126311.jpg)
![](/images/14911213257912.jpg)
避免了正负样本不平均的问题

* 结果： 
    path的weight
    ![](/images/14911215257113.jpg)


