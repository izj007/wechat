#  Transformer在图的应用——睡前小论文

原创 knight  [ JDArmy ](javascript:void\(0\);)

**JDArmy** ![]()

微信号 gh_e9df6be6f498

功能介绍 一个专注于高质量安全研究 、渗透测试、漏洞挖掘的分享平台，点关注，上高速！！！

____

___发表于_

收录于合集

#睡前小论文 2 个

#机器学习 2 个

# 《Do Transformers Really Perform Bad for Graph Representation?》

    从本章开始一个新的系列——睡前小论文。此系列主要是一些最近阅读的AI、大模型和网络安全相关的论文。

## 1、阅读原因

    在网络安全领域，有许多和图相关的工具。如域渗透工具bloodhound，将域内的信息保存为图结构，方便后续通过图内保存的结构化信息或者图相关的算法帮助使用者寻找通往域管的路径。白盒代码审计工具tabby，将代码属性图保存为图结构，方便通过图结构对代码进行查询或者通过图结构进行污点分析。随着现在大模型的发展，能否通过大模型与图结构相结合来提高域渗透或者白盒代码审计的自动化能力呢？本文主要是将transform应用于图数据挖掘领域，并通过实验证明transform在图领域的应用优于传统的GNN。

## 2、资源地址

论文地址：https://arxiv.org/pdf/2106.05234.pdf

github地址：https://github.com/Microsoft/Graphormer

大模型地址：https://huggingface.co/clefourrier/graphormer-base-pcqm4mv2

## 3、《Transformers真的在图表达方面表现不好吗？》

### （1）论文主要思想

  * 提出了 **中心性编码** (Centrality Encoding)来将节点的度中心编码进模型，从而让模型学习到节点的入度和出度相关信息。在代码属性图中则表示方法的调用和被调用关系，映射到代码就是这个方法的代码段信息和这个方法出现在哪些方法里面。

数学表达：

![]()  

  * 提出了 **空间编码** (Spatial Encoding)来将节点在整空间中的关系信息编码进模型，让模型学习到节点和整个空间的关系。在代码属性图则表示这个方法和整体代码的关系。

![]()  

  * 提出了 **边编码** (Edge Encoding)来将边和整个空间相关的信息也编码进模型，让模型学习到边和整个空间之间的关系。在代码属性图中，则表示两个节点之间的调用链。

![]()

下面是Graphormer的结构图，左边表示transform模型，右边是三种编码方式。

![]()

### （2）实现效果

下图是Graphormer和其他图深度学习模型的对比。（train MAE训练平均绝对误差，validate MAE 验证平均绝对误差）

![]()

下图是在不同数据集下，图预测相关的性能（AP表示平均精准度，AUC表示ROC曲线下坐标轴围成的面积，AUC越接近1.0，检测方法真实性越高;等于0.5时，则真实性最低）

![]()

## 4、总结

    本文通过三种编码方式，将图中节点的度信息，节点和整个空间的信息，两个节点之间的信息都编码进了transform模型，从而让模型达到更好的效果。但是对于其二次方复杂度的限制，很难应用于大型图结构，特别是针对代码属性图这种有数十万节点的图。但可以考虑提取其中的子图来进行训练。后面会试试实际效果，也欢迎大家来进行交流。

![]()

预览时标签不可点

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

