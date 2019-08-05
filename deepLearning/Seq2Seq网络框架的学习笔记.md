@[TOC](Seq2Seq网络架构)
# Seq2Seq网络架构模型
> 前期知识储备：RNN网络架构、LSTM网络架构、Word2Vec模型。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190804182231267.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
# Seq2Seq应用
* 机器翻译（谷歌翻译）
* 情感对话生成
* 代码补全（目前只处于概念阶段）
# Seq2Seq存在的问题
* 压缩损失了信息
* 长度限制
# 针对存在的问题，提出了Attention机制
* “高分辨率”聚焦再图片的某个特定区域并以“低分辨率”感知图像的周边区域的模式
* 通过大量事宴证明，将attention机制应用在机器翻译，摘要生成，阅读理解等问题上，取得的成效显著。
# Attention 机制（该机制能够运用到很多地方）
* 关注输入序列中某些状态下的内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190804235853883.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190805000120724.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
* 对Encoder层状态的加权，从而掌握输入语句中的所有细节信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190805000407780.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
* 加权效果（越黑α越低，越白α越高）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190805000942264.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
# Bucket机制
* 正常情况要对所有句子进行补全
* Bucket可以先分组，再计算
   
