---
title: cs224d_note1
date: 2016-12-21 10:58:38
tags: ml_dl 
categories: ml_dl
---

对于lecture note1的总结与记录。有不对的地方还请大家指点：）
这个笔记主要包括：  
1. NLP的概念以及NLP面对的问题  
2. word vector的概念  
3. 生成word vector 流行的方法   

<!--more--> 
## Introduction to NLP    
NLP的目标：使计算机理解自然语言从而能够完成其他的任务。  
几乎所有NLP任务的最重要的共同的事情是如何将word表达成输入的形式给我们的模型  
我们需要直观的表示来知晓词之间的相似性以及差异。  
运用wordvector，可以很容易的将这些性质编码到向量本身中去。  

## Word vectors  
假设词之间是有某些联系的。所以将词编码成vector表示"词空间"中的点。  
最简单的是使用one-hot vector
每个词是一个<a href="http://www.codecogs.com/eqnedit.php?latex=R^{|v|&space;*&space;1}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?R^{|v|&space;*&space;1}" title="R^{|v| * 1}" /></a>的向量，只有一个地方是1，表示这个单词的索引。    
这样我们就讲每个词表示成为了完全独立的了。这不能给我们带来任何相似性的信息。因为可以发现任何两个wordvector做内积都是0。  
试着降低空间的维度，尝试找到一个子空间能够编码词之间的关系。  

## SVD Based Methods
用此方法找word vector  
对于整个dataset，在某种形式的矩阵X中累计词共同出现的次数，然后对X应用SVD分解得到<a href="http://www.codecogs.com/eqnedit.php?latex=USV^{T}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?USV^{T}" title="USV^{T}" /></a>,然后使用U的行作为我们词典中词的word embeding。  
接下来讨论X的几种选择  
1. **词-文档矩阵**  
假设在同一文档中出现的词是有关的。  
使用如下方法构造X: 遍历所有文档中的所有词，对于word i 出现在 document j 中，就在<a href="http://www.codecogs.com/eqnedit.php?latex=X_{ij}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?X_{ij}" title="X_{ij}" /></a>上+1，所以<a href="http://www.codecogs.com/eqnedit.php?latex=X\in{R^{|V|&space;*&space;M}}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?X\in{R^{|V|&space;*&space;M}}" title="X\in{R^{|V| * M}}" /></a>。 M 为documents 的个数。  
2. **window based Co-occurrenceMatrix**  
X 保存词共同出现的次数，对于每个词计算在特定大小的window中的周边词出现的次数。  
假设语料库就3句话，window size 为1：  
![Co-occur_mat](/images/Co-occur_mat.png)  
使用Word-Word Co-occurrence Matrix:  
1. 生成<a href="http://www.codecogs.com/eqnedit.php?latex=R^{|V|&space;*&space;|V|}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?R^{|V|&space;*&space;|V|}" title="R^{|V| * |V|}" /></a>的矩阵X  
2. 对X运用SVD分解 ，<a href="http://www.codecogs.com/eqnedit.php?latex=X&space;=&space;USV^{T}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?X&space;=&space;USV^{T}" title="X = USV^{T}" /></a>  
3. 对于U选择前k列，得到k维的word vector  
4. <a href="http://www.codecogs.com/eqnedit.php?latex=\frac{\sum_{i=1}^{k}\sigma_i}{\sum_{i=1}^{|V|}\sigma_i}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\frac{\sum_{i=1}^{k}\sigma_i}{\sum_{i=1}^{|V|}\sigma_i}" title="\frac{\sum_{i=1}^{k}\sigma_i}{\sum_{i=1}^{|V|}\sigma_i}" /></a> 表明前k维获得的方差的数量。  
![svd](/images/SVD.png)  
存在一些问题包括矩阵维度变化频繁，时间开销大等，也有一些解决办法（具体可以查看原note，这里不再赘述）

使用iteration based methods 能够解决大部分这些问题。  

## Iteration Based Methods  
不计算以及存储全局的信息，试着建立一个模型一次学一个迭代，最终对于给定上下文的word编码出概率。  
**1. 语言模型**  
对于一个token序列分配一个概率，如果是一个完全合法的句子，应该给出一个高的概率，否则应该给低的。  
数学表达：  <a href="http://www.codecogs.com/eqnedit.php?latex=P(w_{1},w_{2},...,w_{n})" target="_blank"><img src="http://latex.codecogs.com/gif.latex?P(w_{1},w_{2},...,w_{n})" title="P(w_{1},w_{2},...,w_{n})" /></a>  
假设完全独立的，即为一元语言模型(unigram model)：  
<a href="http://www.codecogs.com/eqnedit.php?latex=P(w_{1},w_{2},...,w_{n})&space;=&space;\prod_{i=1}^{n}P(w_{i})" target="_blank"><img src="http://latex.codecogs.com/gif.latex?P(w_{1},w_{2},...,w_{n})&space;=&space;\prod_{i=1}^{n}P(w_{i})" title="P(w_{1},w_{2},...,w_{n}) = \prod_{i=1}^{n}P(w_{i})" /></a>  
接下来令序列的概率取决于成对的概率，二元语言模型(bigram model)：  
<a href="http://www.codecogs.com/eqnedit.php?latex=P(w_{1},w_{2},...,w_{n})&space;=&space;\prod_{i=2}^{n}P(w_{i}|w_{i-1})" target="_blank"><img src="http://latex.codecogs.com/gif.latex?P(w_{1},w_{2},...,w_{n})&space;=&space;\prod_{i=2}^{n}P(w_{i}|w_{i-1})" title="P(w_{1},w_{2},...,w_{n}) = \prod_{i=2}^{n}P(w_{i}|w_{i-1})" /></a>  
**2. Continuous Bag of Words Model(CBOW)**  
利用周边的词对中间的词进行预测或者生成  
定义我们的输入输出，以及目标输出：  
<a href="http://www.codecogs.com/eqnedit.php?latex=x^{(c)}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?x^{(c)}" title="x^{(c)}" /></a>为input one hot vector 或者上下文，output 表示为<a href="http://www.codecogs.com/eqnedit.php?latex=y^{(c)}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?y^{(c)}" title="y^{(c)}" /></a>，目标输出为<a href="http://www.codecogs.com/eqnedit.php?latex=y" target="_blank"><img src="http://latex.codecogs.com/gif.latex?y" title="y" /></a>是已知的中间的词的one hot vector.  
<a href="http://www.codecogs.com/eqnedit.php?latex=V\in{R^{n*|V|}}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?V\in{R^{n*|V|}}" title="V\in{R^{n*|V|}}" /></a> :输入的词矩阵  
<a href="http://www.codecogs.com/eqnedit.php?latex=v_{i}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?v_{i}" title="v_{i}" /></a>：V的一列，代表一个word i的输入向量表达  
<a href="http://www.codecogs.com/eqnedit.php?latex=U\in{R^{|V|&space;*&space;n}}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?U\in{R^{|V|&space;*&space;n}}" title="U\in{R^{|V| * n}}" /></a>：输出的词矩阵  
<a href="http://www.codecogs.com/eqnedit.php?latex=u_i" target="_blank"><img src="http://latex.codecogs.com/gif.latex?u_i" title="u_i" /></a>:U的一行，代表一个word i的输出向量表达  
**CBOW步骤:**   
1. 获得onehot word vectors <a href="http://www.codecogs.com/eqnedit.php?latex=(x^{c-m},...,x^{c-1},x^{c&plus;1},...,x^{c&plus;m}))" target="_blank"><img src="http://latex.codecogs.com/gif.latex?(x^{c-m},...,x^{c-1},x^{c&plus;1},...,x^{c&plus;m}))" title="(x^{c-m},...,x^{c-1},x^{c+1},...,x^{c+m}))" /></a>  
2. 获得embedded word vectors对于上下文<a href="http://www.codecogs.com/eqnedit.php?latex=(v_{c-m}&space;=&space;Vx^{c-m},...,v_{c&plus;m}&space;=&space;Vx^{c&plus;m})" target="_blank"><img src="http://latex.codecogs.com/gif.latex?(v_{c-m}&space;=&space;Vx^{c-m},...,v_{c&plus;m}&space;=&space;Vx^{c&plus;m})" title="(v_{c-m} = Vx^{c-m},...,v_{c+m} = Vx^{c+m})" /></a>  
3. 平均这些向量 <a href="http://www.codecogs.com/eqnedit.php?latex=\hat{v}&space;=&space;\frac{v_{c-m}&plus;...&plus;v_{c&plus;m}}{2m}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\hat{v}&space;=&space;\frac{v_{c-m}&plus;...&plus;v_{c&plus;m}}{2m}" title="\hat{v} = \frac{v_{c-m}+...+v_{c+m}}{2m}" /></a>  
4. 生成score vector <a href="http://www.codecogs.com/eqnedit.php?latex=z&space;=&space;U\hat{v}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?z&space;=&space;U\hat{v}" title="z = U\hat{v}" /></a>  
5. 将scores 变成概率<a href="http://www.codecogs.com/eqnedit.php?latex=\hat{y}&space;=&space;softmax(z)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\hat{y}&space;=&space;softmax(z)" title="\hat{y} = softmax(z)" /></a>  
6. 期待<a href="http://www.codecogs.com/eqnedit.php?latex=\hat{y}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\hat{y}" title="\hat{y}" /></a>与y相同  

定义loss function(使用交叉熵):  
![loss1](/images/loss.png)  
因为y是一个one hot vector，所以化简为：  
![loss2](/images/loss2.png)  
形式化我们的优化目标：  
![fun1](/images/CBOW_func.png)  
然后就可以使用梯度下降法更新所有相关的<a href="http://www.codecogs.com/eqnedit.php?latex=u_c" target="_blank"><img src="http://latex.codecogs.com/gif.latex?u_c" title="u_c" /></a> 和<a href="http://www.codecogs.com/eqnedit.php?latex=v_j" target="_blank"><img src="http://latex.codecogs.com/gif.latex?v_j" title="v_j" /></a>了  
**3. Skip-Gram Model**  
运用中间词来预测上下文  
定义与之前的CBOW基本相同，将x,y调换即可。  
**步骤：**  
1. 获得onehot vector<a href="http://www.codecogs.com/eqnedit.php?latex=x\in{R^{|V|&space;*&space;1}}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?x\in{R^{|V|&space;*&space;1}}" title="x\in{R^{|V| * 1}}" /></a>  
2. 获得上下文的embedded word vectors <a href="http://www.codecogs.com/eqnedit.php?latex=v_c&space;=&space;Vx" target="_blank"><img src="http://latex.codecogs.com/gif.latex?v_c&space;=&space;Vx" title="v_c = Vx" /></a>  
3. 不需要平均，<a href="http://www.codecogs.com/eqnedit.php?latex=\hat{v}&space;=&space;v_c" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\hat{v}&space;=&space;v_c" title="\hat{v} = v_c" /></a>  
4. 生成2m个score vectors，<a href="http://www.codecogs.com/eqnedit.php?latex=u_{c-m},...,u_{c&plus;m}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?u_{c-m},...,u_{c&plus;m}" title="u_{c-m},...,u_{c+m}" /></a> （不包括<a href="http://www.codecogs.com/eqnedit.php?latex=u_c" target="_blank"><img src="http://latex.codecogs.com/gif.latex?u_c" title="u_c" /></a>)  
5. 将scores 利用softmax转化成概率 <a href="http://www.codecogs.com/eqnedit.php?latex=y&space;=&space;softmax(u)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?y&space;=&space;softmax(u)" title="y = softmax(u)" /></a>  
6. 期望这个生成的概率向量能够跟标准输出的onehotvector 相同  
同样采用交叉熵，然后定义优化目标，并利用贝叶斯假设，假设输出的词之间都是完全独立的：  
![](/images/J_skip.png)  
对于这个目标函数，同样可以利用SGD的方法来更新未知参数。  
**4. Negative Sampling**  
对于之前的目标函数，每一次更新都需要O(|V|) 的时间复杂度，一个简单的想法就是估计他，而不是精确的求解。也就是利用已知的概率密度函数来估计未知的概率密度函数。  
对于每个training step，不遍历整个词典，而是单纯的抽样出几个负样本。从一个对应着词频的噪声分布中<a href="http://www.codecogs.com/eqnedit.php?latex=(P_n(w))" target="_blank"><img src="http://latex.codecogs.com/gif.latex?(P_n(w))" title="(P_n(w))" /></a>中采样。  
将negative sampling应用在skip-gram model上，实际上就是优化一个不同的目标函数。  
定义<a href="http://www.codecogs.com/eqnedit.php?latex=P(D&space;=&space;1|w,c)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?P(D&space;=&space;1|w,c)" title="P(D = 1|w,c)" /></a>为(w,c)来自语料数据的概率，<a href="http://www.codecogs.com/eqnedit.php?latex=P(D&space;=&space;0|w,c)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?P(D&space;=&space;0|w,c)" title="P(D = 0|w,c)" /></a>为不来自语料数据的概率。  
定义：  
<a href="http://www.codecogs.com/eqnedit.php?latex=P(D&space;=&space;1|w,c,\theta)&space;=&space;\frac{1}{1&space;&plus;&space;exp(-v_c^{T}v_w)}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?P(D&space;=&space;1|w,c,\theta)&space;=&space;\frac{1}{1&space;&plus;&space;exp(-v_c^{T}v_w)}" title="P(D = 1|w,c,\theta) = \frac{1}{1 + exp(-v_c^{T}v_w)}" /></a>  
目标在于最大化概率使其符合真实情况。并利用极大似然估计去估计<a href="http://www.codecogs.com/eqnedit.php?latex=\theta" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\theta" title="\theta" /></a>：  
![theta](/images/theta.png)  
<a href="http://www.codecogs.com/eqnedit.php?latex=\hat{D}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\hat{D}" title="\hat{D}" /></a>表示为错误的语料  
可以随机的从word bank 中抽样来构成这个。  
定义新的目标函数：  
![J](/images/J_nega.png)  
在上式中<a href="http://www.codecogs.com/eqnedit.php?latex=\{\hat{u_k}|k=1...K\}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\{\hat{u_k}|k=1...K\}" title="\{\hat{u_k}|k=1...K\}" /></a>是从<a href="http://www.codecogs.com/eqnedit.php?latex=P_u(w)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?P_u(w)" title="P_u(w)" /></a>采样得来的。

