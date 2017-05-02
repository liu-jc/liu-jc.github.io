---
title: 论文笔记Neural-Image-QA_16
date: 2017-04-29
tags: paper_note
categories: paper_note
---
论文为Ask Your Neurons: A Neural-based Approach to Answering Questions about Images
这篇文章也是一个end-to-end的LSTM+CNN的模型，就是图片特征每次都要参与到预测答案中，同时提出了两种新的测试方法。
总结自己对这篇文章的理解以及疑惑的地方，可能不太到位，欢迎指教。
<!--more--> 
### Ask Your Neurons: A Neural-based Approach to Answering Questions about Images
* 主要的贡献以及解决的问题：
    1. 提出了一个新的模型（CNN+LSTM），end-to-end的预测答案，并得到更好的效果。
    2. 收集额外的数据来研究人类在这个任务上的共识，提出了两种新的测试方法，一个新的baseline
* 模型：
![](/images/neural-image_model.jpg)
将CNN提取出的图片特征和问题序列中的每个词一起输入到LSTM中，只对预测的词进行loss 的计算。
* 实验： 
    * 数据集： DAQUAR
    * 测试方法： WUPS
    ![](/images/neural-image_WUPS.jpg)
    * 对数据集的扩展，人工收集了一些额外的答案构成了DAQUAR-Consensus
    * Consensus Measures：
        1. Average Consensus
            ![](/images/neural-image_avg_consensus.jpg)
        2. Min Consensus
            ![](/images/neural-image_min_consensus.jpg)



