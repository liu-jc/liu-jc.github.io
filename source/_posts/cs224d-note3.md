---
title: cs224d-note3
date: 2017-03-31 11:30:14
tags:
---
note3主要介绍了神经网络的基本原理与一些技巧，都比较简单。以前看过写起来比较快，但是不排除我理解有错误或者说我写错了，所以如果发现有错误，希望指出并与我联系。 
Keyphrases:
* 前向计算，后向传播
* Max-margin loss
* 梯度检测以及参数初始化
* 学习率和Adagrad

<!--more-->
### 神经网路基础
由于大量的数据是非线性可分的，所以需要非线性分类器，那么神经网络就是一组非线性分类器。

1. 一个神经元  
单个计算单元将n维的向量 $x$ 作为输入，产生一个标量的输出 $a$，当然这个神经元同样与一个n维的权值向量$w$连接，以及一个偏移标量$b$。
假设我们使用了sigmoid 激活函数： 
$$ 
f(z) = \frac1 {1 + exp(z)}
$$
神经元的输出可以表示为： 
$$
a = \frac1 {1+exp(-w^Tx+b)}
$$
将偏移量和权值向量组合到一起可表示为： 
$$
a = \frac1 {1+exp(-[w^T\quad b] \cdot [x\quad1]}  
$$
下图直观的表示了这一过程
![](/images/14909331043266.jpg)    

2. 单层的网络
就是简单的将每个神经元叠加起来就好了。都比较简单。 
假设这一层有m个神经元，前一层有n个神经元： 
![](/images/14909338524658.jpg)
定义符号，并都用矩阵和向量表示
![](/images/14909339654416.jpg)
那么很容易知道激活前的值就是: 
$$
z = Wx + b 
$$
经过sigmoid激活函数之后： 
![](/images/14909341580955.jpg)

3. 前向传播计算
输入向量$x \in R^n$,经过一层网络生成 $a \in R^m$
使用另一个矩阵$U \in R^{m*1}$来生成一个没有被标准化的值用于进行分类任务： 
$$
s = U^Ta = U^Tf(Wx+b)
$$
直观来说假设我们4维的词向量以及window的大小为5，作为输入,那么$x \in R^{20}$。如果使用8个神经元，生成一个值，那么$W \in R^{8*20}$,$b \in R^8$, $U \in R^{8*1}$, $s \in R$。

4. 最大间距目标函数
思想在于确保正样本的值高于负样本的值。
假设正样本的值为$s$,负样本的值为$s_c$
$s_c = U^Tf(Wx_c+b)$,$s = U^Tf(Wx+b)$
那么目标函数的就是希望去最大化$(s-s_c)$,或者说是最小化$(s_c-s)$。
如果$s_c-s<0$，我们就不管认为这时候的代价为0，只计算负样本的值比正样本的值大的时候。所以修改目标为： 
$$
minimize J = max(s_c-s,0)
$$ 
还可以修改一下，设定一个安全的边界$\Delta$，在$s - s_c < \Delta$时计算，修改优化目标： 
$$
minimize J = max(\Delta+s_c-s,0)
$$
可以取$\Delta = 1$，则：
$$
minimize J = max(1+s_c-s,0)
$$

5. 后向传播
更新网络参数的关键在于求出$J$对每个参数的导数。利用链式法则进行。给出如下的网络图： 
![](/images/14909367021587.jpg)

可以用$W^{(2)} = U$来表示统一权值的符号
假设现在要对$W_{14}^{(1)}$进行更新，可以知道$w_{14}^{(1)}$只对$z^{(2)}_1$以及$a^{(2)}_1$有贡献，所以只受这两个的影响。可以先求出$j$对$s$的导数
$$
\frac{\partial J}{\partial s} = - \frac{\partial J}{\partial s_c} = -1
$$
那么接下来求$\frac{\partial s}{\partial W^{(1)}_{ij}}$：![](/images/14909371685191.jpg)

令$\delta_i^{(2)} = W_i^{(2)}f'(z_i^{(2)})$  
则可得 $\frac{\partial s}{\partial W_{ij}^{(1)}}= \delta_i \cdot a_j^{(i)}$

根据上面的过程可以很容易的知道： 
$$
\begin{split}
\frac{\partial s}{\partial {b_{i}^{(2)}}} &= W_i^{(2)} \frac{\partial {a_i^{(2)}}} {\partial {z_i^{(2)}}} \frac{\partial {z_i^{(2)}}} {\partial {b_{i}^{(2)}}}\\
&=W_i^{(2)}f'(z_i^{(2)})\\
&=\delta_i^{(2)}
\end{split}
$$
lecture note上还提供了另一种解释，这里就不赘述了，有兴趣的可以看一下。

将$\delta^{(k)}$推广到$\delta^{(k-1)}$：
直观的图片演示
![](/images/14909469255545.jpg)

链式求导，推导：
$$
\begin{split}
\delta_j^{(k-1)} &= \frac{\partial s}{\partial z_j^{(k-1)}}\\
&= \frac{\partial s}{\partial z^{(k)}} \frac{\partial z^{(k)}}{\partial z_j^{(k-1)}}\\
&=  \frac{\partial s}{\partial z^{(k)}} \frac{\partial z^{(k)}}{\partial a_j^{(k-1)}}\frac{\partial a_j^{(k-1)}}{\partial z_j^{(k-1)}}\\
&= \sum_i \frac{\partial s}{\partial z_i^{(k)}}\frac{\partial z_i^{(k)}}{\partial a_j^{(k-1)}}\frac{\partial a_j^{(k-1)}}{\partial z_j^{(k-1)}}\\
&= f'(z_j^{(k-1)})\sum_i \delta_i^{(k)}W_{ij}^{(k-1)}
\end{split}
$$
6. 向量化后向传播的过程
因为
$$
\frac{\partial s}{\partial W_{ij}^{(k)}}= \delta_i^{(k+1)} \cdot a_j^{(k)}
$$

所以$W^{(k)}$的导数为：
![](/images/14909480724842.jpg)

现在计算$\delta$的向量化传播：
$$
\delta_j^{(k)}= f'(z_j^{(k-1)})\sum_i \delta_i^{(k)}W_{ij}^{(k-1)}
$$
$$
\delta^{(k)} = f'(z^{(k)}) \circ (W^{(k)T}\delta^{(k+1)})
$$
($\circ$ 为元素按位相乘：$R^N \times R^N \rightarrow R^N$）

在计算的时候$\delta^{(k)}$只取决于前一个$\delta^{(k+1)}$

### Tips & Tricks
1. 梯度检查 
利用一个数值方法估计梯度： 
$f'(\theta) \approx \frac{J(theta^{(i+)} - J(\theta^{(i-)})}{2\varepsilon}$
其中 $\theta^{(i+)} = \theta + \varepsilon \times e_i$
利用这个数值估计跟反向传播得到的梯度进行比较，进行梯度检查，是否出错。

2. 正则化
用来防止overfitting，利用$L_2$范数：
$$
J_R = J + \lambda\sum_{i=1}^L\left \lVert W^{(i)}\right \rVert_F
$$
其中 $\left \lVert W^{(i)} \right \rVert_F$ 为Frobenius norm 
利用这样的正则项来惩罚权值过大，从而降低了分类器的灵活性降低了过拟合的风险。
要注意的是偏移量不需要被正则化，这里我的理解是如果限制偏移量，会限制灵活性，从而导致泛化能力变差吧？ 因为偏移量的作用就是让他离开0的位置，如果惩罚的话$\lambda$极度大的时候等于就没有了。（个人理解也不知道对不对）

3. 激活函数（具体可以查，这里简单说一下个别）
    * sigmoid
    * Tanh
    * Hard tanh
    * Soft sign   
        $\text{softsign}(z) = \frac{z}{1 + |z|}$  
        这里我感觉lecture note 里面导数推错了，我认为是这样的:
        $\text{softsign}'(z) = \frac{1}{(1+sgn(z)*z)^2}$  
        其中$sgn$ 为$\pm1$取决于$z$的符号
    * ReLU(很常用，并且好像有文章对比过收敛会快点）
        $$
        rect(z) = max(z,0)
        $$
        $$ rect'(z)=\left\{
        \begin{aligned}
        & 1 : z>0\\
        & 0 : otherwise\\ 
        \end{aligned}
        \right.
        $$
    * Leaky ReLu

4. Xavier参数初始化
    对于sigmoid 和 tanh的激活单元，当$W \in R^{n^{(l+1)} \times n^{(l)}}$时以这样的方式进行初始化的话，可以获得更快的收敛速度和更小的错误率：
    ![](/images/14909642115056.jpg)
对于偏移量的话初始化为0

5. 学习率

固定的学习率$\alpha$，不能设置太大
![](/images/14909657862801.jpg)
还有一种方法是让学习率随着迭代次数下降：
![](/images/14909660780243.jpg)

其中$a_0$是一个可调的参数表示初始的学习率，$\tau$也是可调的参数表示学习率开始下降的时间

6. 自动调节学习率的方法，Adagrad  
    可以使得每个参数的学习率都不同，取决于历史的更新，越少进行更新的参数采用的学习率大于那些更新频繁的参数。
    具体如下：
    ![](/images/14909665683626.jpg)
    简单实现如下： 
    ![](/images/14909666223917.jpg)




