# Rosetta: Large Scale System for Text Detection and Recognition in Images（大规模图像文本提取和识别系统）

### 摘要

​		在本文中，我们提出了一个可部署、可扩展的光学字符识别 (OCR) 系统，称之为 Rosetta，用于处理 Facebook 上每天上传的图片。对于 Facebook 这样社交网络中的互联网用户而言，通过图像内容共享实现对图像及其包含文字的理解，已经成为信息沟通的一种主要方式，这对促进搜索和推荐应用来说也是至关重要的。这里， 我们提出 Rosetta 系统结构，这是一种有效的建模技术用于检测和识别图像中的文本。通过进行大量的评估实验，我们解释了这种实用系统是如何用于构建 OCR 系统，以及如何在系统的开发期间部署特定的组分。

### 简介

​		OCR 系统分为文本检测和文本识别两个阶段：基于 Faster-RCNN 模型，在文本检测阶段我们的系统能够检测出图像内包含文本的区域；采用基于全卷积网络的字符识别模型，在文本识别阶段我们的系统能够处理检测到的位置并识别出文本的内容。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128210949390.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)

### 文本检测模型

​		文本检测阶段，我们采用最先进的 Faster-RCNN 目标检测网络。简而言之，Faster-RCNN 通过一个全卷积神经网络和区域建议网络 (RPN) 同时实现目标的检测和识别：学习表征一张图像的卷积特征映射并生成 k 个高可能性的文本建议区域候选框及其置信度得分，随后按置信度分数排序这些候选框并利用非极大值抑制 (NMS) 算法得到最有希望的检测区域，再从候选框中提取相关的特征映射并学习一个分类器来识别它们。此外，边界框回归 (bounding-box regression) 通常用于提高边界框生成的准确性。

​		考虑到模型效率的问题，我们的文本检测模型采用基于 ShuffleNet 结构的 Faster-RCNN 模型，而 ShuffleNet 卷积结构是在 ImageNet 数据集上经过预训练得到的。整个文本检测系统是以监督式的，端到端的方式进行训练的。训练过程中，该检测系统采用内部合成的数据进行训练，并在 COCO-Text 数据集上进行微调后应用于学习真实世界数据集特征。

### 文本识别模型

​		文本识别阶段，我们尝试了以下两种不同的模型结构，并采用了不同的文本损失函数。

​		基于字符序列的编码模型 (CHAR)。该模型假设所有图像都具有相同的大小并且存在最大可识别字符数量 k。对于较长的单词，单词中只有 k 个字符能够被识别出。该 CHAR 模型的主体由一系列卷积结构组成，后接上 k 个独立的多类分类器，用于预测在每个位置上出现的字符。在训练期间，共同学习卷积体和 k 个不同的分类器。使用 k 个并行损失 (softmax + negative cross-entropy) 并提供合理的基线就能很容易地训练 CHAR 模型，但这有两个重大缺点：它无法正确识别长的单词串 (如 URL 地址)，分类器中大量的参数容易导致模型出现过拟合现象。
​		基于全卷积模型。我们将此模型称为 CTC，因为它使用 seq2seq 的CTC损失函数用于模型的训练，并输出一系列字符。CTC 模型的结构示意图如下图3所示，基于 ResNet-18 结构，在最后一层的卷积中预测输入字符在每个图像中最可能的位置。与其他工作不同的是，我们在此不使用显式循环神经网络结构 (如 LSTM 或 GRU) 或任何的注意力机制，而直接生成每个字符的概率。训练时，我们采用 CTC 损失函数，通过边缘化所有可能对齐的路径集合来计算给定标签的条件概率，这就能够使用动态编程进行有效地计算。 如图3所示，特征映射的每一列对应于图像每个位置所有字符的概率分布，CTC 能够找到它们之间的对齐预测，即可能包含重复的字符或空白字符和真实标签。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128211027859.png)

### Rosetta 系统

​		下图展示了 Rosetta 的系统结构，其在线图片处理的流程主要包含以下几个步骤：

​		Rosetta 将客户端的图片下载到本地计算机集群，并通过预处理步骤，如调整大小和规范化来进一步处理。
​		执行文本检测模型 (图中的步骤4) 获取图像中所有单词的位置信息 (边界框坐标和置信度分数)。
​		将单词的位置信息传递给文本识别模型 (图中的步骤5)，用于提取图像给定裁剪区域的单词字符。
​		所提取的文本信息及图像中文本的位置信息都被存储在 TAO 中，这是 Facebook 的一个分布式图形数据库 (图中的步骤6)。
​		诸如图片搜索等下游应用程序可以从 TAO 中访问所提取的图像文本信息 (图中的步骤7)。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128211050185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)

### 实验

​		我们对 Rosetta OCR 系统进行了大量的评估实验。首先，我们定义用于评估准确性和系统处理时间的度量，并描述用于训练和评估的数据集。我们在单独的数据集上进行保准的模型训练和评估过程。进一步，我们评估文本检测和文本识别模型，以及系统准确性和运行时间之间的权衡。

### 评估度量

​		对于文本检测模型，我们采用 mAP 和 IoU 作为评估度量。而对于文本识别模型，我们使用 accuracy 和 Levenshtein’s edit distance 作为我们的评估指标。

### 数据库

​		我们采用 COCO-Text 数据集对我们的模型进行训练和测试。COCO-Text 数据集包含大量自然场景下注释的文字，由超过63000张图片和145000文本实例组成。为了解决 COCO-Text 数据与 Facebook 上图片数据分布不匹配的问题，我们还通过随机重叠 Facebook 中图像的文本来生成了一个大规模的合成数据集。

### 模型检测性能

​		下表1，表2，表3分别展示了 Faster-RCNN 检测模型在不同数据测试集上的的检测性能，不同卷积主体结构的推理时间，以及 ResNet-18 和 ShuffleNet 为卷积主体的检测性能。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128211125856.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)

> 表1 在不同数据测试集上 Faster-RCNN 检测模型的mAP。准确性是 mAP 在合成训练数据集上的相对改进。→表示微调，即 A→B 表示在 A 上训练并在 B 上微调。﻿

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128211138507.png)

> 表2 以各种卷积结构为主体的 Faster-RCNN 模型的推理时间。表中的数字为相对于 ResNet-50 的改进。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128211157316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)

> 表3 使用 ResNet-18 和 Shuffle 结构的 Faster R-CNN 在 COCO-Text 数据集上评估结果。表格中的 mAP 是对 ResNet-18 的 3个RPN 宽高比的相对改进。

### 模型识别性能

​		下表4，表5分别展示了在不同数据集上模型的识别性能以及结合检测和识别系统检测到的词召回率下降的归一化幅度。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128211216581.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)

> 表4不同数据集上模型的识别性能。越高的 accuracy 和越低的 edit distance 代表越好的结果。表中的数字是相对于在合成数据集上训练的 CHAR 模型的改进。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128211232893.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)

> 表5 检测和识别组合系统检测到词召回率下降的归一化幅度

### 结论

​		**本文，我们提出了鲁棒而有效的文本检测和识别模型，并用于构建可扩展的 OCR 系统 Rosetta。**我们对 Rosetta 系统进行了大量的评估，结果展示系统在权衡模型精度和处理时间方面都能实现高效率的性能。进一步地，我们的系统将部署到实际生产中并用于处理 Facebook 用户每天上传的图片。

