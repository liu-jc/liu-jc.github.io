---
title: 论文笔记VIS+LSTM
date: 2017-04-25
tags: paper_note
categories: paper_note
---
论文为Image Question Answering: A Visual Semantic Embedding Model and a New Dataset
这篇文章算是比较早的一篇了吧，创建了COCO-QA这个数据集，问题是自动生成的，提出了一个CNN+LSTM的模型来解决VQA问题。
总结自己对这篇文章的理解以及疑惑的地方，可能不太到位，欢迎指教。
<!--more--> 
### Image Question Answering: A Visual Semantic Embedding Model and a New Dataset
* 主要的贡献以及解决的问题： 
    1. 提出end-to-end model（CNN+LSTM）来解决VQA问题
    2. 自动问题生成算法，将图片描述转换成问题
    3. 利用上面的问题生成算法，整理了一个新的数据集COCO-QA
* 之前的方法以及存在的不足：
    1. DAQUAR数据集发布时中使用的方法：利用semantic parsing 和 image segmentation。为了获得predicates，依赖于好的图像分割算法。同时计算量大，正确率不高。
* 任务形式： 
    1. input： image，question
    2. output： one word answer（将问题视作一个分类问题）
* 模型：
    1. visual embedding 利用vgg最后一个隐层
    2. word embedding ：skip-gram 等
    3. 将图片视作句子的第一个词，同时利用线性变换将维度转换成跟word embedding 匹配的维度
    4. 也可同时将图片视作最后一个词
    5. 可以采用Bi-LSTM
    6. softmax 分类器，生成答案
![](/images/VIS+LSTM_model.jpg)
* QA对的生成算法：  
    利用image descriptions来生成question，以获得更加像人类的问题。  
    利用一系列的策略来生成句子，并且利用 stanford parser 来获得原始描述的句法结构，同时对四类问题有特别的处理，最后为了为了获得更高质量的QA对拒绝某些出现次数过于频繁的答案。
* 实验结果：   
    1. 模型细节：
        1. CNN+LSTM (VIS+LSTM)
        2. CNN + Bi—LSTM，图片特征在开始和结束都给入（VIS+LSTM-2）
    2. Baselines： 
        1. GUESS MODEL：根据问题类型猜测最常见的答案
        2. BOW MODEL： 不使用图片信息。只用BOW vector作为question的特征，利用LR进行分类**(one of the simplest blind model is to sum all the word vectors of the question into a bag-of-word vector? 这里的word vector 是 one-hot ? 还是就是直接词向量全部加起来？)**
        3. LSTM：也是不使用图片
        4. VIS+BOW： vgg的图片特征和BOW的句子特征组合
        5. VIS：对于每种类型的问题，CNN的最后一层可以训练。同时模型只知道问题的类型，不知道问题。
    3. 测试方法： 
        1. ACC
        2. WUPS@0.0
        3. WUPS@0.9
* 局限：
    1. 仅为答案分类器，答案不够复杂，只有一个词
    2. QA对生成的算法实现强烈依赖于问题类型
    3. 只有英文
    4. 可解释性不强

