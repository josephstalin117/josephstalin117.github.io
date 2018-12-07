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

散点图
```
N = 50
x = np.random.rand(N)
y = np.random.rand(N)
colors = np.random.rand(N)
area = (30 * np.random.rand(N))**2  # 0 to 15 point radii

plt.scatter(x, y, s=area, c=colors, alpha=0.5)
plt.show()
```

subplots
```
#First create some toy data:
x = np.linspace(0, 2*np.pi, 400)
y = np.sin(x**2)

#Creates just a figure and only one subplot
#fig 是整体图形，ax是分别的图形
fig, ax = plt.subplots()
ax.plot(x, y)
ax.set_title('Simple plot')

#Creates two subplots and unpacks the output array immediately
f, (ax1, ax2) = plt.subplots(1, 2, sharey=True)
ax1.plot(x, y)
ax1.set_title('Sharing Y axis')
ax2.scatter(x, y)

#Creates four polar axes, and accesses them through the returned array
fig, axes = plt.subplots(2, 2, subplot_kw=dict(polar=True))
axes[0, 0].plot(x, y)
axes[1, 1].scatter(x, y)

#Share a X axis with each column of subplots
plt.subplots(2, 2, sharex='col')

#Share a Y axis with each row of subplots
plt.subplots(2, 2, sharey='row')

#Share both X and Y axes with all subplots
plt.subplots(2, 2, sharex='all', sharey='all')
```

fig and axarr
```
f, axarr = plt.subplots(2, sharex=True)
f.suptitle('Sharing X axis')
axarr[0].plot(x, y)
axarr[1].scatter(x, y)
```

![images](/source/subplots.png)

spines
```
an axis spine -- the line noting the data area boundaries

ax = fig.add_subplot(2, 2, 4)
ax.set_title('spines at data (1, 2)')
ax.plot(x, y)
ax.spines['left'].set_position(('data', 1))
ax.spines['right'].set_color('none')
ax.spines['bottom'].set_position(('data', 2))
ax.spines['top'].set_color('none')
ax.spines['left'].set_smart_bounds(True)
ax.spines['bottom'].set_smart_bounds(True)
ax.xaxis.set_ticks_position('bottom')
ax.yaxis.set_ticks_position('left')
```

![images](/source/spines.png)

