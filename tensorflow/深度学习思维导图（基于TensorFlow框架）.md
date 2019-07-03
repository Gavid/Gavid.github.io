@[toc]
# 深度学习

## 深度学习介绍

### 深度学习与机器学习的区别

- 特征提取方面
  - 机器学习：特征工程步骤是手动完成的，需要大量的专业领域知识
  - 深度学习：通常由多个层组成，他们通常将更简单的模型组合在一起，将数据从一层传递到另一层来构建更复杂的模型。通过大量数据自动得出模型，不需要人工特征提取环节
- 数据量和计算i性能要求方面
  - 机器学习：随着数据量的增加，模型性能会有瓶颈
  - 深度学习：随着数据量的增加，模型的性能会越来越好，但是深度学需要大量的算力
- 算法代表方面
  - 机器学习：朴素贝叶斯、决策树等
  - 深度学习：神经网络

### 深度学习的应用场景：

- 图像识别
- 自然语言处理
- 语音技术

### 深度学习框架介绍

- PyTorch和Torch 更适用于学术研究；TensorFlow、Caffe、Caffe2 更适用于工业界的生产环境部署
- Caffe 适用于处理静态图像； Torch和PyTorch 更适用于动态图像；TensorFlow 在两种情况下都很实用
- TensorFlow 和 Caffe 可在移动端使用

### TensorFlow 的特点

- 高度灵活
- 语言多样
- 设备支撑
- TensorFlow 可视化

### TensorFlow 的安装

- CPU 版本
- GPU 版本
- 两个版本的比较
  - CPU:诸葛亮
    - 综合能力比较强
    - 核芯的数量更少
    - 更适用于处理连续性（sequential）任务。
  - GPU:臭皮匠
    - 专做某一个事情很好
    - 核芯的数量更多
    - 更适用于并行（parallel）任务

## TensorFlow 框架介绍

### TF 数据流图

- TensorFlow结构分析
  - 构建图阶段
    - 流程图：定义数据（张量Tensor）和操作（节点Op）
  - 执行图阶段
    - 调用各方资源，将定义好的数据和操作运行起来
- 数据流图介绍
  - Tensor - 张量 - 数据
  - Flow - 流动

### 图与TensorBoard

- 什么是图结构
  - 数据（Tensor） + 操作（Operation）
- 图相关操作
  - 默认图
    - 查看默认图的方法
      - 调用方法
        - 用tf.get_default_graph()
      - 查看属性
        - .graph
  - 创建图
    - new_g = tf.Graph()
          with new_g.as_default():
              定义数据和操作
- TensorBoard:可视化学习
  - 数据序列化-events文件
    - tf.summary.FileWriter(path, graph=sess.graph)
  - 启动tensorboard
- OP
  - 数据：Tensor对象
    操作：Operation对象 - Op
  - 常见OP
    - 操作函数        &                           操作对象
      tf.constant(Tensor对象)           输入Tensor对象 -Const-输出 Tensor对象
      tf.add(Tensor对象1, Tensor对象2)   输入Tensor对象1, Tensor对象2 - Add对象 - 输出 Tensor对象3
  - 指令名称
    - 一张图 - 一个命名空间

### 会话

- tf.Session：用于完整的程序当中
  tf.InteractiveSession：用于交互式上下文中的TensorFlow ，例如shell
- 会话掌握资源，用完要回收 - 上下文管理器
- 初始化会话对象时的参数
  - graph=None
    target：如果将此参数留空（默认设置），
    会话将仅使用本地计算机中的设备。
    可以指定 grpc:// 网址，以便指定 TensorFlow 服务器的地址，
    这使得会话可以访问该服务器控制的计算机上的所有设备。
    config：此参数允许您指定一个 tf.ConfigProto
    以便控制会话的行为。例如，ConfigProto协议用于打印设备使用信息
- run(fetches,feed_dict=None)
- feed操作
  - a = tf.placeholder(tf.float32, shape=)
    b = tf.placeholder(tf.float32, shape=)

### 张量Tensor

- 张量(Tensor)
  - 张量 在计算机当中如何存储？
    - 标量 一个数字                 0阶张量
      向量 一维数组 [2, 3, 4]       1阶张量
      矩阵 二维数组 [[2, 3, 4],     2阶张量
                  [2, 3, 4]]
      ……
      张量 n维数组                  n阶张量
  - 张量的类型（dtype）
    - 创建张量的时候，如果不指定类型
      默认 tf.float32
          整型 tf.int32
          浮点型 tf.float32
  - 张量的阶（shape）
- 创建张量的指令
  - tf.zeros(shape=( * , * ))
  - tf.ones(shape = ( * , * ))
  - tf.canstant()
- 张量的变换
  - 类型的修改
    - 1）ndarray.astype(type)
      tf.cast(tensor, dtype = *)
      不会改变原始的tensor
      返回新的改变类型后的tensor
      2）ndarray.tostring()
  - 形状的修改
    - 静态形状 - 初始创建张量时的形状
      - 什么情况下才可以改变/更新静态形状？
        只有在形状没有完全固定下来的情况下
         tensor.set_shape(shape)
    - 如何改变动态形状
      - tf.reshape(tensor, shape)
        不会改变原始的tensor
        返回新的改变形状后的tensor
        动态创建新张量时，张量的元素个数必须匹配
  - 张量的数学运算
    - 查API（https://www.tensorflow.org/api_docs/python/tf/math）

### 变量OP

- 特点
  - 存储持久化
  - 可修改值
  - 可指定被训练
- 作用：存储模型参数
- 具体使用
  - 变量需要显式初始化，才能运行值
- 使用tf.variable_scope()修改变量的命名空间
  - 使得结构更加清晰

### 高级API

- 其他基础API
- 高级API
- 关于TensorFlow的API图示

### 案例：实现线性回归

- 线性回归原理复习
  - 1）构建模型
    y = w1x1 + w2x2 + …… + wnxn + b
    2）构造损失函数
        均方误差
    3）优化损失
        梯度下降
- 案例：实现线性回归的训练
  - 准备真实数据
        100样本
        x 特征值 形状 (100, 1)
        y_true 目标值 (100, 1)
        y_true = 0.8x + 0.7
  - 假定x 和 y 之间的关系 满足
    y = kx + b
    k ≈ 0.8 b ≈ 0.7
  - 流程分析：
    (100, 1) * (1, 1) = (100, 1)
    y_predict = x * weights(1, 1) + bias(1, 1)
    1）构建模型
    y_predict = tf.matmul(x, weights) + bias
    2）构造损失函数
    error = tf.reduce_mean(tf.square(y_predict - y_true))
    3）优化损失
    optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.01).minimize(error)
    5 学习率的设置、步数的设置与梯度爆炸
- 增加其他功能
  - 增加变量显示
    - 1）创建事件文件
    - 2）收集变量
    - 3）合并变量
    - 4）每次迭代运行一次合并变量
    - 5）每次迭代将summary对象写入事件文件
  - 增加命名空间
    - 使代码结构更加清晰，TensorBoard 图结构更清楚
  - 模型的保存与加载
    - saver = tf.train.Saver(var_list=None,max_to_keep=5)
          1）实例化Saver
          2）保存
              saver.save(sess, path)
          3）加载
              saver.restore(sess, path)
  - 命令行参数的使用
    - 使用方式
    - 步骤
      - 1）tf.app.flags
        tf.app.flags.DEFINE_integer("max_step", 0, "训练模型的步数")
        tf.app.flags.DEFINE_string("model_dir", " ", "模型保存的路径+模型名字")
        2）FLAGS = tf.app.flags.FLAGS
        通过FLAGS.max_step调用命令行中传过来的参数
        3、通过tf.app.run()启动main(argv)函数

## 数据读取、神经网络基础

### 文件读取流程

- 多线程 + 队列
- 文件读取流程
  - 1）构造文件名队列
    - file_queue = tf.train.string_input_producer(string_tensor,shuffle=True)
  - 2）读取与解码
    - 文本：
      读取：tf.TextLineReader()
      解码：tf.decode_csv()
      图片：
      读取：tf.WholeFileReader()
      解码：
        tf.image.decode_jpeg(contents)
        tf.image.decode_png(contents)
      二进制：
      读取：tf.FixedLengthRecordReader(record_bytes)
      解码：tf.decode_raw()
      TFRecords
      读取：tf.TFRecordReader()
      key, value = 读取器.read(file_queue)
      key：文件名
      value：一个样本
  - 3）批处理队列
    - tf.train.batch(tensors, batch_size, num_threads = 1, capacity = 32, name=None)
  - 手动开启线程
    - tf.train.QueueRunner()
      开启会话：
      tf.train.start_queue_runners(sess=None, coord=None)

### 图片数据

- 图像基本知识
  - 文本  特征词 -> 二维数组
    字典  one-hot -> 二维数组
    图片  像素值
  - 图片三要素
    - 长度、宽度、通道数
      - 黑白图、灰度图
            一个通道
      - 彩色图
            三个通道
  - TensorFlow中表示图片
    - Tensor对象
          指令名称、形状、类型
          shape = [height, width, channel]
  - 图片特征值处理
    - [samples, features]
          为什么要缩放图片到统一大小？
          1）每一个样本特征数量要一样多
          2）缩小图片的大小
          tf.image.resize_images(images, size)
  - 数据格式
    - 存储：uint8
      训练：float32
- 案例：狗图片读取
  - 1）构造文件名队列
    file_queue = tf.train.string_input_producer(string_tensor,shuffle=True)
    2）读取与解码
    读取：
        reader = tf.WholeFileReader()
        key, value = reader.read(file_queue)
    解码：
        image_decoded = tf.image.decode_jpeg(value)
    3）批处理队列
    image_decoded = tf.train.batch([image_decoded], 100, num_threads = 2, capacity=100)
    手动开启线程

### 二进制数据

- tensor对象
    shape:[height, width, channel] -> [32, 32, 3] [0, 1, 2] -> []
    [[32 * 32的二维数组],
    [32 * 32的二维数组],
    [32 * 32的二维数组]]
        --> [3, 32, 32] [channel, height, width] 三维数组的转置 [0, 1, 2] -> [1, 2, 0]
        [3, 2] -转置-> [2, 3]
    1）NHWC与NCHW
    T = transpose 转置

  3.3.2 CIFAR10 二进制数据读取
      流程分析：
          1）构造文件名队列
          2）读取与解码
          3）批处理队列
          开启会话
          手动开启线程

### TFRecords

- TFRecords 文件
  - 定义
    - TFRecords  其实是一种二进制文件，虽然它不如其他格式好理解，但是它能更好的利用内存，更方便复制和移动，并且不需要单独的标签文件
  - 使用步骤
- Example 结构解析
  - 使用步骤
- 读取TFRecords 文件API
  - 1）构造文件名队列
    2）读取和解码
        读取
        解析example
        feature = tf.parse_single_example(value, features={
        "image":tf.FixedLenFeature([], tf.string),
        "label":tf.FixedLenFeature([], tf.int64)
        })
        image = feature["image"]
        label = feature["label"]
        解码
        tf.decode_raw()
    3）构造批处理队列

### 神经网络基础

- 神经网络
  - 输入层
  - 隐藏层
  - 输出层
- 感知机
  - 可以解决 “ 与 ” 、 “ 或 ” 问题 ， 不能解决 “ 异或问题 ”

### 神经网络原理

- 逻辑回归
    y = w1x1 + w2x2 + …… + wnxn + b
    sigmoid -> [0, 1] -> 二分类问题
    损失函数：对数似然损失
  用神经网络进行分类
      假设函数
          y_predict =
          softmax - 多分类问题
      构造损失函数
          loss = 交叉熵损失
      优化损失
          梯度下降
      3.6.1 softmax回归 - 多分类问题
          假设要进行三分类
          2.3, 4.1, 5.6
      3.6.2 交叉熵损失

### 案例：Mnist手写数字识别

- Mnist 数据获取API

### 线性神经网络局限性

## 卷积神经网络

### 卷积神经网络简介

- 与传统多层神经网络对比
- 发展历史
- 卷积网络在ImageNet比赛错误率

### 卷积神经网络原理

- 结构（隐藏层）
  - 卷积层
    通过在原始图像上平移来提取特征
    激活层
    增加非线性分割能力
    池化层（pooling layer）/下采样（subsample）
    减少学习的参数，降低网络的复杂度（最大池化和平均池化）
    全连接层
- 卷积层
  - 每层卷积层都是由若干卷积单元（卷积核）组成，每个卷积单元的参数都是通过反向传播算法最佳化得到的
  - 卷积运算的目的是特征提取，第一层卷积层可能只能提取一些低级的特征，如边缘、线条和角等层级，更多层的网络能从低级特征中迭代提取更复杂的特征
  - 卷积核（filter/过滤器/模型参数/卷积单元）
    - 四大要素
      - 卷积核个数
      - 卷积核大小
        - 通常设置为：1*1、3*3、5*5
      - 卷积核步长
      - 卷积核零填充大小
  - 总结 -  输出大小计算公式
  - 卷积网络API
- 激活函数
  - ReLU = max(0,x)
    - 函数图像
    - 效果图
  - 采用新的激活函数的原因
  - 激活函数API
- 池化层
  - 主要作用
    - 特征提取，减少参数数量
  - 主要方法
    - max_pooling : 取池化窗口的最大值
    - avg_pooling：取池化窗口的平均值
  - 利用了图像上像素点之间的联系
  - 池化层API
- 全连接层
  - 主要作用
    - “ 分类器 ”
    
# 思维导图图片形式
![深度学习思维导图](https://img-blog.csdnimg.cn/20190317191030994.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
