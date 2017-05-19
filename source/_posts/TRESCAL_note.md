---
title: 论文笔记TRESCAL
date: 2017-05-19
tags: paper_note
categories: paper_note
---
论文为Typed Tensor Decomposition of Knowledge Bases for Relation Extraction 
这篇文章运用了张量分解的方法来获取knowledge bases embedding，同时考虑了entity的类型，并在关系抽取的任务中验证了该模型。
总结自己对这篇文章的理解以及详细的公式推导，可能不太到位，欢迎指教。
<!--more--> 
## Typed Tensor Decomposition of Knowledge Bases for Relation Extraction 
### 主要贡献 
1. 利用张量分解，在RESCAL 的基础上，提出了一个创建新的KB embedding 的模型 TRESCAL。利用了实体的类型信息，降低了时间和空间的复杂度。
2. 在关系抽取的任务上，提出的模型相比于之前的模型有着显著的提高。

### 现有的张量分解的模型
1. CP decompositions，通过一些rank为1的张量的和来估计一个张量。
2. Tucker decomposition，高维度的SVD分解，将张量分解成一个核心的张量乘以每个维度的矩阵。

### 模型
1. 在张量中表示二元关系：  
    假设有$n$种实体，$m$种关系的类型。  
    $\Gamma = {(e_i,r_k,e_j)}$,$(e_i,r_k,e_j)$三元组表示这存在着这样的二元关系，第$i$个实体和第$j$个实体，存在着第$k$种的关系。  
    这些三元组被编码到一个张量中$X \in \{0,1\}^{n \times n \times m}$, $X_{i,j,k} = 1$当且仅当$(e_i,r_k,e_j) \in \Gamma$  
    同时利用$\chi_k$表示张量的第$k$个切片，如下图所示：
    ![](/images/tensor.jpg)
2. RESCAL
    * 一种张量分解的方法，为了确定张量中的潜在成分。  
        给定张量$X_{n\times n\times m}$，希望得到以下估计：
        $$
        X_k \approx A R_kA^T
        $$
        其中，$A \in R^{n \times r}$,第$i$行表示第$i$个实体的$r$个潜在成分。  
        $R_k \in R^{r\times r}$是一个非对称矩阵，表示关于关系$k$的潜在成分之间的影响。每一种关系$R_k$都不一样，但是$A$始终是一样的。  
    * 定义损失函数： 
        $$
        \begin{align*}
        J =& min\ \frac{1}{2}(\sum_k \left\lVert X_k - AR_kA^T \right\rVert_F^2) + \frac{1}{2}\lambda(\left\lVert A \right\rVert_F^2 + \sum_k\left\lVert R_k \right\rVert_F^2) \\
        =& min\ \frac{1}{2}(\sum_k tr((X_k - AR_kA^T)^T(X_k - AR_kA^T)) + \frac{1}{2}\lambda (tr(A^TA) + \sum_k tr(R_k^TR_k)) \\
        =& min\  \frac{1}{2}(X_k^TX_k - X_k^TAR_kA^T - AR_k^TA^TX_k + AR^T_kA^TAR_kA^T) + \frac{1}{2}\lambda(tr(A^TA) + \sum_k tr(R_k^TR_k))
        \end{align*}
        $$
        其中$tr$表示矩阵的迹。
    * 利用交替最小二乘解决优化问题：
        * 固定$R_k$对$A$进行优化： 
            $$
            \begin{align*}
            \frac{\partial J}{\partial A} &= \frac{1}{2} \sum_k(-X_k^TAR_k-X_kAR_k^T + AR_kA^TAR_k^T + AR_k^TA^TAR_k) + \lambda A
            \end{align*}
            $$
            令$\frac{\partial J}{\partial A} = 0$：
            $$
            \begin{align*}A\sum_k(R_kA^TAR_k^T+R_k^TA^TAR_k) + \lambda \mathbf{I}) = \sum_k(X_k^TAR_k + X_kAR_k^T)\\
            A = (\sum_k(X_k^TAR_k + X_kAR_k^T))(\sum_k(R_kA^TAR_k^T+R_k^TA^TAR_k) + \lambda \mathbf{I}))^{-1} \\
            \end{align*}
            $$
            以此更新$A$
        * 固定$A$对$R_k$进行优化： 
            $$
            \begin{align*}
            \frac{\partial J}{\partial R_k} &= \frac{1}{2}(-A^TX_kA - A^TX_kA + A^TAR_kA^TA) + \frac{1}{2}\lambda R_k \\
            &= -A^TX_kA + A^TAR_kA^TA + \lambda R_k
            \end{align*}
            $$        
            令$ \frac{\partial{J}}{\partial{R_k}} = 0$:
            $$
            \begin{align*}
            vec(\lambda R_k + A^TAR_kA^TA) = vec(A^TX_kA) \\
           ((A^TA)^T\otimes(A^TA) + \lambda \mathbf{I})\ vec(R_k) = (A^T \otimes A^T) vec(X_k) \\
           ((A^T \otimes A^T)(A \otimes A) + \lambda \mathbf{I}) vec(R_k) = (A \otimes A)^T vec(X_k) \\
           ((A\otimes A)^T + (A\otimes A) + \lambda \mathbf{I})vec(R_k)  = (A \otimes A)^T vec(X_k) \\
           vec(R_k) =  ((A\otimes A)^T + (A\otimes A) + \lambda \mathbf{I})^{-1} (A \otimes A)^T vec(X_k) \\
            \end{align*}
            $$
            以此更新$R_k$
3. TRESCAL:
    * 思想：   
        利用关系的知识，从损失函数中移除类型不匹配的三元组，从而使得训练效率提高，准确率提高。  
        例如： bornin 这个关系中不可能两个实体都是人
    * 形式化的符号定义：  
        只有在$e_i \in \mathbb{L_k} ,and e_j \in \mathbb{R_k}$的情况下，$(e_i,r_k,e_j)$才是可行的。  
        其中$\mathbb{L_k} , \mathbb{R_k}$分别表示左右部可行的类型。  
        使用$A_{k_l}, A_{k_r}$来表示对应的$A$的子矩阵，分别包含$\mathbb{L_k}$和$\mathbb{R_k}$对应的行。  
        $X_{k_{lr}}$表示$X_k$对应的子矩阵，只包含可行的实体对。  
        希望能够找到这样的估计： 
         ![](/images/TRESCAL.jpg)
    * 定义损失函数： 
        $$
        \begin{align*}
        J &= min\ \frac{1}{2}(\sum_k \left\lVert X_{k_{lr}} - A_{k_l}R_kA_{k_r}^T \right\rVert_F^2) + \frac{1}{2}\lambda(\left\lVert A \right\rVert_F^2 + \sum_k\left\lVert R_k \right\rVert_F^2) \\
        \end{align*}
        $$
    * 优化：  
        推导与之前的RESCAL类似。如何更新：  
        $$
        A = (\sum_k(X_{k_{lr}}^TA_{k_r}R_k + X_{k_{lr}}A_{k_r}R_k^T))(\sum_k(R_kA^T_{k_r}A_{k_r}R_k^T+R_k^TA^T_{k_l}A_{k_l}R_k) + \lambda \mathbf{I}))^{-1}       
        $$
        $$
         vec(R_k) =  (A_{k_r}^TA_{k_r} \otimes A_{k_l}^TA_{k_l} + \lambda \mathbf{I})^{-1} vec(A_{k_l}^TX_{k_{lr}}A_{k_r}) \\
        $$
    * 高效的处理正则项：  
        最耗时的操作是矩阵的逆操作，时间复杂度为$O(n^3)$，$n$为矩阵的行列个数。  
        当前仅考虑RESCAL的情况。
        * 如果$\lambda = 0$，$(Z^TZ+\lambda \mathbf{I})^{-1}$则变为：   
            $$
            (Z^TZ)^{-1} = (A^TA)^{-1}A \otimes (A^TA)^{-1}A
            $$
            这里对$A$求逆，$A\in R^{r \times r}$，所以需要时间$O(r^3)$。
        * 如果$\lambda >0 $,  
            为了降低计算的复杂性，对$A$应用SVD，$A = U \Sigma V^T$：
        ![](/images/SVD.jpg)
            注意到$(\lambda\mathbf{I} + \Sigma^2 \otimes \Sigma^2)^{-1}$是一个对角阵，求逆操作简单。
    * 在TRESCAL中更新$R$的算法： 
        ![](/images/update_R.jpg)
        推导过程： 
        $$
        \begin{align*}
        J &= min\ \frac{1}{2}(\sum_k \left\lVert X_{k_{lr}} - A_{k_l}R_kA_{k_r}^T \right\rVert_F^2) + \frac{1}{2}\lambda(\left\lVert A \right\rVert_F^2 + \sum_k\left\lVert R_k \right\rVert_F^2) \\
        &= min\ \frac{1}{2}\sum_k tr((X_{k_{lr}} - A_{k_l}R_kA_{k_r}^T)^T(X_{k_{lr}} - A_{k_l}R_kA_{k_r}) + \frac{1}{2}\lambda(tr(A^TA) + \sum_k tr(R_k^TR_k)) \\
        \end{align*}
        $$
        $$
        \frac{\partial{J}}{\partial{R_k}} = -A_{k_l}^TX_{k_{lr}}A_{k_r} + A_{k_r}^TA_{k_l}R_kA_{k_r}^TA_{k_r} + \lambda R_k        
        $$
        对$A_{k_l},A_{k_r}$应用SVD分解： 
        $$
        \begin{align*}
        A_{k_l} = U_{k_l}\Sigma_{k_l}V_{k_l}^T \\
        A_{k_r} = U_{k_r}\Sigma_{k_r}V_{k_r}^T 
        \end{align*}
        $$
        令$\frac{\partial{J}}{\partial{R_k}} = 0$：
        $$
        \begin{align*}
        -A_{k_l}^TX_{k_{lr}}A_{k_r} + A_{k_r}^TA_{k_l}R_kA_{k_r}^TA_{k_r} + \lambda R_k = 0 \\
       -A_{k_l}^TX_{k_{lr}}A_{k_r} + V_{k_l}\Sigma_{k_l}^2V_{k_l}^TR_kV_{k_r}\Sigma_{k_r}^2V_{k_r} + \lambda R_k = 0 \\
       -V_{k_l}^TA_{k_l}^TX_{k_{lr}}A_{k_r}V_{k_l} + \Sigma_{k_l}^2V_{k_l}^TR_kV_{k_r}\Sigma_{k_r}^2 + \lambda V_{k_l}^TR_kV_{k_r} = 0 \\
       \Sigma_{k_l}^2V_{k_l}^TR_kV_{k_r}\Sigma_{k_r}^2 + \lambda V_{k_l}^TR_kV_{k_r} &= V_{k_l}^TA_{k_l}^TX_{k_{lr}}A_{k_r}V_{k_l} \\
       diag(\Sigma_{k_l}^2)diag(\Sigma_{k_r}^2)^T \odot V_{k_l}^TR_kV_{k_r} + \lambda V_{k_l}^TR_kV_{k_r} &= V_{k_l}^TA_{k_l}^TX_{k_{lr}}A_{k_r}V_{k_l} \\
       (diag(\Sigma_{k_l}^2)diag(\Sigma_{k_r}^2)^T + \lambda \mathbf{1}) \odot V_{k_l}^TR_kV_{k_r} &= V_{k_l}^TA_{k_l}^TX_{k_{lr}}A_{k_r}V_{k_l} 
        \end{align*}
        $$
        所以：
        $$
        R_k = V_{k_l}^TA_{k_l}^TX_{k_{lr}}A_{k_r}V_{k_l} ./    (diag(\Sigma_{k_l}^2)diag(\Sigma_{k_r}^2)^T + \lambda \mathbf{1})
        $$
        其中$\odot$表示按位乘，$./$表示按位除，$\mathbf{1}$表示全为1矩阵。
        极大的减少了时间
        
### 实验
1. 知识图谱的补全
    训练的时间减少，也就是更新$A$和${R_k}$的所需的空间和时间都减少了。  
    在Entity Retrieval 和 Relation Retrieval 这两个任务上效果都得到了提高。
    ![](/images/result.jpg)
2. 关系抽取  
    从文本语料和结构化的知识库中运用TRESCAL 进行关系抽取。  
    单纯使用TRESCAL模型效果并不好，如果使用TR + RI13能够取得比较好的效果。
    
### Future work
1. 可能可以利用一些tags来帮助词汇的语义表达
2. 在三元组中考虑其他限制，不局限于实体的类型。


