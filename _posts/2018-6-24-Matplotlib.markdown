---
layout: post
title:  "Matplotlib"
date:   2018-6-24 20:35:35 +0800
categories: python
---

导入

```
import matplotlib.pyplot as plt
import numpy as np
```

```
# 简单的绘图
x = np.linspace(0, 2 * np.pi, 50)
# 如果没有第一个参数 x，图形的 x 坐标默认为数组的索引
plt.plot(x, np.sin(x))
plt.show() # 显示图形
```