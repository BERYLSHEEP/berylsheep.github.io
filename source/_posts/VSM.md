---
layout: post
title: "复现Entropy-based Term Weighting Schemes for Text Categorization in VSM小结"
date: 2018-06-01
comments: true
tags: 
   - 机器学习
   - NLP
   - VSM
---


论文 [Entropy-based Term Weighting Schemes for Text Categorization in VSM](https://ieeexplore.ieee.org/document/7372153/) 提出了新的基于熵的用于文本分类的词权重计算方法tf·dc,tf·bdc，通过和目前流行的权重计算方法如tf·idf, tf·chi, tf·ig, tf·eccd, tf·rf and iqf·qf·icf进行实验比较，证实其提出的计算方法的可行性和优越性。 笔者通过复现论文新提出的tf_dc，tf_bdc，以及用于实验比较的tf·idf, tf·chi, tf·ig, tf·eccd, tf·rf and iqf·qf·icf，在使用和论文实验一样的语料库 Reuters-R8 和 同样的分类模型 ：KNN和SVM后，发现确实如同论文说的outperform，起码在Reuters上的分类结果优于tf-idf。

在此将整个复现的流程记录和小结一下，从阅读论文到实现计算方法再到使用分类模型到评估结果，整个过程虽然遇到了不少问题，但最终能够逐个克服并最终完成复现。

复现基于python2.7，KNN使用[sklearn](http://scikit-learn.org/)包，SVM和原论文同样使用[liblinear](https://www.csie.ntu.edu.tw/~cjlin/liblinear/)，鉴于只是大致复现，因此除了和原论文同样对KNN的邻居数目参数进行实验外，没有细致对knn和SVM做调参。

<!-- more -->

## 理论介绍

### VSM向量空间模型

在自然语言处理过程中，第一步都是将要处理的字、词或文本转换成向量，毕竟计算机不懂文字，它只会处理数字。把词转换成向量我们有one hot, word embedding。到了文档层级，既然文档是由词语组成的，那么可以试着用词语来表示文档。来看看一个用one hot表示文章的例子：

假设词汇表有  ['one', 'apple','a','day','an'], 此时只使用one hot，即只判断记录词是否出现，不记录词的频率

文章a = "one day". 那么 它的向量则是 [1,0,0,1,0]

文章b = "an apple"， 则代表b的向量是[0,1,0,0,1]

one hot表示法虽然简单，但也有很多缺点，比如只记录词出现与否，词的区分能力被认为是一样的等等，由此人们提出了很多计算方法，核心思想就是表示出**一个词的辨别能力**。词语的辨别能力是指：这个词将一篇文档从其它文档中区分出来的能力（或者将一个类从其它类区分出来的能力）,比如说一篇文章出现 "算法" 这个词较多，那么它通常会是计算机等领域的文章，而不太可能会是体育、艺术类的文章。

就拿之前的例子而言，an和One这种词明显在很多地方都会出现，因此它们的辨别能力不强，而apple就比它们好一点，那么在特征权重计算中，它的权重就会比其它两个高一些。

由此，每个词语对应一个维度，每个词语有一定的权重（由训练语料训练出来，代表这个词区分各类文档或各个标签的能力），再结合词语在文本出现的次数，就能够构成一个多维向量，将文档成功投射到多维空间中，这就是向量空间模型。投射之后，计算文章之间的相似度就可以有很多方法了，比如直接计算空间当中的距离啊，cosine啊等等，那么我们就可以将文章归到和它相似度高的那类中，由此完成文档分类的过程。

### 旧方法

为什么需要提出新的权重计算方法呢？因为旧的不够好，不够好在哪里？论文给出了理由：

大多数监督学习的计算方法基于词在 PC（positive category） 和 NC（negative category） 中的出现次数，就会有以下问题：

1. PC是单独一个类， 而 NC包含多个类，而把它们统一成一个数字，那么显然NC的数目要远远大于PC，在权重计算中也会占主导地位。
2. NC包含多个类，仅归为一个数字后，词语在这些类中的分布信息就丢失了
3. 计算权重得基于标签，但测试文档本身就不具备标签

对于非监督的计算方法，就拿tf-idf来说，其能力在于**将一篇文档从其它文档区分出来，而不是将一个类从其它类区分出来**。

文章列举了其它较为流行的权重计算方法，并依照上面提出的问题一一举出了例子。

#### tf·idf

作为最流行的权重计算方法，其计算方法分为两个部分

一个是tf(i,j)，即词i在文章j中出现的频率： $tf(i,j) = \frac{n_{i,j}}{\sum_{k} n_{k,j}}$ , 用词i出现的次数 / 这篇文章总长即可

另一个是 idf(i,j)，称为逆文档频率，和这个词出现的文档数相关：  $idf(i) = log \frac{|D|}{nd_{i}}$ , 用文档总数 / 出现了 i 的文档数目，而后再取log，一般为了防止分母为0会在分母上加一

最终的 tf-idf就等于两者相乘。

#### tf·chi

基于统计的卡方检验和下面一些计算方法都基于以下这个表格：

![](http://ot1c7ttzm.bkt.clouddn.com/abcd.png)

A：类别k中出现了词j的文档数目

B：除类别k外的其它类出现了词j的文档总数，用词j出现的文档总数 - A 即可

C：负文档数目，即类别k中不包含词j的数目，用 类别k的文章总数-A

D：其它类别不包含词j的数目，用其它类文档总数 - B

卡方检验的原始公式和近似公式：

![](http://ot1c7ttzm.bkt.clouddn.com/vsm_chi.png)

#### tf·ig

Information Gain 信息增益：增加了这个信息使得系统的熵降低了多少。

在特征权重计算中，以词语出现与否分别计算整个语料库的熵，以熵的差值作为词语的信息增益，即词的权重。

![](http://ot1c7ttzm.bkt.clouddn.com/ig.png)

P(Ci)：表示类别Ci出现的概率，用Ci包含的文档数除以文档总数 

P(t)：词语T出现的概率，用出现过T的文档数除以总文档数 

P(Ci|t)：出现T的时候，类别Ci出现的概率，用出现了T并且属于类别Ci的文档数除以出现了T的文档数 

P(～t)：词语T不出现的概率，用 1 - P(t) 即可

P(Ci|～t)表示未出现T的时候，类别Ci出现的概率，用未出现了T并且属于类别Ci的文档数除以未出现T的文档数 

#### tf·eccd

论文 [Entropy based feature selection for text categorization](https://hal.archives-ouvertes.fr/hal-00617969/document) 同样提出了一种基于熵的权重计算方法

![](http://ot1c7ttzm.bkt.clouddn.com/eccd0.png)

![](http://ot1c7ttzm.bkt.clouddn.com/eccd1.png)

#### tf·rf

由于表格中B、D的数目显然很大，为了避免它们带来的影响，人们提出了relevance frequency（rf），只是用 a和c的比值来表明一个词的辨别能力。

![](http://ot1c7ttzm.bkt.clouddn.com/rf.png)

#### iqf·qf·icf

这篇论文  [Term weighting schemes for question categorization](https://www.ncbi.nlm.nih.gov/pubmed/20733219) 面对短文本（用户提出的问题）提出三种新的权重计算方式： iqf\*qf\*icf、qf\*icf 和 vrf。

和rf相比，iqf\*qf\*icf额外考虑了一个词出现了类数目，然而正如论文提出的那样，只考虑了类的数目，却没有考虑到词在这些类内部的分布情况。

![](http://ot1c7ttzm.bkt.clouddn.com/iqf.png)

![](http://ot1c7ttzm.bkt.clouddn.com/iqf1.png)

### 新的权重计算方法

#### tf·dc

论文论证了用熵来表示词语辨别能力的可行性，由此提出了新的计算方法 dc：distribution concentration。

思想基于两点：

1. 辨别能力和词语在所有类的集中程度有关，词语集中程度越高，则它只出现在很少几个类，那么它的辨别能力就越高
2. 集中程度越高的词，具有的熵越小

![](http://ot1c7ttzm.bkt.clouddn.com/dc.png)

H(t)即代表t的熵， f(t,ci)表明词语t在类别 ci出现的文档数目

由于 $H(t) \in [0,log(|C|)]$ ，在除了log(|C|)之后就能将熵化为[0,1]区间，这使得同一篇文章内的词有了可比性。

#### tf·bdc

考虑到现实中语料库类别包含的文档数目有差异，并非理想中所有类的文档数大致相等，那么为了平衡类之间的规模差异，论文在dc的基础上提出了bdc：balanced distribution concentration。

为了避免大类文档数目过多带来的偏差，论文将绝对频率换成了概率，由此平衡类之间的差异。

![](http://ot1c7ttzm.bkt.clouddn.com/bdc.png)

## 复现过程

### 用特征权重进行文本分类的思路

文本分类的整个过程如下：

1. 通过训练语料库计算得到词语权重，并通过将语料库文章投影成向量构成训练特征，标签则为类标签索引，以此训练KNN或SVM模型。 此步主要得到三样东西：
   1. 词语权重表
   2. 词汇表：计算词语频率后删减频率过高和过低的词的产物，每个词汇表里面的词将作为一维，每篇文章为 1*n 的向量，n为词汇表大小。
   3. 模型参数
2. 对于每篇测试文档：
   1. 根据词汇表删去无关词汇
   2. 查词语权重表，若使用tf则额外计算每个词语在文本中出现的频率。 得到每个词语的词语权重，由此得到文档的向量表示
   3. 将文档向量作为特征输入分类模型中，得到预测结果

### 数据处理

语料库和论文中同样选用路透社的语料 Reuters-21578 R8，鉴于Reuters的语料是有名的难处理再加上复现的重点不在此，因此笔者直接使用处理好的语料：[Reuters-21578 R8](http://ana.cachopo.org/datasets-for-single-label-text-categorization)，[TrainingSet](http://ana.cachopo.org/datasets-for-single-label-text-categorization/r8-train-all-terms.txt?attredirects=0)， [TestSet](http://ana.cachopo.org/datasets-for-single-label-text-categorization/r8-test-all-terms.txt?attredirects=0) .

获得语料之后，一个比较重要的地方在于制作特征向量和标签。

我的特征向量和标签制作方法是：

1. 对于文档，首先将所有词转换成one hot，转换使用了sklearn.preprocessing中的Encoder，然而一件很重要的事情在于one hot的顺序，因为Encoder会按照词汇出现顺序设置one hot的顺序，因此我的做法是把文档的词连接在词汇表后面一起导入Encoder，而后再对生成的one hot进行截取，因为词汇表的顺序和大小是固定的，所以截取也很方便。 需要注意的是要将词汇表设置为全局的，并且仅在训练集中构造词汇表，而测试集仅用词汇表进行筛选。
2. 对于类别标签，同样在训练集中构造标签表，而后串接起来导入Encoder再截取，然后用argmax获得唯一的1所在的下标，由此将字符串转换成了单个数字。

### 维度压缩

在复现的过程中，首先遇到的第一个小问题就是维度过大，虽然只使用了几MB的语料库，但要是把每个词都作为一维的话，那就有将近两万维，刚开始直接运行的时候电脑就报出超出内存的错误了。

解决方法就是删去频率过高和过低的词：

1. 统计训练语料中的词语频率得到词频表和词汇表
2. 使用Counter得到各个频率的词汇数目并使用matplotlib.pyplot将词汇频率绘制成直方图，此外还将词汇表的长度作为额外参考
3. 根据长度、频率分布挑选阈值，根据上下界删减词汇表
4. 根据词汇表删去训练和测试语料的其它词，仅保留在词汇表中的词语。

通过这么个维度压缩，使得维度从两万维降低到五千多维，不仅加快了运行速度，减少了运行需要的空间，同时也减少了停用词和自造词的干扰。

![](http://ot1c7ttzm.bkt.clouddn.com/vsm_freq1.png)

<center>删减前</center>

![](http://ot1c7ttzm.bkt.clouddn.com/vsm_freq2.png)

<center>删减后</center>

### KNN中的cos近似

下一个比较重要的小问题在于sklearn中的KNN提供的计算距离的函数并没有cos距离，而后在寻找解决方案时发现了这条[stackoverflow上的回答](https://stackoverflow.com/questions/34144632/using-cosine-distance-with-scikit-learn-kneighborsclassifier)：

回答分为两个方面：

1. 指出为什么cosine相似度没有在sklearn包中：

   cosine相似度在两个向量完全一样时的输出结果是1，在它们完全相反时结果是-1，而这严格上并不能算作衡量指标，其它如欧几里得距离，向量相似度越高距离越小即越接近于0. （不过为什么通过1-cos近似），因此不能使用knn的加速结构来加快运算。

2. 给出了解决方法：

   1. 可以自行实现cos相似度并作为函数参数传进去，代价就是不能使用knn中用于加速的结构，只能使用暴力计算。
   2. 第二个方案比较巧妙，通过深入到公式的转换把计算cosine相似度转换成用归一化之后的欧几里得即可。 

![](http://ot1c7ttzm.bkt.clouddn.com/cosine.png)

看完之后笔者被那个公式转换惊叹到，而后果断地采用了这个方案，在计算出文档向量之后，额外做一次归一化，之后只需要正常传入knn，距离函数用默认的欧几里得距离即可。

### 论文细节

论文中的实验部分有这么一句话：

> To represent test documents with category-specific schemes, e.g., tf·chi, tf·ig, tf·eccd, tf·rf and iqf·qf·icf, we adopt a popular method in previous studies hat assigning the maximum value among |C| estimated weights to each term in test documents.

鉴于笔者缺乏大量阅读英文文档的经验，再加上当时没有考虑到实现，所以阅读论文的时候始终不明白这句话的意思，但后来在实现话中提到的 tf-chi、tf-ig等方法时程序频繁报出词典key error，而后想起这句似乎关键但并不太明白什么意思的话，再结合实现时候的问题，终于明白了。

权重计算方法如 tf-idf 分为两个部分，一个是 tf ，由一个词在一篇文章内出现的频率得到，训练集和测试集均要计算，用python代码表示即是一个两层的词典  

```python
tf[document][word] = frequency[document][word] / doclength[document]
```

另一个是 idf ，由一个词在语料库中出现的文档数和文档总数计算得到，对于训练集是需要计算得到的，对于测试集则相当于权重词典，要用的时候直接查就行，而它的表现形式在tf-idf中是一个单层的词典，idf只计算每个词，和词在哪个类中没有关系。

```python
tf_idf[document][word] = tf[document][word] * idf[word]
```

而对于如chi等词，它们词权重计算不仅和词相关，还和类别相关。意思就是每个词的权重在不同类是不一样的，用代码表示出来即是：

```python
tf_chi[document][word] = tf[document][word] * chi[label][word]
```

那么就有一个比较重要的问题：要是测试集的词在测试文档属于的类中不存在怎么办？那句话就给出了答案：若是词在所属类中不存在权重，那么就在其它类里面选择这个词权重最大的那个作为权重，用代码表示就是：

```python
if weights[labell].has_key(word):
	tf_chi[doc][word] *= weights[labell][word]
else:
	tf_chi[doc][word] *= max([ weights[x][word]  for x in weights if weights[x].has_key(word)])
```

由此，缺失的拼图找到了，笔者最终成功实现了这些权重计算方法。

### 衡量标准

根据论文原文，实验采用了两种衡量标准：MicroF1和MacroF1：

1. MicroF1就是一般说的准确率：预测正确的数目 / 测试样本总数

2. MacroF1 就是通常的F1的均值：

   1. $ MacroF1 = avg(F1)$ 

   2. $ F1(C) = \frac{2*precision*recall}{(precision + recall)}$ 

   3. $ precision = \frac{TP} { (TP + FN)} $： 正确预测为C的数目 / 预测为C的总数
      $ recall = \frac{ TP}{(TP + TN)}$：  正确预测为C的数目 / 真实为C的数目

   4. TP: true positive  属于C被分到C（预测正确）
      TN: true nagative  属于C被分到其它类（预测错误）

      FP: false positive 不属于C被正确分类（预测正确）
      FN: false nagative 不属于C被分到C（预测错误）

## 复现结果

下面是复现之后的结果：

笔者在未调参的SVM、KNN上，使用Reuters R8语料库的结果（KNN邻居数在1-35内选择结果最优的）：

![](http://ot1c7ttzm.bkt.clouddn.com/result.png)

![](http://ot1c7ttzm.bkt.clouddn.com/Result.png)

论文给出的最终结果：

![](http://ot1c7ttzm.bkt.clouddn.com/vsm_result.png)

笔者复现的在Reuters R8中KNN邻居数和MicroF1的关系：

![](http://ot1c7ttzm.bkt.clouddn.com/knn.png)

论文给出的关系图：

![](http://ot1c7ttzm.bkt.clouddn.com/vsm_knn.png)

可以看到，虽然数据上略微有差距，但经过在模型上的优化之后应该能够接近或达到论文给出的结果，提出的新的权重计算方式tf_dc和tf_bdc在Reuters R8上的表现还是不错的，不说能够傲视所有权重计算方式，起码表现足够优异，有一席之地。

## 小结

通过这次复现，笔者较为深入地学习了向量空间模型VSM，还了解了各种权重计算方法，谈到权重计算也不再只有单一的tf_idf了。此外，这么一个完整的，从数据到特征（虽然语料库预处理不是我做的），再到导入模型进行训练，再到预测，最后进行评估，这么个流程走下来之后，笔者对于机器学习的理解也加深了。光是从调包上讲也知道怎么用KNN和SVM，怎么做数据可视化了。

## 源代码

原论文、笔者实现过程的完整代码（包括训练模型、测试、评估、所有权重计算方法）、笔者实验得到的数据（MicroF1和MacroF1，knn各个邻居数上的MicroF1，可直接调用评估函数查看结果）都可以在这里看到：[我的github](https://github.com/zedom1/nlp/tree/master/VSM)