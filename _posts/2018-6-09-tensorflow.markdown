---
layout: post
title:  "tensorflow"
date:   2018-5-16 20:35:35 +0800
categories: command
---

测试tensorflow
```
import tensorflow as tf

hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```
