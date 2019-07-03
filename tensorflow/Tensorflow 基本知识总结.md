@[toc]
# 简单事例入门
```
# 导入tensorflow
import tensorflow as tf
# 定义常量
hello = tf.constant('Hello, TensorFlow!')
# 定义session
sess = tf.Session()
# 运行
print(sess.run(hello))
sess.close()
```
# 计算图

```
# 定义两个常量节点
node1 = tf.constant(3.0, dtype=tf.float32,name='x')
node2 = tf.constant(4.0) # also tf.float32 implicitly
# 定义一个加法操作
node3 = tf.add(node1, node2)
```
# Session 会话

```
# session打开就要关闭，需要使用sess.close()，这边为了后面的使用没有close
sess = tf.Session()
print(sess.run([node1, node2]))
```
# placeholder

```
a = tf.placeholder(tf.float32)
b = tf.placeholder(tf.float32)
adder_node = a + b  # + provides a shortcut for tf.add(a, b)
```
# variable

```
W = tf.Variable([.3], dtype=tf.float32)
b = tf.Variable([-.3], dtype=tf.float32)
x = tf.placeholder(tf.float32)
linear_model = W*x + b
```
# tf.matmul(a, b)
Multiplies matrix `a` by matrix `b`, producing `a` * `b`

```
# 创建数组a,b
a = np.array([[0,0],[2,3]],dtype=np.float32)
b = np.array([[3,4],[5,6]],dtype=np.float32)
print(a)
print(b)
# 计算数组a*b的值
x = tf.matmul(a,b)
sess = tf.Session()
sess.run(x)
```
# tf.sigmoid(x)
* y = 1 / (1 + exp(-x))

```
output = tf.sigmoid(x)
sess.run(output)
```
# tf.nn.sigmoid_cross_entropy_with_logits(labels=z, logits=x)
Computes sigmoid cross entropy given 'x'
* x = logits, z = labels
* z * -log(sigmoid(x)) + (1 - z) * -log(1 - sigmoid(x))

```
labels = np.array([0,1,0,1],dtype=np.float32).reshape(2,2)
# print(labels)
loss = tf.nn.sigmoid_cross_entropy_with_logits(labels=labels, logits=x)
sess.run(loss)
```
# tf.reduce_mean(input_tensor)
* 计算均值
Computes the mean of elements across dimensions of a tensor

```
loss2 = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(labels=labels, logits=x))
loss3 = tf.reduce_mean([1.,2.,3.,4.])
sess.run(loss2)
```
# tf.greater(x,y)
* 如果x>y 则返回true
Returns the truth value of (x > y) element

```
g = tf.greater(output,0.8)
sess.run(g)
```
# tf.cast(x, dtype)
* 将布尔值类型转化为float32
Casts a tensor to a new type

```
cast = tf.cast(g, dtype=tf.float32)
sess.run(cast)
```
# tf.train.GradientDescentOptimizer(learning_rate).minimize(loss)
使用梯度下降进行优化loss
# 一般程序结构

```
# 定义一个计算图
graph = tf.Graph()

# 建立计算图
with graph.as_default():
    x = tf.placeholder(tf.float32)
    # y用于输入真实值
    y = tf.placeholder(tf.float32)
    W = tf.Variable([.3], dtype=tf.float32)
    b = tf.Variable([-.3], dtype=tf.float32)
    linear_model = W*x + b
    # 计算预测值与真实值差的平方
    squared_deltas = tf.square(linear_model - y)
    # 计算均值
    loss = tf.reduce_mean(squared_deltas)
    # 梯度下降优化器，参数为学习率
    optimizer = tf.train.GradientDescentOptimizer(0.01)
    train = optimizer.minimize(loss)
    
# 执行计算图
with tf.Session(graph=graph) as sess:
    # 变量执行之前必须进行初始化
    init = tf.global_variables_initializer()
    sess.run(init) # reset values to incorrect defaults.
    for i in range(4000):
        sess.run(train, {x: [1, 2, 3, 4], y: [0, -1, -2, -3]})
        if i%100 == 0:
            print(sess.run([loss,W, b],{x: [1, 2, 3, 4], y: [0, -1, -2, -3]}))
    print('train over')
    print(sess.run([W, b]))
```

