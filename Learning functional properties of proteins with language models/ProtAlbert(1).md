#### **ProtAlbert**

**数据集**：Uniref100

 - 数据的处理：
   1. 将单个氨基酸作为输入单词/token，因此蛋白质数据库所包含的token比NLP中使用的与语料库多几个数量级。
     2. Uniref50、UniRef100和BFD在每个标记之间使用单个空格（指示词边界）进行标记。
     3. 每个蛋白质序列存在单独的行中，相当于句子。
     3. 在每个蛋白质序列之间插入以个空行，以表示"文档结尾"

**模型**：Albert

**模型参数**：使用官方Github Albert库的参数。相较于原始版本，在相同的硬件配置上训练模型的全局`batch size`从4096增加到了10753

**训练方法**：首先在最大长度为512的序列上进行150k步计算，然后在最大长度为2k的长度上进行150k步计算

**特征提取概述**：如何使用ProtTrans模型在氨基酸或整个蛋白质水平上为任意蛋白质序列提取特征

1. 首先，对任一个蛋白质序列"SEQ"进行标记(tokenization)并添加位置编码
2. 生成的向量通过ProtTrans模型来为每个输入标记(氨基酸)创建上下文感知嵌入。我们使用Transformer注意堆栈的最后一个隐藏状态作为下游预测方法的输入。
3. 这些嵌入(来自2)可以作为单个token级别的预测任务的输入。eg：CNN——预测氨基酸的二级结构
4. 这些嵌入(来自2)也可以沿着长度维度进行连接和池化，来获得固定大小的蛋白质嵌入，生成的蛋白质嵌入可用于预测蛋白质当面的输入。eg：FNN——用于蛋白质的细胞定位

​	![特征提取概述](.\images\PFP\ProtTrans\特征提取概述.png)



#### **AlBert**

**模型**：采用了 Transformer 以及 GELU 激活函数，与Bert的结构基本一样

**调整**：

- embedding 层参数因式分解

- 跨层参数共享

- 将 NSP 任务改为 SOP 任务

  > 前两个改进之处是为了减少模型的参数量，最后一个改进是因为之前有许多工作已经验证了NSP任务对于Bert并没有什么太大的积极影响

1. embedding 层参数因式分解

     Bert模型中的Embedding Size 与 Hiden Size 是相等的，这就造成了一个问题，因为E和H是绑定的，当H增大时，E也相应的增大，最终导致参数数量呈平方级的增加。所以作者在这里将E和H进行解绑，具体操作是在embedding后加入一个矩阵来进行维度变换，这样就可以在E保持不变的情况下去调整H的大小，从而把参数量从O ( V × H )降低到了O ( V × E + E × H ) ，当E<<H时参数量减少的很明显。

   ![img](https://s1.ax1x.com/2020/08/18/du74L6.png#shadow)

   由上图可知，当不进行参数共享时，随着E的增大，模型的性能也在不断地提升；当共享参数时，E并不是越大越好，同时，参数共享可能会带来性能的略微下降。

2. 参数共享

   传统Transfrmer的每一层参数都是独立的，这就导致当模型的层数增加时，其参数量也在明显上升。作者尝试将所有的参数进行共享，相当于只学习第一层的参数，并在剩下所有层重用第一层的参数，导致多层的Attention层其实就是一层Attention的叠加，进而减少了模型的参数量。

   ![img](https://s1.ax1x.com/2020/08/18/duW7m4.png#shadow)

   由上图可知，使用参数共享可以有效的提升模型稳定。

   ![img](https://s1.ax1x.com/2020/08/18/duhvLD.png#shadow)

   由上图可知，共享参数对于模型参数量减少的影响是可观的，模型的性能下降<3%。其中，共享FFN层参数对参数数量和性能影响较大，共享Attention层的参数影响较小。

3. SOP任务

   > NSP任务：预测下一个句子（二分类），RoBERTa 和 XLNet 这样的论文已经阐明了 NSP 的无效性，并且发现它对下游任务的影响是不可靠的
   >
   > - 正例：一个文档里面连续的两句话
   > - 反例：不同文档里面的两句话
   >
   >  正反例的选择导致了NSP任务中包含了主题预测，主题预测相较于预测两句话的连续性较为简单

​		AlBert提出了SOP（Sentence-Order Prediciton）任务，关键思想是：

		- 正样本： 同一个文档中取两个连续的句子
		- 负样本：交换这两个句子的顺序

![img](https://s1.ax1x.com/2020/08/18/duo7nK.png#shadow)

![‘](https://s1.ax1x.com/2020/08/18/duT1N4.png#shadow)

由上图可知，SOP 提高了下游多种任务（SQUAD，MNLI，SST-2，RACE）的表现。

1. NSP不能很好的预测SOP任务
2. SOP能够很好的预测NSP任务
3. SOP在下游任务中表现优于NSP和None

**实验**：

![img](https://s1.ax1x.com/2020/08/18/dKm8mt.png#shadow)

![img](https://s1.ax1x.com/2020/08/18/dKeoFS.png#shadow)

![image-20220914135423502](C:\Users\WMGray\AppData\Roaming\Typora\typora-user-images\image-20220914135423502.png)

Speedup 是训练时间而不是 Inference 时间。

1. 在相同的训练时间下，ALBERT 得到的效果确实比 BERT 好
2. 在相同的 Inference 时间下，ALBERT base 和 large 的效果都没有 BERT 好，而且差了 2-3 个点，作者在最后也提到了会继续寻找提高速度的方法（Sparse attention 和 Block attention）



**参考**：

1. [ALBERT 详解](https://wmathor.com/index.php/archives/1480/)
2. [ALBERT 论文解读](https://zhuanlan.zhihu.com/p/88099919)
3. [Google ALBERT原理讲解](https://zhuanlan.zhihu.com/p/84273154)
4. [[BERT系列之详解ALBERT](https://segmentfault.com/a/1190000022397993)]
5. [一文揭开ALBERT的神秘面纱](https://blog.csdn.net/u012526436/article/details/101924049)

