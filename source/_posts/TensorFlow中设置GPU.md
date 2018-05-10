---

title: TensorFlow中设置GPU
date: 2016-10-29 10:26:36
tags: [TensorFlow,Deep Learning]
categories: TensorFlow

---

# 设置GPU个数

## 设置单个GPU或者CPU

如果没有事先指定GPU或者CPU，那么TensorFlow默认的机制是寻找在机器中找到的第一个GPU，**并且占有该GPU所有的显存资源**。如果没有找到GPU那么默认是利用第一个CPU来做运算。首先说一下TensorFlow对于设备的表示："/cpu:0"表示第一块CPU，"/gpu:0"表示第一块GPU，其他情况以此类推。我们可以用*nvidia-smi*来查看设备GPU信息。如下图所示：  

<img src="/images/nvidia-smi.png" width="90%" height="80%"/>

我们就用TensorFlow官网的例子作为示例：

```python
# Creates a graph.
with tf.device('/gpu:2'):
  a = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[2, 3], name='a')
  b = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[3, 2], name='b')
  c = tf.matmul(a, b)
# Creates a session with log_device_placement set to True.
sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
# Runs the op.
print sess.run(c)

```
还有一个方法使程序强制使用具体的设备。这是CUDA的参数设置：

```bash
CUDA_VISIBLE_DEVICES="1" python tf.py

```

## 使用多GPU

上面演示了我们如何为一些操作指定固定的GPU或者CPU。但是如果我们有多个GPU可以使用，而且想为一些运算指定多块GPU同时承担运算任务时，我们就可以用以下的方式来完成。

```python
# Creates a graph.
c = []
for d in ['/gpu:2', '/gpu:3']:
  with tf.device(d):
    a = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[2, 3])
    b = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[3, 2])
    c.append(tf.matmul(a, b))
with tf.device('/cpu:0'):
  sum = tf.add_n(c)
# Creates a session with log_device_placement set to True.
sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
# Runs the op.
print sess.run(sum)

```

# 设置GPU资源分配

前面说过TF会默认抓取当前使用GPU的所有显存。官网上解释说这样设计初衷就是为了提高计算效率。但是我们工作在多人使用的服务器上时就应该有所约束。官网提供了两种方式。第一种方式就是让TensorFlow在运行过程中动态申请显存，需要多少就申请多少。第二种方式就是限制GPU的使用率。
第一种方式如下：

```python
config = tf.ConfigProto()
config.gpu_options.allow_growth = True
session = tf.Session(config=config, ...)

```

第二种方式如下：

```python
config = tf.ConfigProto()
config.gpu_options.per_process_gpu_memory_fraction = 0.4
session = tf.Session(config=config, ...)

```

# 参考资料

+ [Using GPUs](https://www.tensorflow.org/how_tos/using_gpu/)
