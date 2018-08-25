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

