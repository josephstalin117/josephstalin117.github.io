---
layout: post
title:  "tensorflow"
date:   2018-6-09 20:35:35 +0800
categories: command
---

测试tensorflow
```
import tensorflow as tf

hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```

tensorboard
```
tensorboard --logdir ./_logs
```

禁用开源驱动nouveau
```
sudo vim /etc/modprobe.d/blacklist.conf
# 添加以下两行
blacklist nouveau
options nouveau modeset=0

# 更新内核
sudo update-initramfs -u
sudo reboot

# 查看是否禁用成功
lsmod | grep nouveau
```

tfdbg
```
from tensorflow.python import debug as tf_debug

sess = tf_debug.LocalCLIDebugWrapperSession(sess)

python -m tensorflow.python.debug.examples.debug_mnist --debug
```

查看显卡占用
```
nvidia-smi -l

# 查看GPU是否可用
lspci | grep -i nvidia

# 卸载nvidia驱动
sudo nvidia-uninstall

其实只安装cuda即可，cuda自带nvidia驱动
```

查看CUDA版本信息
```
# 添加cuda环境变量
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

# 更新动态链接库
ldconfig /usr/local/cuda-10.0/lib64

nvcc --version
```

查看cuDNN版本信息
```
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
```

显存使用按需分配
```
# for tensorflow
gpu_options = tf.GPUOptions(allow_growth=True)
sess = tf.Session(config=tf.ConfigProto(gpu_options=gpu_options))

# for keras
import tensorflow as tf
import keras.backend.tensorflow_backend as KTF

config = tf.ConfigProto()
config.gpu_options.allow_growth = True
session = tf.Session(config=config)
KTF.set_session(session)
```

指定显卡
```
# 终端执行程序时设置使用的GPU
CUDA_VISIBLE_DEVICES=1 python my_script.py

# python代码中设置使用的GPU
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "2"
```

docker
```
# 测试nvdia docker运行情况
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi

# 测试GPU模式
docker run --runtime=nvidia -it --rm tensorflow/tensorflow:latest-gpu python -c "import tensorflow as tf; tf.enable_eager_execution(); print(tf.reduce_sum(tf.random_normal([1000, 1000])))"

# 进入docker bash 环境
docker run --runtime=nvidia -it tensorflow/tensorflow:latest-gpu bash
```

math
```
# 返回均值
tf.math.reduce_mean

# Creates a tensor with all elements set to 1.
# tf.ones_like
tensor = tf.constant([[1, 2, 3], [4, 5, 6]])
tf.ones_like(tensor)  # [[1, 1, 1], [1, 1, 1]]

# reshape
# tensor 't' is [1, 2, 3, 4, 5, 6, 7, 8, 9]
# tensor 't' has shape [9]
tf.reshape(t, [3, 3]) ==> [[1, 2, 3],
                        [4, 5, 6],
                        [7, 8, 9]]

# tensor 't' is [[[1, 1, 1],
#                 [2, 2, 2]],
#                [[3, 3, 3],
#                 [4, 4, 4]],
#                [[5, 5, 5],
#                 [6, 6, 6]]]
# tensor 't' has shape [3, 2, 3]
# pass '[-1]' to flatten 't'
reshape(t, [-1]) ==> [1, 1, 1, 2, 2, 2, 3, 3, 3, 4, 4, 4, 5, 5, 5, 6, 6, 6]

# Outputs random values from a truncated normal distribution.
# 产生一个截断的正态分布
tf.truncated_normal
```

tf.einsum()
```
tf.einsum(equation, `*inputs`,`**kwargs`)
# A generalized contraction between tensors of arbitrary dimension.
# C[i,k] = sum_j A[i,j] * B[j,k]
# Matrix multiplication
>>> einsum('ij,jk->ik', m0, m1)  # output[i,k] = sum_j m0[i,j] * m1[j, k]

# Dot product
>>> einsum('i,i->', u, v)  # output = sum_i u[i]*v[i]

# Outer product
>>> einsum('i,j->ij', u, v)  # output[i,j] = u[i]*v[j]

# Transpose
>>> einsum('ij->ji', m)  # output[j,i] = m[i,j]

# Batch matrix multiplication
>>> einsum('aij,ajk->aik', s, t)  # out[a,i,k] = sum_j s[a,i,j] * t[a, j, k]

```

查看变量
```
# 返回的是需要训练的变量列表
tf.trainable_variables

# 返回的是所有变量的列表
tf.all_variables
```

tf.layer
```
# build a model to classify the images in the MNIST dataset using the following CNN architecture:
def cnn_model_fn(features, labels, mode):
  """Model function for CNN."""
  # Input Layer
  input_layer = tf.reshape(features["x"], [-1, 28, 28, 1])

  # Convolutional Layer #1
  conv1 = tf.layers.conv2d(
      inputs=input_layer,
      filters=32,
      kernel_size=[5, 5],
      padding="same",
      activation=tf.nn.relu)

  # Pooling Layer #1
  pool1 = tf.layers.max_pooling2d(inputs=conv1, pool_size=[2, 2], strides=2)

  # Convolutional Layer #2 and Pooling Layer #2
  conv2 = tf.layers.conv2d(
      inputs=pool1,
      filters=64,
      kernel_size=[5, 5],
      padding="same",
      activation=tf.nn.relu)
  pool2 = tf.layers.max_pooling2d(inputs=conv2, pool_size=[2, 2], strides=2)

  # Dense Layer
  pool2_flat = tf.reshape(pool2, [-1, 7 * 7 * 64])
  dense = tf.layers.dense(inputs=pool2_flat, units=1024, activation=tf.nn.relu)
  dropout = tf.layers.dropout(
      inputs=dense, rate=0.4, training=mode == tf.estimator.ModeKeys.TRAIN)

  # Logits Layer
  logits = tf.layers.dense(inputs=dropout, units=10)

  predictions = {
      # Generate predictions (for PREDICT and EVAL mode)
      "classes": tf.argmax(input=logits, axis=1),
      # Add `softmax_tensor` to the graph. It is used for PREDICT and by the
      # `logging_hook`.
      "probabilities": tf.nn.softmax(logits, name="softmax_tensor")
  }

  if mode == tf.estimator.ModeKeys.PREDICT:
    return tf.estimator.EstimatorSpec(mode=mode, predictions=predictions)

  # Calculate Loss (for both TRAIN and EVAL modes)
  loss = tf.losses.sparse_softmax_cross_entropy(labels=labels, logits=logits)

  # Configure the Training Op (for TRAIN mode)
  if mode == tf.estimator.ModeKeys.TRAIN:
    optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.001)
    train_op = optimizer.minimize(
        loss=loss,
        global_step=tf.train.get_global_step())
    return tf.estimator.EstimatorSpec(mode=mode, loss=loss, train_op=train_op)

  # Add evaluation metrics (for EVAL mode)
  eval_metric_ops = {
      "accuracy": tf.metrics.accuracy(
          labels=labels, predictions=predictions["classes"])}
  return tf.estimator.EstimatorSpec(
      mode=mode, loss=loss, eval_metric_ops=eval_metric_ops)
```

tf.layers
```
# Functional interface for the 2D convolution layer.
layer1 = tf.layers.conv2d(inputs=inputs_img, filters=128, kernel_size=3, strides=2, padding='same')
```


tf.tile
```
# tile() 平铺，用于在同一维度上的复制
with tf.Graph().as_default():
    a = tf.constant([1,2],name='a') 
    b = tf.tile(a,[3])
    sess = tf.Session()
    print(sess.run(b))

# 对[1,2]的同一维度上复制3次，multiples参数维度与input维度应一致
[1 2 1 2 1 2]

a = 10*tf.random_normal([3,3,3,3])
c = tf.tile(a, [2,4,6,8], name=None)
# c.shape[6,12,18,24]
```

tf.split
```
# Splits a tensor into sub tensors.

# 'value' is a tensor with shape [5, 30]
# Split 'value' into 3 tensors with sizes [4, 15, 11] along dimension 1
split0, split1, split2 = tf.split(value, [4, 15, 11], 1)
tf.shape(split0)  # [5, 4]
tf.shape(split1)  # [5, 15]
tf.shape(split2)  # [5, 11]
# Split 'value' into 3 tensors along dimension 1
split0, split1, split2 = tf.split(value, num_or_size_splits=3, axis=1)
tf.shape(split0)  # [5, 10]
```

tf.concat
```
# Concatenates tensors along one dimension.
t1 = [[1, 2, 3], [4, 5, 6]]
t2 = [[7, 8, 9], [10, 11, 12]]
tf.concat([t1, t2], 0)  # [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]]
tf.concat([t1, t2], 1)  # [[1, 2, 3, 7, 8, 9], [4, 5, 6, 10, 11, 12]]

# tensor t3 with shape [2, 3]
# tensor t4 with shape [2, 3]
tf.shape(tf.concat([t3, t4], 0))  # [4, 3]
tf.shape(tf.concat([t3, t4], 1))  # [2, 6]
```

tf.expand_dims
```
Inserts a dimension of 1 into a tensor's shape. (deprecated arguments)
# 't' is a tensor of shape [2]
tf.shape(tf.expand_dims(t, 0))  # [1, 2]
tf.shape(tf.expand_dims(t, 1))  # [2, 1]
tf.shape(tf.expand_dims(t, -1))  # [2, 1]

# 't2' is a tensor of shape [2, 3, 5]
tf.shape(tf.expand_dims(t2, 0))  # [1, 2, 3, 5]
tf.shape(tf.expand_dims(t2, 2))  # [2, 3, 1, 5]
tf.shape(tf.expand_dims(t2, 3))  # [2, 3, 5, 1]
```

keras
```
from keras.models import Sequential

model = Sequential()
# model.add(Dense(32, input_dim=784))
model.add(Dense(units=64, input_shape=(train_X.shape[1], train_X.shape[2]), activation='relu'))
model.add(Dense(units=10, activation='relu'))
model.add(Dense(1))
model.compile(loss='mae', optimizer=keras.optimizers.SGD(lr=0.01, momentum=0.9, nesterov=True))
# show net structure
print(model.summary())

model.fit(x_train, y_train, epochs=5, batch_size=32)
# evaluate performance
loss_and_metrics = model.evaluate(x_test, y_test, batch_size=128)

# prediction new data
classes = model.predict(x_test, batch_size=128)
```
