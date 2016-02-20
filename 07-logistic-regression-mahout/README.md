# 实例分析：Mahout 机器学习研究红酒质量影响因素


## 准备知识

* Logistics回归和随机森林分类算法
* Mahout和R
* Hadoop Streaming




### 数据预处理
从 UCI 机器学习库直接下载得到的数据集 winequality-red.csv 是分号分割格式的数据, 且没有变量名信息,参考它提供的文件 winequality.names,使用 R 读入数据,并添加变量 名信息. 为了比较不同机器学习的算法,首先,使用分层随机抽样的方式(根据因变量
quality 分层),取数据集的 70% 为训练集,剩下的 30% 作为测试集. 拆分后,训练集 共有 1120 个样本,测试集共有 479 个样本,分别保存为文件 train_redwine.csv 和 test_redwine.
## 使用 Mahout 计算 Logistic 回归

Logistic 回归常是一种有效的分类问题学习方法,并且模型可解释性较好. 然而它的求 解算法是迭代的,在 Hadoop 分布式计算中的 MapReduce 一般要求算法是可以拆分并行
进行的,也就是要求算法不是迭代的. 所以要分布式计算 Logistic 回归的解,简单的对 Logistic 回归求解的迭代算法拆分成 Map 和 Reduce 两个部分是不合理的.




### Logistic 回归模型训练
因为在这个问题中我们使用的数据集样本量仅 1000 多,所以为了计算的准确性,仅使用单机模式计算模型的解. 在 Linux 系统下的 Mahout 下计算 Logistic 的解的代码如下。



```
$ mahout trainlogistic --input train_redwine.csv \
```

以上代码在存放数据集train_redwine.csv和test_redwine.csv的目录下运行. 其中使用的选项包括:

- trainlogistic代表计算Logistic回归的模型解

### Logistic 回归模型测试


```
$ mahout runlogistic --input test_redwine.csv \ 
> --model ./logit_model --auc --confusion
```
以上代码中使用的选项包括:

- runlogistic代表通过测试集检验之前得到的Logistic回归的模型解的效果
输出结果显示该模型的 AUC 得分为 0.64 > 0.5 ,说明训练的效果还可 以. 混淆矩阵显示品质为 0 的 223 个红酒样本有 148 个被判断为 0,75 个判断为 1;品质 为 1 的 256 个红酒样本有 121 个被判断为 0,135 个被判断为 1. 从混淆矩阵显示的误判 率可以看出 Logistic 回归的模型学习结果不够令人满意.



在 Linux 系统下的 Mahout 下并行计算随机森林模型要求的输入数据集是纯数据集, 不包含变量信息,所以我们将 Logistic 回归中使用的数据集 train_redwine.csv 和 test_redwine.csv 删除首行变量名信息,并删除红酒样本序号一列,得到数据集 redwine_train.arff 和 redwine_test.arff.


$ hadoop jar $MAHOUT_HOME/mahout-examples-0.10.1-job.jar \ > org.apache.mahout.classifier.df.tools.Describe \






$ hadoop fs -chmod 751 redwine_train.arff 
$ hadoop fs -chmod 751 redwine_train.info



$ hadoop jar $MAHOUT_HOME/mahout-examples-0.10.1-job.jar \ 
> org.apache.mahout.classifier.df.mapreduce.BuildForest \





$ hadoop jar $MAHOUT_HOME/mahout-examples-0.10.1-job.jar \ 
> org.apache.mahout.classifier.df.mapreduce.TestForest \


得到的随机森林模型测试输出结果可以看到随机森林的正确率高达 98.33%. Logistic 模型的测试结果的混淆矩阵,随机森林模型的准确性十分高. 虽 然随机森林的判断结果准确性更高,但是模型解释性相较 Logistic 回归模型较差,我们没 有办法通过输出的结果说明红酒的各个属性变量在判断红酒质量方面的贡献.
（感谢北京大学张诗玉提供素材和案例。）