# hhblits: lightning-fast iterative protein sequence searching by hmm-hmm alignment       
HHblits：基于HMM-HMM比对的快速迭代蛋白质序列搜索

1. `HHBITS`: 基于HMM-HMM的快速迭代序列搜索
   - 与`PSI-BLAST`相比，速度更快，高出50%-100%的sensitivity，且准确率更高
   - 它扩展了 HHsearch 以实现快速、迭代的序列搜索。
2. Profile-profile 和 HMM-HMM是序列搜索方法中最敏感的两类。
3. HHblits 需要一个涵盖整个序列空间的 HMM 数据库，使用 kClust用于将大型序列数据库聚类到最大成对序列同一性的 20-30%，同时需要几乎全长的可比性（>80%更长的序列）。
4. 查询过程：
  - HHbits首先将查询序列（或MSA）转换为HMM，通常是通过添加在物理化学上与查询中的氨基酸相似的氨基酸的伪计数来实现的。
  - 然后搜索HHM数据库，将低于设定的期望值(E值)的HMM序列添加到查询MSA中，然后从这个MSA中构建下一次迭代的HMM。
  - 预滤波器：通过将每个HMM列中包含20个氨基酸概率的向量离散化为219个字母的字母表，实现作为序列到图谱的比较
5. ，HHblits的profile-profile对齐预过滤器将完整HMM-HMM对齐的数量从数百万减少到几千，使其比PSI-BLAST更快，但仍然像HHSearch一样敏感。
6. 查询速度方面：HHbits快于PSI-BlAST，快于HMMER3
7. HHblits的一个潜在缺陷是要求其数据库由MSAs及其hmm组成，而不是单个序列。


# 补充
1. `MSA`: 多序列比对。
   >把两个以上序列对齐，逐列比较其字符的异同，使得每一列的字符尽可能一致，以发现其共同的结构特征。
   - **系统发育分析**:用于描述同源序列之间的亲缘关系的远近，应用到分子演化分析中。是构建分子演化树的基础。
   - **功能分析**:用于描述一组序列之间的相似关系，以便了解一个基因家族的基本特征，寻找motif、保守区域等。用于预测新序列的二级和三级结构，进而推测其生物学功能。
   - **突变分析**:用于揭示不同个体的基因组由于突变而产生的差异。        
                 不同物种基因组范围的MSA能分析基因组结构变异和共线性。
   - **测序分析**:用于获得共性序列；用于序列拼接。

2. `HMM`: 隐马尔可夫模型。
   - 使用HMM模型时我们的问题一般有这两个特征：
     1. 我们的问题是基于序列的，比如时间序列，或者状态序列。
     2. 我们的问题中有两类数据，一类序列数据是可以观测到的，即观测序列；而另一类数据是不能观察到的，即隐藏状态序列，简称状态序列。