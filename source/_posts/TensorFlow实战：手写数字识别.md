---

title: TensorFlow实战：手写数字识别
date: 2016-08-09 22:49:29
tags: [TensorFlow,Deep Learning]
categories: TensorFlow
descriptions: 手写数字识别是一个非常经典的机器学习问题。很多新的模型提出都会在这个测试集上做实验。本文就是用TensorFlow实现多种模型来解决手写数字识别。旨在给出TensorFlow的基本用法。

---

# 手写数字识别概述

手写数字识别问题是一个颇为经典的机器学习问题，很多机器学习模型的提出都会在这个数据集上做实验。手写数字如下图所示：
![](/images/c1s0-2.png)
每张都是一个28*28像素点的灰度图片，每个像素点的值都是介于0~1之间的灰度值。用非机器学习算法解决这个问题是极其困难的，也很难写一些规则去解决这个问题。机器学习的很多分类模型都能很好的解决这个问题，例如SVM。利用SVM的默认参数就能达到90%的准确率，而用精心设计参数的SVM能够达到98%~99%左右的准确率。而本文的主要目的是展示几种思路，主要的重点放在tensorflow的简单实战，并不是全面的机器学习教程。关于手写数字数据集(`MNIST`)可以详见colah的博客[Visualizing MNIST: An Exploration of Dimensionality Reduction](http://colah.github.io/posts/2014-10-Visualizing-MNIST/)。本教程用到的代码可在[Github](https://github.com/hpzhao/tensorflow_examples)上下载。

**说明**：这份实例代码版本比较陈旧，而且不是很规范，之后会更新规范版本的。  

# Softmax线性回归

我们简单将手写数字`28x28`的像素值直接每一行收尾连接成一个784维度的向量。当然你可能会说这将会损失一些结构的信息。我们在这个模型不会利用结构信息，而在接下来的CNN模型我们会利用结构信息。利用softxmax回归解决这个问题的思路如下：
<image src='/images/softmax-regression-scalargraph.png' width="60%" height="100%" />
## 准备数据
我们利用tensorflow自带的处理mnist数据的函数。这个函数会判断当前目录是否有MNIST_data，如果没有会在线下载。我们读取之后用one-hot的形式表示10个数字，例如用[0,1,0,0,0,0,0,0,0,0]来表示1。
```python
import tensorflow as tf
import tensorflow.examples.tutorials.mnist.input_data as input_data
mnist = input_data.read_data_sets('MNIST_data',one_hot=True)

```

## 搭建模型
接下来就是构建计算图。

```python
'''
我们首先定义x和对应的正确标签y_。这里我们使用的是占位符，这样我们就能够动态扩展维度，并且只有在赋值时才真正申请空间。
None这一维是用来存一个batch的样本个数的。
'''
x = tf.placeholder(tf.float32,[None,784])
y_ = tf.placeholder(tf.float32,[None,10])


#这里我们定义了两个变量来存储权重和偏置

W = tf.Variable(tf.zeros([784,10]))
b = tf.Variable(tf.zeros([10]))

#输入经过简单的加权之后计算softmax的值作为每个种类的概率
y = tf.nn.softmax(tf.matmul(x,W) + b)

#接来是定义交叉熵代价函数，并申请了一个学习率为0.01的SGD的优化方法来学习参数
cross_entropy = -tf.reduce_sum(y_*tf.log(y))
optimizer = tf.train.GradientDescentOptimizer(0.01)
train = optimizer.minimize(cross_entropy)

#下面是计算准确率
correct_prediction = tf.equal(tf.argmax(y,1),tf.argmax(y_,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction,tf.float32))

```
## 模型训练

```python
with tf.Session() as sess:
    #初始化所有变量
    sess.run(tf.initialize_all_variables())
    #迭代1000轮
    for _ in range(1000):
        #batch大小为50
        batch_xs,batch_ys = mnist.train.next_batch(50)
        sess.run(train,feed_dict={x:batch_xs,y_:batch_ys})
    #计算测试集准确率
    print accuracy.eval(feed_dict={x:mnist.test.images,y_:mnist.test.labels})

```

最后得到的准确率大概在91%左右。

# 全连接神经网络
```python

#!/usr/bin/env python
#coding:utf8
import tensorflow as tf
import tensorflow.examples.tutorials.mnist.input_data as input_data
import math

def weight_variable(shape):
    return tf.Variable(tf.truncated_normal(shape=shape,stddev = 1.0/math.sqrt(float(28*28))))
def bias_variable(shape):
    return tf.Variable(tf.zeros(shape=shape))
def mnist_mlp():
    mnist = input_data.read_data_sets('MNIST_data')

    x = tf.placeholder(tf.float32,[None,784])
    y_ = tf.placeholder(tf.int32,[None])

    #first layer
    W_fc1 = weight_variable([784,128])
    b_fc1 = bias_variable([128])

    h_1 = tf.nn.relu(tf.matmul(x,W_fc1) + b_fc1)

    #second layer
    W_fc2 = weight_variable([128,32])
    b_fc2 = bias_variable([32])

    h_2 = tf.nn.relu(tf.matmul(h_1,W_fc2) + b_fc2)

    #linear

    W_fc3 = weight_variable([32,10])
    b_fc3 = bias_variable([10])

    logits = tf.matmul(h_2,W_fc3) + b_fc3

    #loss
    cross_entropy = tf.nn.sparse_softmax_cross_entropy_with_logits(logits,y_)
    loss = tf.reduce_mean(cross_entropy)

    #train
    train = tf.train.GradientDescentOptimizer(0.01).minimize(loss)

    #eval
    correct_prediction = tf.nn.in_top_k(logits,y_,1)
    accuracy = tf.reduce_mean(tf.cast(correct_prediction,tf.float32))

    with tf.Session() as sess:
        sess.run(tf.initialize_all_variables())
        for step in xrange(100000):
            batch = mnist.train.next_batch(50)
            if step%100 == 0:
                print accuracy.eval(feed_dict={x:mnist.test.images,y_:mnist.test.labels})
            train.run(feed_dict={x:batch[0],y_:batch[1]})

if __name__=='__main__':
    mnist_mlp()

```
# 循环神经网络
```python
#!/usr/bin/env python
#coding:utf8
from __future__ import print_function

import tensorflow as tf
import numpy as np
from tensorflow.examples.tutorials.mnist import input_data

flags = tf.app.flags
FLAGS = flag.FLAGS
FLAGS.DEFINE_integer('batch_size',128,'batch size')
# Import MNIST data
mnist = input_data.read_data_sets("../../data/MNIST", one_hot=True)

'''
To classify images using a recurrent neural network, we consider every image
row as a sequence of pixels. Because MNIST image shape is 28*28px, we will then
handle 28 sequences of 28 steps for every sample.
'''

# Parameters
learning_rate = 0.001
training_iters = 100000
batch_size = 128
display_step = 10

# Network Parameters
n_input = 28 # MNIST data input (img shape: 28*28)
n_steps = 28 # timesteps
n_hidden = 128 # hidden layer num of features
n_classes = 10 # MNIST total classes (0-9 digits)

# tf Graph input
x = tf.placeholder("float", [None, n_steps, n_input])
y = tf.placeholder("float", [None, n_classes])

# Define weights
weights = {
    'out': tf.Variable(tf.random_normal([n_hidden, n_classes]))
}
biases = {
    'out': tf.Variable(tf.random_normal([n_classes]))
}


def RNN(x, weights, biases):

    # Prepare data shape to match `rnn` function requirements
    # Current data input shape: (batch_size, n_steps, n_input)
    # Required shape: 'n_steps' tensors list of shape (batch_size, n_input)

    # Permuting batch_size and n_steps
    x = tf.transpose(x, [1, 0, 2])
    # Reshaping to (n_steps*batch_size, n_input)
    x = tf.reshape(x, [-1, n_input])
    # Split to get a list of 'n_steps' tensors of shape (batch_size, n_input)
    x = tf.split(0, n_steps, x)

    # Define a lstm cell with tensorflow
    lstm_cell = tf.nn.rnn_cell.BasicLSTMCell(n_hidden, state_is_tuple=True)

    # Get lstm cell output
    outputs, states = tf.nn.rnn(lstm_cell, x, dtype=tf.float32)

    # Linear activation, using rnn inner loop last output
    return tf.matmul(outputs[-1], weights['out']) + biases['out']

pred = RNN(x, weights, biases)

# Define loss and optimizer
cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(pred, y))
optimizer = tf.train.AdamOptimizer(learning_rate=learning_rate).minimize(cost)

# Evaluate model
correct_pred = tf.equal(tf.argmax(pred,1), tf.argmax(y,1))
accuracy = tf.reduce_mean(tf.cast(correct_pred, tf.float32))

# Initializing the variables
init = tf.initialize_all_variables()

# Launch the graph
config = tf.ConfigProto()
config.gpu_options.allow_growth=True
with tf.Session(config=config) as sess:
    sess.run(init)
    step = 1
    # Keep training until reach max iterations
    while step * batch_size < training_iters:
        batch_x, batch_y = mnist.train.next_batch(batch_size)
        # Reshape data to get 28 seq of 28 elements
        batch_x = batch_x.reshape((batch_size, n_steps, n_input))
        # Run optimization op (backprop)
        sess.run(optimizer, feed_dict={x: batch_x, y: batch_y})
        step += 1
    # Calculate accuracy for 128 mnist test images
    test_len = 10000
    test_data = mnist.test.images[:test_len].reshape((-1, n_steps, n_input))
    test_label = mnist.test.labels[:test_len]
    print("Testing Accuracy:", sess.run(accuracy, feed_dict={x: test_data, y: test_label}))
```

# 卷积神经网络

```python
#!/usr/bin/env python
#coding:utf8
import tensorflow as tf
import tensorflow.examples.tutorials.mnist.input_data as input_data
def weight_variable(shape):
    initial = tf.truncated_normal(shape,stddev=0.1)
    return tf.Variable(initial)
def bias_variable(shape):
    initial = tf.constant(0.1,shape=shape)
    return tf.Variable(initial)
def conv2d(x,W):
    return tf.nn.conv2d(x,W,strides=[1,1,1,1],padding='SAME')
def max_pool_2x2(x):
    return tf.nn.max_pool(x,ksize=[1,2,2,1],
            strides=[1,2,2,1],padding='SAME')
def mnist_cnn():
    mnist = input_data.read_data_sets('MNIST_data',one_hot=True)
    #first_layer
    x = tf.placeholder(tf.float32,[None,784])
    y_ = tf.placeholder(tf.float32,[None,10])

    W_conv1 = weight_variable([5,5,1,32])
    b_conv1 = bias_variable([32])

    
    x_images =  tf.reshape(x,[-1,28,28,1])

    h_conv1 = tf.nn.relu(conv2d(x_images,W_conv1) + b_conv1)
    h_pool1 = max_pool_2x2(h_conv1)

    #second_layer
    W_conv2 = weight_variable([5,5,32,64])
    b_conv2 = bias_variable([64])

    h_conv2 = tf.nn.relu(conv2d(h_pool1,W_conv2) + b_conv2)
    h_pool2 = max_pool_2x2(h_conv2)

    #full_connected_layer
    W_fc1 = weight_variable([7*7*64,1024])
    b_fc1 = bias_variable([1024])

    h_pool2_flat = tf.reshape(h_pool2,[-1,7*7*64])
    h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat,W_fc1) + b_fc1)
    
    #dropout
    keep_prob = tf.placeholder(tf.float32)
    h_fc1_drop = tf.nn.dropout(h_fc1,keep_prob)

    #output_layer
    W_fc2 = weight_variable([1024, 10])
    b_fc2 = bias_variable([10])

    y_conv=tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2) + b_fc2)
    #train &  eval
    cross_entropy = -tf.reduce_sum(y_*tf.log(y_conv))
    train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
    correct_prediction = tf.equal(tf.argmax(y_conv,1), tf.argmax(y_,1))
    accuracy = tf.reduce_mean(tf.cast(correct_prediction, "float"))
    with tf.Session() as sess:
        sess.run(tf.initialize_all_variables())
        for i in range(20000):
            batch = mnist.train.next_batch(50)
            if i%1000 == 0:
                print accuracy.eval(feed_dict={x:mnist.test.images, y_:mnist.test.labels, keep_prob: 1.0})
            train_step.run(feed_dict={x: batch[0], y_: batch[1], keep_prob: 0.5})

if __name__=='__main__':
    mnist_cnn()

```  

