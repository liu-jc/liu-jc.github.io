---
title: 论文笔记Yin and Yang
date: 2017-04-19
tags: paper_note
categories: paper_note
---
论文为Yin and Yang: Balancing and Answering Binary Visual Questions
这篇文章平衡了之前的抽象场景数据集，为了消除语言特征上的偏好，从而更注重于学习视觉信号的细微变化，并且仅关注yes/no问题，关注语义上的推理。
总结自己对这篇文章的理解，可能不太到位，欢迎指教。
<!--more--> 
### Yin and Yang: Balancing and Answering Binary Visual Questions
* 主要的贡献以及解决的问题： 
    * 平衡VQA的抽象场景数据集
    * 在抽象场景中，将yes/no的问题转化成一个tuple，并回答
    * 利用抽象的场景能够更好的关注于high-level的语义推理
* 任务的定义：   
    * 输入：一个抽象场景的图片，一个三元组  
    * 输出：回答在这个抽象场景中这个三元组是否存在
* 数据集： 
    * 利用VQA abstract dataset 作为不平衡的数据集
    * 平衡abstract binary VQA dataset：  
        * 通过尽可能少的修改场景来使同一个问题的答案转变到另一个方向，这样来获取互补的场景。
        * 尽可能少的修改是为了让模型更够学习的到视觉信号中微小的区别，更好的回答问题。
        * 注意数据集不是完全平衡的。
* 方法：   
    * 步骤：
        1. 将question 转换成 tuple
        2. 判断这个tuple是否在图片中
    * tuple extraction
        * parsing：  
            standford parser
        * summarizing： 
            1. 去除stop words
            2. 去除 nominal subject 和 passive nominal subject 之前的单词  
            获取问题的summary
        * extracting tuple： 
            将summary，用the Hunpos Part of Speech（POS） tagger
    * 将不同物体与P 和 S对齐  
        * 计算物体和词之间的mutual information 
        * 在测试中，选择和P之间有最大的mutual information的物体，如果超过一个，就随机选一个
    * visual verification  
        集成Q-model和Tuple-modle，使用同样的图片特征，不同的语言特征
        * Q-model：  
            将问题的自然序列喂进LSTM，使用隐层来作为language embedding
        * Tuple-model：  
            使用P，R，S的word2vec embedding的连接来作为language features
        * visual features：  
            总共有1432维的向量，表示每个场景的特征向量
            * P object 和 S object 分别有 563维，编码物体类别，以及绝对地址（用GMMs建模）等
            * 48维的P，S之间的相对地址
            * 258维编码P、S周围的物体  
        最后将语言和视觉的表达用point-wise multiplied 融合起来，在传个一个MLP，最后用softmax进行分类
* 实验： 
    * baselines:
        1. prior： 用最常用的答案直接回答
        2. Blind-Q+Tuple： 忽视视觉信息
        3. SOTA Q+Tuple+H-IMG： 图片的特征使用的是整体的特征。
    * 在不平衡的数据集上测试：
        1. 视觉信息的帮助
        2. 关注特定的区域是重要的
        3. 对于语言有偏好
    * 在平衡的数据集上测试： 
        1. 在平衡的数据集上训练更好
        2. 关注特定的区域更好
        3. 平衡数据集能够消除语言的偏好，并且能够训练更好的理解场景中的细节
        4. 划分一对互补的场景：  
            输入一对互补场景和对应的问题，只有都回答正确，这个测试样例才算正确。
        
         

