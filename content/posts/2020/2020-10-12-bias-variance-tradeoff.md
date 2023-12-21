---
title: "机器学习中的偏差(Bias)与方差(Variance)"
slug: bias-variance-tradeoff
date: 2020-10-12
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/wind-turbine-1149604_1920.jpg
thumbnailImagePosition: right
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/wind-turbine-1149604_1920.jpg
categories:
- data science
tags:
- machine learning
- bias-variance
math: true
---

一个机器学习方法的泛化性能与它对独立测试数据的预测能力有关。对泛化性能的评估在机器学习实践中极为重要<!--more-->，因为它可以指导学习方法或模式的选择，使我们对最终选择的模型的质量有一个衡量标准。如何权衡偏差和方差是机器学习中不得不面对的任务。

{{< toc >}}

## Bias-Variance Tradeoff

> One of the most reliable ways to get a high performance machine learning system is to take a low bias learning algorithm and to train it on a massive training set.
> <p align='right'>-- Andrew Ng</p>

在有监督学习中，泛化误差的来源主要包括偏差(bias)、方差(variance)和不可减少的误差(irreducible error，也叫噪声)。

- 偏差度量了学习算法的期望预测真实结果的偏离程度，即刻画了学习算法本身的拟合能力。高偏差（high bias）对应了学习算法的欠拟合（underfitting）；为了减少学习算法的偏差，可以增加模型的复杂度，比如增加新的特征项，这样一来又容易造成过拟合（overfitting），从而造成高方差（high variance）。
- 方差度量了同样大小的训练集的变动所导致的学习性能的变化，即刻画了数据扰动所造成的影响。高方差对应了学习算法的过拟合，方差与偏差存在一定冲突，被称为偏差-方差窘境（bias-variance dilemma）。除了通过减少模型特征来降低模型的复杂度进而降低模型方差，还可以通过增加数据规模，或者正则化、交叉验证法等方法减少方差。
- 噪声则表达了在当前任务上任何学习算法所能达到的期望泛化误差的下界，即刻画了学习问题本身的难度。

## Bias-Variance Decomposition

对于观测数据$X$、预测变量$Y$，假设两者符合关系$Y=f(X)+\epsilon$，其中$\epsilon \sim \mathcal{N}(0,\sigma_\epsilon^2)$。我们无法得到真实模型$f(X)$，我们可以选择机器学习模型，比如线性回归、决策树或其他模型，通过算法学习$f(X)$一个近似模型$\hat{f}(X)$来对$X$的观测进行预测。给定$X$的一个观测点$x$，其真实观测（待预测变量）为$y=f(x)+\epsilon$。

该模型$\hat{f}(X)$在点$x$的期望预测误差，以均方损失误差为例：


$$\begin{align}
Err(x)&=E[(y-\hat{f})^2]    \\\
&=E[(f+\epsilon-\hat{f}+E[\hat{f}]-E[\hat{f}])^2]   \\\
&=E[(f-E[\hat{f}])^2]+E[\epsilon^2]+E[(E[\hat{f}]-\hat{f})^2]+2E[(f-E[\hat{f} ])\epsilon]   \\\
&\quad+2E[\epsilon(E[\hat{f}]-\hat{f})]+2E[(E[\hat{f}]-\hat{f})(f-E[\hat{f}])]      \\\
&=(f-E[\hat{f}])^2+E[\epsilon^2]+E[(E[\hat{f}]-\hat{f})^2]+2(f-E[\hat{f}])E[\epsilon]   \\\
&\quad+2E[\epsilon]E[E[\hat{f}]-\hat{f}]+2E[E[\hat{f}]-\hat{f}]\(f-E[\hat{f}]\)     \\\
&=(f-E[\hat{f}])^2+E[\epsilon^2]+E[(E[\hat{f}]-\hat{f})^2]  \\\
&=(f-E[\hat{f}])^2+Var[\epsilon]+Var[\hat{f}]       \\\
&=Bias[\hat{f}]^2+Var[\hat{f}]+Var[\epsilon]    \\\
&=Bias[\hat{f}]^2+Var[\hat{f}]+\sigma^2
\end{align}$$



## Bias-Variance Calculation with Python

采用`mlxtend`可以很方便的计算Bias-Variance误差分解，下面是回归决策树方法的偏差-方差分解。


```python
from mlxtend.evaluate import bias_variance_decomp
from sklearn.tree import DecisionTreeRegressor
from mlxtend.data import boston_housing_data
from sklearn.model_selection import train_test_split

X, y = boston_housing_data()
X_train, X_test, y_train, y_test = train_test_split(X, y,
                                                    test_size=0.3,
                                                    random_state=123,
                                                    shuffle=True)

tree = DecisionTreeRegressor(random_state=123)

avg_expected_loss, avg_bias, avg_var = bias_variance_decomp(
        tree, X_train, y_train, X_test, y_test, 
        loss='mse',
        random_seed=123)

print('Average expected loss: %.3f' % avg_expected_loss)
print('Average bias: %.3f' % avg_bias)
print('Average variance: %.3f' % avg_var)
```

    Average expected loss: 31.917
    Average bias: 13.814
    Average variance: 18.102


作为对比，下面是Bagging方法的偏差-方差，可以看出采用Bagging方法可以降低variance。


```python
from sklearn.ensemble import BaggingRegressor

tree = DecisionTreeRegressor(random_state=123)
bag = BaggingRegressor(base_estimator=tree,
                       n_estimators=100,
                       random_state=123)

avg_expected_loss, avg_bias, avg_var = bias_variance_decomp(
        bag, X_train, y_train, X_test, y_test, 
        loss='mse',
        random_seed=123)

print('Average expected loss: %.3f' % avg_expected_loss)
print('Average bias: %.3f' % avg_bias)
print('Average variance: %.3f' % avg_var)
```

    Average expected loss: 18.593
    Average bias: 15.354
    Average variance: 3.239


## Reference

- [Understanding the Bias-Variance Tradeoff](http://scott.fortmann-roe.com/docs/BiasVariance.html)
- [Bias-Variance Tradeoff - Wikipedia](https://en.wikipedia.org/wiki/Bias–variance_tradeoff)
- [Bias-Variance Decomposition](https://github.com/rasbt/mlxtend/blob/master/docs/sources/user_guide/evaluate/bias_variance_decomp.ipynb)
- 机器学习西瓜书
