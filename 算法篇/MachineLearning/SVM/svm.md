# SVM

## 1. SVM基本形式

给定训练样本即为$D=\{(\bf{x_1}, y_1), (\bf{x_2}, y_2), \ldots, (\bf{x_m}, y_m)\}$, 其中$y_i \in \{-1, +1\}, \bf{x_i} \in R^d$.

SVM的基本基本想法则是基于训练集$D$在样本空间找到一个划分的超平面，将不同的类别来区分开，那么接下来问题就是如何寻找这个超平面。

即设超平面为$\bf{w}^Tx + b = 0$，则参数$(\bf{w}，b)$则是需要学习的参数。

假设超平面可以将样本正确分类，则有
$$
\begin{cases}
\bf{w}^Tx_i+b \geq +1, \quad y_i=+1 \\
\bf{w}^Tx_i+b \leq +1, \quad y_i=-1
\end{cases}
$$
上面提到的两个异类的超平面的间隔为$\frac{2}{||\bf{w}||}$，则我们可以定义我们模型的目标为，最大化间隔，
$$
\begin{aligned}
& \max_{w,b}\ \frac{2}{||\bf{w}||} \\
& s.t.\ y_i(\bf{w^T}x_i + b) \geq 1, i=1,2,\ldots,m
\end{aligned}
$$
进一步可以表达为
$$
\begin{aligned}
& \min_{w,b}\ \frac{1}{2} ||w||^2 \\
& s.t.\ y_i(\bf{w^T}x_i + b) \geq 1, i=1,2,\ldots,m
\end{aligned}
$$
这就是支持向量机的基本形式。

## 2. 支持向量机的求解

对于上面的基本型，使用拉格朗日乘子法得到其对偶问题，来进行求解。

上面问题的拉格朗日函数可以写为
$$
L(w, b, \alpha) = \frac{1}{2}||w||^2 + \sum_{i=1}^m \alpha_i(1-y_i(w^Tx_i+b))
$$
其中，$\alpha_i, i=1,2,\ldots,m$为拉格朗日乘子

对$w$和$b$求偏导令其为零，得$w=\sum_{i=1}^m \alpha_i y_i x_i$和$\sum_{i=1}^m\alpha_i y_i = 0$. 

然后将这个两个式子带入上述拉格朗日函数有
$$
\begin{aligned}
& \max_{\alpha} \sum_{i=1}^m \alpha_i - \frac{1}{2}\sum_{i=1}^{m} \sum_{j=1}^m \alpha_i \alpha_j y_i y_j x_i^T x_j \\
& s.t. \sum_{i=1}^m \alpha_i y_i = 0, \\
& \quad\ \ \alpha_i \geq 0, i=1,2, \ldots, m
\end{aligned}
$$
在求出$\alpha$后，超平面即可以确定，
$$
\begin{aligned}
f(x) &= w^Tx + b \\ 
     &= \sum_{i=1}^m \alpha_i y_i x_i^Tx + b
\end{aligned}
$$
除了上面的求解外（上面相当于已经考虑下述不等式的约束了），不等式约束的对偶问题需要满足KKT条件，上述问题的KKT条件为
$$
\begin{cases}
\alpha_i \geq 0; \\
y_i f(x_i) - 1 \geq 0; \\
\alpha_i(y_if(x_i) - 1)=0.
\end{cases}
$$
从上面条件可以得出，若$\alpha_i = 0$，则该样本不会对函数有任何影响，反之若$y_if(x_i)=1$，则表示该样本位于最大间隔的边界上，表示这是一个支持向量。所以：**训练完成后，大部分的训练样本都不要保留，最终模型只与支持向量有关**。



最终在对偶问题的求解上，采用SMO(Sequential Minimal Optimization)算法，基本思路是先固定$\alpha_i$之外的所有参数，然后求$\alpha_i$上的极值。优于存在约束$\sum_{i}^m y_i \alpha_i = 0$,若固定$\alpha_i$之外的其他变量，则$\alpha_i$可由其他变量导出，于是SMO算法每次选择两个变量$\alpha_i, \alpha_j$，并固定其他参数，在参数初始化后，SMO不断执行下面两个步骤直至收敛：

- 选取一对需要更新变量$\alpha_i, \alpha_j$
- 固定$\alpha_i, \alpha_j$以外的参数，通过求解对偶问题来更新$\alpha_i, \alpha_j$

## 2. 核函数

上面做的求解中，是默认训练样本是线性可分的，即存在一个超平面可以将训练样本正确分类，而显示任务中，往往这个假设是不成立的，所以一个想法是将样本的原始空间映射到一个更高维的特征空间，使得样本在新的特征空间中线性可分。

> 如果原始空间有限维，即属性（特征）有限，那么一定存在一个高维特征空间使得样本可分

令$\phi(x)$表示将$x$映射后的特征向量，于是新的特征空间下所对应的超平面为
$$
f(x) = w^T \phi(x) + b
$$
其中$w$和$b$是模型的参数

后面求解过程完全类似上面的过程，

基本形式可以写为
$$
\begin{aligned}& \min_{w,b}\ \frac{1}{2} ||w||^2 \\& s.t.\ y_i({w^T}\phi(x_i) + b) \geq 1, i=1,2,\ldots,m\end{aligned}
$$
对偶问题为
$$
\begin{aligned}
& \max_{\alpha} \sum_{i=1}^m \alpha_i - \frac{1}{2}\sum_{i=1}^{m} \sum_{j=1}^m \alpha_i \alpha_j y_i y_j \phi(x_i)^T \phi(x_j) \\
& s.t. \sum_{i=1}^m \alpha_i y_i = 0, \\
& \quad\ \ \alpha_i \geq 0, i=1,2, \ldots, m
\end{aligned}
$$
上面的求解涉及到计算$\phi(x_i)^T\phi(x_i)$，这是样本$x_i$和$x_j$映射到特征空间后的内积，由于特征空间维数可能很高，甚至是无穷维，因此直接计算$\phi(x_i)^T\phi(x_i)$是困难的，为了避开这个障碍，如果可以设计这样一个函数：

$$
k(x_i, x_j) = <\phi(x_i), \phi(x_j)>=\phi(x_i)^T\phi(x_i)
$$

即$x_i$和$x_j$在新特征空间的内积等于它们在原始样本空间通过函数$k(\cdot, \cdot)$计算的结果。显然如果有这样的函数，就不不需要计算高维甚至是无穷维特征空间的内积（这种方式称为核技巧）

所以上面的对偶问题可以进一步写为
$$
\begin{aligned}
& \max_{\alpha} \sum_{i=1}^m \alpha_i - \frac{1}{2}\sum_{i=1}^{m} \sum_{j=1}^m \alpha_i \alpha_j y_i y_j k(x_i, x_j) \\
& s.t. \sum_{i=1}^m \alpha_i y_i = 0, \\
& \quad\ \ \alpha_i \geq 0, i=1,2, \ldots, m
\end{aligned}
$$
在求得$\alpha$后，最终的模型可以表示为
$$
\begin{aligned}
f(x) &= w^T \phi(x) + b \\ 
     &= \sum_{i=1}^m \alpha_i y_i \phi(x_i)^T\phi(x) + b \\
     &= \sum_{i=1}^m \alpha_i y_i k(x, x_i) + b
\end{aligned}
$$

> 只要一个对称函数所对应的核矩阵半正定，它就可以作为核函数。

**我们希望样本可以在特征空间中线性可分，因此特征空间的好快对支持向量机至关重要，而核函数隐式的定义了这个特征空间，于是核函数的选择称为支持向量机的最大变数，如果核函数选择不合适，则以为将样本映射到了一个不合适的空间，很可能导致分类性能不佳。**

## 3. 软间隔

上面所做的讨论均是在假设训练样本在特征空间是线性可分的，即在特种空间中，存在一个超平面将不同类的样本完全划分开，亦或是可能存在一个这样的超平面线性可分，但是有可能是过拟合导致的。

缓解这种问题的办法是允许支持向量机在一些样本出错，因此提出**软间隔**的概念。

具体来说，上面讨论下是要求所有的样本均满足约束，即所有的样本需要划分正确，这种为**硬间隔**，而软间隔则是允许部分样本不满足约束($y_i(w^Tx_i+b) \geq 1$)，或者说允许部分样本划分错误。

最后我们优化函数应该在最大化间隔的同时，不满足约束的样本应该尽量少，可以写为
$$
\min_{w,b} = \frac{1}{2} ||w||^2 + C\sum_{i=1}^m \text{loss}(y_i(w^Tx_i+b) - 1)
$$
其中$\frac{1}{2}||w||^2$为最大化间隔的表示，$\text{loss}$为损失函数，用户表征当样本误分时，所应当对应的损失，$C$则用于权衡两者，$C=0$时回退至上面的形式，而$C=+\infty$时，表示所有的样本应当满足约束。

在支持向量机中，损失函数$\text{loss}(z) = \max(0, 1-z)$，即为hinge损失。

则上式变为
$$
\min_{w,b} = \frac{1}{2} ||w||^2 + C\sum_{i=1}^m \text{max}(0, 1- y_i(w^Tx_i+b))
$$
进一步引入松驰变量$\xi_i >= 0$，整体式子可以写为
$$
\begin{aligned}
& \min_{w,b,\xi} \frac{1}{2} ||w||^2 + C \sum_{i=1}^m \xi_i \\
& s.t.\ y_i(w^Tx_i + b) \geq 1-\xi_i \\ 
& \quad\ \ \ \xi_i \geq 0, i=1,2,\ldots,m
\end{aligned}
$$
其中$\xi_i = \max(0, 1-y_i(w^Tx_i + b))$.

求解方式和上面类似，转换为对偶问题然后通过SMO算法求解。**指的注意的是，在引入软间隔这个概念后，在分析KKT条件时，和前面的结论一致，最终模型仅于支持向量有关，即通过采用hinge损失函数仍然保持了稀疏性**。

> 其实上面的损失函数也可以替换为其他的损失函数，比如对数损失，其实如果将其替换为对数损失，那么几乎就得到了逻辑回归模型，其实两者优化的目标接近，但是对数损失输出具有概率意义，而支持向量机则不能输出概率，通过hinge损失可以看出，其图像中有一块平坦的零区域，这使得解具有稀疏性，而对数损失则是单调递减的函数，显然不可以导出类似支持向量机这样的概念。

## 4. 总结

对于上面的优化目标，可以看出主要分为两部分，优化目标中第一项用来描述划分超平面的**间隔**大小，另一项$\sum_{i=1}^m \text{loss}(f(x_i), y_i)$用来表示训练集上的误差 ，写为一般的形式
$$
\min_f \Omega(f) + C\sum_{i=1}^m \text{loss}(f(x_i), y_i)
$$
其中$\Omega(f)$称为结构风险，用于描述模型$f$的某些性质；第二项称为经验风险，用于描述模型与训练数据的契合程度；$C$用于对二者进行折中。

其实$\Omega(f)$就是我们所说的正则项，**它表述了我们希望获得具有何种性质的模型（例如希望获得复杂度较小的模型）；另一方面，该信息有助于削减假设空间，从而降低最小化训练误差的过拟合风险**。



- 优点

  - 对小数据集效果不错，泛化能力也很好
  - svm允许决策边界很复杂，即使数据只有几个特征。它在低维数据和高维数据（即很少特征和很多特征）上都表现都很好
  - 有着非常扎实理论推导基础

- 缺点

  - 大规模训练样本难以实施（存储训练样本和核矩阵(m阶)空间消耗大,时间复杂度高O(m2) )

    sklearn上提到超过10000条数据，就难以处理了

  - 对参数调节和核函数的选择敏感，对缺失数据敏感

  - 原始分类器不加修改仅适用于处理二分类问题（改进：通过多个二类支持向量机的组合来解决等等）

  - 不支持概率估计(sklearn是通过5-折交叉实现的)

  - 高斯核容易过拟合

### References

- [SVM优点、缺点和参数](https://zhuanlan.zhihu.com/p/48101233)
- [支持向量机（SVM）复习总结](https://www.cnblogs.com/arachis/p/SVM.html)
- 机器学习西瓜书(p121-p133)

