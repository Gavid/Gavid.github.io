# 利用Tensorflow实现多项式回归

```
import tensorflow as tf
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline
import pickle
# 定义 生成器
def gen_batch(data):
    for i in range(len(data) // 64):
        pos = 64 * i
        data__ = data[pos : pos + 64]
        x = np.reshape(data__[:, 0], (-1, 1))
        y = np.reshape(data__[:, 1], (-1, 1))
        yield x,y
# 定义 计算图        
graph = tf.Graph();
# 创建 计算图
with graph.as_default():
	# 此处的数据形式为： [[1,2,0],[2,1,0],[5,6,0]...[6,9,1]]
	#数据为前两维为点坐标，第三维为数据标签
    with open('./poly.pkl', 'rb') as f:
        x, y = pickle.load(f)
    # 绘制曲线
    # plt.scatter(x, y)
    # plt.show()
    # 对数据进行预处理
    whole = np.array([x, y]).T
    # 打乱数据顺序，进行随机梯度下降
    # 111np.random.shuffle(whole)
    train = whole[:-64]
    test = whole[-64:]
    
    a = tf.Variable([1],dtype=tf.float32)
    b = tf.Variable([2],dtype=tf.float32)
    c = tf.Variable([3],dtype=tf.float32)
    d = tf.Variable([4],dtype=tf.float32)
    e = tf.Variable([5],dtype=tf.float32)
    x_t = tf.placeholder(shape=[None, 1], dtype=tf.float32, name='x')
    y_t = tf.placeholder(shape=[None, 1], dtype=tf.float32, name='y')
    linear_model = a+b*x_t+c*tf.pow(x_t,2)+d*tf.pow(x_t,3)+e*tf.pow(x_t,4)
    # 计算预测值与真实值差的平方
    squared_deltas = tf.square(linear_model - y_t)
    # 计算均值
    loss = tf.reduce_mean(squared_deltas)
    # 梯度下降优化器，参数为学习率
    optimizer = tf.train.GradientDescentOptimizer(0.00001)
    train_t = optimizer.minimize(loss)
# 运行 计算图
f_res = []
with tf.Session(graph=graph) as sess:
    init = tf.global_variables_initializer();
    sess.run(init)
    step = 0
    for i in range(100):
        for x_,y_ in gen_batch(train):
            t,_ = sess.run([loss,train_t], {x_t: x_, y_t: y_})
            f_res.append(t);
            step+=1
            if step%100 == 0:
                # print("step:{},TrainLoss{:>10.4f}".format(step,t))
                pass
    print('train over')
    print(sess.run([a, b,c,d,e]))
# 绘制 数据曲线
plt.plot(f_res)
plt.show()
```

