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

tfdbg
```
from tensorflow.python import debug as tf_debug

sess = tf_debug.LocalCLIDebugWrapperSession(sess)

python -m tensorflow.python.debug.examples.debug_mnist --debug
```

查看显卡占用
```
nvidia-smi -l
```

查看CUDA版本信息
```
nvcc --version
```

查看cuDNN版本信息
```
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
```

显存使用按需分配
```
gpu_options = tf.GPUOptions(allow_growth=True)
sess = tf.Session(config=tf.ConfigProto(gpu_options=gpu_options))
```

指定显卡
```
# 终端执行程序时设置使用的GPU
CUDA_VISIBLE_DEVICES=1 python my_script.py

# python代码中设置使用的GPU
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "2"
```

