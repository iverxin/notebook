[TOC]

# 1. C4.5

## Feature：

1. 有监督学习
2. 

### Advantage

### Disadvantage



# 2. K-means

## Feature：

### Advantage：

### Disadvantage：





# 3. SVM



# 4. Apriori



# 5. EM



# 6. PageRank



# 7. AdaBoost



# 8. KNN

## 8.1 Feature

采用测量不同特征值之间的距离方法来进行分类。



### Advantage

- 精度高
- 对异常值不敏感
- 无数据输入假定

### Disadvantage

- 计算复杂度高、空间复杂度高

- 无法给出数据的内在含义

## 8.2 Scope of application

数值型和标称型



## 8.3 Theory

存在**带标签**的训练样本集，输入没有标签的新数据后，`新数据`的每个特征与`样本集`中数据对应的特征进行比较，然后提取样本集中特征最相似的分类的标签。一般只选择前k个最相似的数据，所以叫做k-邻近算法。

>  example：
>
> 确定一个电影是动作片还是爱情片，已知数据集特征：打斗场面次数、接吻镜头次数。给出新的未标签的电影，计算该电影与数据集中每个电影的距离，然后排序取前k个，通过分析k个电影的种类来确定新的未标签的电影类型。



# 9. Naive Bayes



# 10. CART



# Other

## Difference of C4.5/ID3/CART

- ID3决策树利用信息熵增益来划分结点

- C4.5决策树：先算信息熵增益，然后取增益率最高的
- CART决策树(Classification And Regression Tree)：可用于分类和回归