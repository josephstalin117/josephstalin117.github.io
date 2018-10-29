---
layout: post
title:  "Matplotlib"
date:   2018-6-24 20:35:35 +0800
categories: command
---

导入

```
import matplotlib.pyplot as plt
import numpy as np
```

```
# 简单的绘图
x = np.linspace(0, 2 * np.pi, 50)
# 设定图像的大小
plt.figure(figsize=(10,5))
# 如果没有第一个参数 x，图形的 x 坐标默认为数组的索引
plt.plot(x, np.sin(x))
plt.show() # 显示图形
```

调整线条颜色
```
x = range(len(data))
plt.figure(figsize=(10,5))
plt.plot(x,data['close'],label="actual")
plt.plot(x,data['high'],color='r',label="prediction")
# plt.legend()添加图例
plt.legend(loc="upper left")

plt.title("上证50指数历史最高价、收盘价走势折线图")
plt.xlabel("时间")
plt.ylabel("指数")

plt.show()
```