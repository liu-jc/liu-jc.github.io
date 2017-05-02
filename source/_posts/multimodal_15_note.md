---
title: 论文笔记multimodal_15
date: 2017-04-27
tags: paper_note
categories: paper_note
---
论文为Are you Talking to a Machine? Dataset and Methods for Multilingual Image Question Answering
这篇文章创建了FM-IQA这个数据集，也是一个LSTM+CNN的模型，个人觉得有意思的地方在于他的那个transposed weight share。
总结自己对这篇文章的理解以及疑惑的地方，可能不太到位，欢迎指教。
<!--more--> 
### Are you Talking to a Machine? Dataset and Methods for Multilingual Image Question Answering
1. 主要的贡献以及解决的问题：
    * 构建了一个多语言的数据集（FM-IQA)
    * 构建了mQA 模型来处理VQA的任务
2. 之前的相关工作以及缺陷：
    * 之前有的数据集太小，同时问题是预先定义好的并且很受限。
    * 之前相关的工作模型太小，对于LSTM使用一个单独的，来连接问题和答案，在这篇中考虑到问题和答案的不同之处，分别训练一个LSTM。
    * 语言通常只有一种。
    * 评价标准为自动化评价，这篇中采用人工评价
3. 模型：
    ![](/images/multimodal_15_model.jpg)

    * 模型的四个部分：
        1. LSTM 抽取问题的语义表达（记做LSTM(Q)）, 最后一个时刻中的memory cells 被视作是问题整个问题句子的表达
        2. CNN 抽取图像表达（使用了GoogleNet）
        3. LSTM 抽取当前答案中的词的表达，以及语言上的上下文（记做LSTM(A)）
        4. 一个融合的模块来将之前三个部分的表达以及答案中词的word embedding整合起来，同时生成答案中的下一个单词。
    * 细节以及权值共享：
        * 对于word embedding，fusing，intermediate layer都使用scaled hyperbolic tangent function:
        $$
        g(x) = 1.7159 \cdot tanh(\frac{2}{3}x)
        $$
        * 两个LSTM上面利用的word-embedding layer的权值是共享的
        * word-embedding layers 和 全连接的softmax layer的权值是以一种转置的方式进行共享的，直观上来说因为word-embedding是把one-hot编码到一个稠密的词表达，而softmax 是把一个稠密的词表达解码到一个伪one-hot上descriptions of images.**（transposed weight share, 待读原文, Learning like a child: Fast novel visual concept learning from sentence)**
    * 实验： 
        * 评价方法：
            * 之前评价方法的缺陷：
                1. WUPS：自动测量单个词的相似性(**Verbs semantics and lexical selection**)
                2. BLEU，METEOR： 对于答案中通常只有几个词最关键，但是这两种方法的词的权重一样。（**An automatic metric for mt evaluation with high levels of correlation with human judgements / Cider: Consensus-based image description evaluation**）
                3. CIDEr：权重基于tf-idf frequency term，不能完全展示关键词的重要性(**Deep captioning with multimodal recurrent neural networks (m-rnn)**)
            * Visual Turing Test：让人判断是机器还是人类回答的
            * Human Rated Scores：让人用0，1，2打分，越高越正确
        * 模型的变种：
            1. mQA-avg-question：用词的average embedding来代替LSTM
            2. mQA-same-LSTMs: 两个LSTM 共享权值
            3. mQA-noTWS：不适用transposed weight sharing
    * 讨论：
        * 修改第一部分的LSTM可以生成free-style的关于图像的问题，同时给出答案
        * 当背景场景的常识推理不对的时候，模型会出错
        * 当目标物体太小的时候容易出错
    * future work：  
        通过结合更多的视觉信息和语言信息来解决上述问题，例如使用目标检测或者attention model


