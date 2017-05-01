---
title: 论文笔记Ask Me Anything
date: 2017-04-23
tags: paper_note
categories: paper_note
---
论文为Ask Me Anything: Free-form Visual Question Answering Based on Knowledge from External Sources
这篇文章新的model结合通用知识库来解决更加复杂的VQA的问题。
总结自己对这篇文章的理解以及疑惑的地方，可能不太到位，欢迎指教。
<!--more--> 
### Ask Me Anything: Free-form Visual Question Answering Based on Knowledge from External Sources
* 主要贡献以及解决的问题： 
    * 新的解决VQA的方法，结合了图片问题的信息和通用知识库，来回答更加复杂的VQA问题
* 之前的方法以及缺陷： 
    * 利用CNN、LSTM 分别获取图片和文本的信息，但是更加复杂的问题需要的信息可能没有出现在图片那种，可能是常识或者是特定的只是。
* 大致流程： 
    * 利用图片问题对，来预测图片的一系列属性，同时基于这些属性来生成摘要。
    * 使用一些属性来生成查询，应用在Resource Description Framework 的知识库上。从KB中获取了段落，编码。
    * 编码特征和摘要和知识库的信息输入LSTM训练
* 抽取，编码和合并
    模型图如下：
![](/images/14922458429919.jpg)

    * 基于属性的图片表达：  
        作为一个多标签的分类任务，一个共享的CNN与每个proposal 连接，最后利用max pooling 来做最后的预测。  
        初始化的模型利用VggNet，shared CNN 在multi-label上调整过，最后一层全连接输入softmax求$c$种分类标签，$c$表示的是特征的字典大小。  
        最后使用Multiscale Combinatorial Grouping(MCG) (**Multiscale Combinatorial Grouping for Image Segmentation and Object Proposal ,Generation (待细读原文）**)来进行proposal 的生成，最后应用max-pooling 来将所有输出整合到一个预测向量$V_{att}(I)$中。
    * 基于说明的图片表达：   
        将高层次基于属性的表达喂入LSTM来生成说明，就意味着将上一步中的$V_{att}(I)$作为这一步的输入。  
        利用这样的方法来生成5个不同的说明(**using beam search**)来组成一个内在的文本表达。  
        caption-LSTM的隐层向量在生成完每个说明之后都用来表达这个说明的内容。(**有没有可能生成的某两个说明刚好一样？如何处理?**) 最后使用平均池化将5个隐层向量来获取一个512维的$V_{cap}(I)$  
        caption-LSTM是在MS COCO训练集上人类生成的说明上训练的。
    * Knowledge Base  
        使用DBpedia来作为外部知识来源。  
        利用top-5 的 属性来生成查询。利用KB+SPARQL来进行查询。  
        查询返回的是一段文字，利用Doc2Vec来提取出他的语义信息。  
        训练利用的是DBpedia中的10000个文档来训练一个向量维度为500的模型。  
        在进行Doc2Vec之前将5个返回的段落简单的拼接成一个大的段落，之后在利用Doc2Vec抽取信息。
* 模型：
    通过最大化正确答案的概率来训练
    令：$Q = \{q_1,...,q_n\}$，$Q$表示问题中单词的序列,$n$表示问题的长度
    $A = \{a_1,...,a_l\}$，$A$表示答案中单词的序列,$l$表示答案的长度  
    对数似然可以写成下面的形式：
    $$
    \text{log} p(A|I,Q) = \sum_{t=1}^{l}\text{log} p(a_t|a_{1:t-1},I,Q)
    $$
    其中$p(a_t|a_{1:t-1},I,Q)$表示的是给定图片和问题以及之前的答案单词，生成$a_t$的概率是多少。
    使用LSTM来编码图片和问题信息，同时用LSTM解码生成答案。解码和编码的LSTM共用权值。  
    训练的时候将问题和答案拼接起来就可以了，$\{q_1,...,q_n,a_1,..,a_{l+1}\}$，$a_{l+1}$是END符号。  
    损失函数： 
    ![](/images/14922482094349.jpg)
    其中$p_t(a_t^{(i)}$表示对于第$i$个输入softmax层的激活值，$\theta$表示模型的参数。
* 实验
    数据集特点：
    ![](/images/14922486892154.jpg)
    * COCO-QA中的结果： 
        * 评价方法：
            1. Acc
            2. WuPalmer similarity(WUPS)@0.9
            3. WuPalmer similarity(WUPS)@0.0
        ![](/images/14922574164321.jpg)
    * VQA中的结果： 
        * 评价方法与VQA原论文中一样:
        $$
        min(\frac{\text{ # humans that provided that answer}}{3},1)
        $$ 
        ![](/images/14922576813542.jpg)



