---
layout: post
title: "tensorflow 文本序列检错的tricks(0)"
date: 2018-08-29
comments: true
tags: 
	- NLP
	- 文本纠错
	- tensorflow
---

这一大段时期，笔者一直在研究序列检错问题，最近涉及到字级序列的检错。字级序列进行错误检测其实就是一个和标注词性一样的序列标注问题，对于给定的字级序列，预测该序列中每个字是正确还是错误。在用tensorflow使用双向lstm对序列进行检错的过程中，笔者不断根据需要和模型反映的问题对模型进行修改，再加上参考其他人写的代码和总结，由此有了一些体会，在此说说其中几个。

本文介绍两个tricks：

1.  Sequence Mask
2.  Weighted Loss

<!-- more -->

### Sequence Mask

在获得文本语料之后，我们会将文本根据逗号、句号等标点符号切分成一个个句子序列，句子序列显然并不是固定长度的，然而在构造batch时需要所有的序列是同一长度才可以形成矩阵方便运算，由此需要设定一个最大长度，并将句子序列进行尾填充至最大长度。但在训练中，我们希望lstm只考虑非填充部分的序列，由此我们可以使用dynamic_rnn并将sequence length（sequence length的size为[batch_size, 1]）作为参数传进去，这样lstm会根据当前句子的length进行计算，由此提高了准确度和效率。

在lstm和全连接后，我们得到了句子序列每个字的概率值，将其和标准答案计算得到loss，但填充部分也同样有loss，我们不希望填充部分的loss被考虑进去，由此就需要sequence mask了，它以sequence length( size 为 [batch_size, ])和max length( size 为 [1,])为参数，构造出 [batch_size, max_length]的bool矩阵：

```python
from random import randint
import tensorflow as tf
from numpy import *

max_length = 3
batch_size = 5

sequence_length = array([randint(1,max_length) for _ in range(batch_size)])
loss_mask = tf.sequence_mask(tf.to_int32(sequence_length), tf.to_int32(max_length))

print(loss_mask)
print("sequence length: "+str(sequence_length))
with tf.Session() as sess:
	print("sequence mask:")
	print(sess.run(loss_mask))
```

![](http://ot1c7ttzm.bkt.clouddn.com/tf_trick_smask0.png)

可以看到，我们构造出了一个二维的bool矩阵，里面每行true的数目和sequence相应索引的数值是一样的，再将bool转换成float之后，我们就可以将它和loss相乘，把无关部分的loss遮挡去了。

```python
targets = array([[1 for _ in range(max_length)] for _ in range(batch_size)])
logits = array([[[randint(0,10)/10,randint(0,10)/10] for _ in range(max_length)] for _ in range(batch_size)])

targets = tf.convert_to_tensor(targets, dtype = tf.int32)
logits = tf.convert_to_tensor(logits, dtype = tf.float32)

loss_before_mask = tf.nn.sparse_softmax_cross_entropy_with_logits(logits=logits, labels=targets)
loss_mask = tf.sequence_mask(tf.to_int32(sequence_length), tf.to_int32(max_length))
loss_after_mask = loss_before_mask * tf.to_float(loss_mask)

with tf.Session() as sess:
  print("loss before mask:")
  print((loss_before_mask))
  print(sess.run(loss_before_mask))
  print("*"*20)
  print("loss after mask:")
  print((loss_after_mask))
  print(sess.run(loss_after_mask))
```

![](http://ot1c7ttzm.bkt.clouddn.com/tf_trick_smask1.png)

之后我们就可以将loss累加或取均值作为整体的cost并进行优化了，当然，我们也可以同样将mask用于序列的准确率计算上。



### Weighted Loss

在对每个字进行正确与否的二分类时，有一个很大的问题就是类别分布不均衡。显然，正确字的数目要远远大于不正确的字的数目。对于包含错误的句子，一个很长的句子中或许只有一两个字是错的，更别提全对的句子了。类别分布不均在其它分类问题上也很常见，更是由此引发了f1等评价指标的诞生。毕竟若只看正确率的话，一个将所有字都认为是正确的模型在以字为单位统计正确率时将会近似于1！当然，在以句子为单位，即只有模型正确诊断出错误点时整句才算一个正确样本时，正确率将低得伤人。

因此我们想要让模型更加重视样本数本身就少的类别，从目的上看，就是让模型更大胆地预测错误字，而不是胆小地把所有字都标记为正确，以此求得较小的loss。

既然模型只盯着优化最小Loss这个目标，并且更新参数也是以Loss作为基准，那么我们下手的地方自然就是loss了。核心思想很简单，就是将原本的loss再根据类别的权重进行加权并求和得到最终的cost，为了最小化带了权重的loss，模型会将权重也同样进行反向传播。因此，与权重大的类别有关的更新会更加得到重视，而模型也敢放开手脚大胆预测错误字了。

接下来我们通过一个简单的例子来看看Weighted loss 的使用：

首先，随机生成batch_size 为3的二分类one_hot 标签label，标签转换成索引的targets，随机生成模型的最终预测logits：

```python
import tensorflow as tf
from numpy import *
from random import randint

batch_size = 3

label = zeros([batch_size,2])
for i in range(batch_size):
	label[i][randint(0,1)]=1

targets = argmax(label, axis=1)
logits = array([[randint(0,10)/10,randint(0,10)/10] for _ in range(batch_size)])
```

随机生成的结果如下：

![](http://ot1c7ttzm.bkt.clouddn.com/tf_csc_trick0_result0.png)

而后，我们开始计算loss，将targets和logits作为参数传入损失函数中计算得到加权前的loss。

将标签和转置后的权重进行矩阵乘，得到加权后的标签矩阵，因标签是one hot，所以只有特定类别的权重才会保留下来。

将转置后的加权标签与loss进行点乘，每个loss乘以对应标签的权重，此时的loss是特定类别上的loss，如类0的loss，而后乘以类0的权重，由此得到加权后的loss。

```python
label = tf.convert_to_tensor(label, dtype=tf.float32)	# shape : [3,2]
targets = tf.convert_to_tensor(targets, dtype=tf.int32)	# shape : [3,]
logits = tf.convert_to_tensor(logits, dtype=tf.float32)	# shape : [3,2]
class_weight = tf.constant([1.0, 0.2], shape=[1,2], dtype=tf.float32)  # shape : [1,2]

loss_before_weighted = tf.nn.sparse_softmax_cross_entropy_with_logits(logits = logits, labels=targets)
# shape : [3,]
weighted_label = tf.transpose( tf.matmul(label, tf.transpose(class_weight)) )	
# shape : [1,3]
loss_after_weighted = tf.multiply(weighted_label, loss_before_weighted)	
# shape : [1,3]

with tf.Session() as sess:
	print("Class Weight:")
	print(sess.run(class_weight))
	print("\nWeighted Label:")
	print(sess.run(weighted_label))
	print("\nLoss before weighted:")
	print(sess.run(loss_before_weighted))
	print("\nLoss after weighted:")
	print(sess.run(loss_after_weighted))

```

![](http://ot1c7ttzm.bkt.clouddn.com/tf_csc_trick0_result1.png)

可以看到，加权前的loss分布较为正常，而在加权之后，类0的loss保持不变，而类1的loss为原来的0.2，在反向传播的时候，参数的更新也会参考权重，使得模型更注重类0的判断。

值得一提的是，显然类别的权重是一个超参数，一般可以根据样本数目比率得到，也可以人为设定而后观察并调整。

以上是在普通分类问题上的加权，我们同样可以将加权应用到文本序列中，即矩阵多了一维（max_length）：

```python
max_length = 2
batch_size = 3

label = array([[[0,0] for _ in range(max_length)] for _ in range(batch_size)])
for i in range(batch_size):
	for j in range(max_length):
		label[i][j][randint(0,1)]=1
targets = argmax(label, axis=2)
logits = array([[[randint(0,10)/10,randint(0,10)/10] for _ in range(max_length)] for _ in range(batch_size)])

print("label"+str(label))
print("targets"+str(targets))
print("logits"+str(logits))

label = tf.convert_to_tensor(label, dtype=tf.float32)
targets = tf.convert_to_tensor(targets, dtype=tf.int32)
logits = tf.convert_to_tensor(logits, dtype=tf.float32)
class_weight = tf.constant([1.0, 0.2], shape=[1,2], dtype=tf.float32)

loss_before_weighted = tf.nn.sparse_softmax_cross_entropy_with_logits(logits = logits, labels=targets)
# shape [batch_size, max_length]
weighted_label = tf.transpose( tf.matmul(tf.reshape(label,[-1,2]), tf.transpose(class_weight)) ) 
# shape [1, max_length*batch_size]
weighted_label = tf.reshape(weighted_label,[batch_size,max_length])
# shape [batch_size, max_length]
loss_after_weighted = tf.multiply(weighted_label, loss_before_weighted)
# shape [batch_size, max_length]

with tf.Session() as sess:
	print("Class Weight:")
	print(sess.run(class_weight))
	print("\nWeighted Label:")
	print(sess.run(weighted_label))
	print("\nLoss before weighted:")
	print(sess.run(loss_before_weighted))
	print("\nLoss after weighted:")
	print(sess.run(loss_after_weighted))

```

结果如下：

![](http://ot1c7ttzm.bkt.clouddn.com/tf_csc_trick0_result2.png)

![](http://ot1c7ttzm.bkt.clouddn.com/tf_csc_trick0_result3.png)







