# Machine-Learing Week1

机器学习中的主要两种学习算法，一个是监督学习，另一个是非监督学习。Supervised Learing And Unsupervised Learing  

## Supervised Learing

> 已知一些数据集，通过计算来预测得出结果

主要有两大类： 回归问题 以及 分类问题

回归问题指的是我们的目标是预测一个连续的输出值。

分类问题预测离散值的输出

## UnSupervised Learing

> 非监督学习中，这些数据都是没有区别的，算法不会给出一个正确答案。

聚类算法

鸡尾酒会问题算法

```octave
[w,s,v] = svd(repmat(sum(x.*x,1),size(x,1),1).*x)*x');
```

软件使用 `Octave`