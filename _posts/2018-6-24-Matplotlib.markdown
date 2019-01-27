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

画图
```
# 简单的绘图
x = np.linspace(0, 2 * np.pi, 50)
# 设定图像的大小
plt.figure(figsize=(10,5))
# 如果没有第一个参数 x，图形的 x 坐标默认为数组的索引
plt.plot(x, np.sin(x))
plt.show() # 显示图形
```

调整label字体大小
```
plt.xlabel("time",fontsize=20)
plt.ylabel("DO",fontsize=20)
plt.xticks(fontsize=20)
plt.yticks(fontsize=20)

# 调整全部字体大小
matplotlib.rcParams.update({'font.size': 22})
```

调整线条颜色
```
x = range(len(data))
# 调整图像大小
plt.figure(figsize=(10,5))

# 绘制曲线
plt.plot(x, y1, label="actual")
plt.plot(x, y2, color='r',label="prediction")

# 设置坐标轴范围
plt.xlim((-5,5))
plt.ylim((-2,2))

#设置坐标轴刻度
plt.xticks(np.arange(-5, 5, 0.5))
plt.yticks(np.arange(-2, 2, 0.3))

# plt.legend()添加标注
plt.legend(loc="upper left")
# 自动调整标注位置
plt.legend(loc="best")

plt.title("上证50指数历史最高价、收盘价走势折线图")
plt.xlabel(Time Slot Count")
plt.ylabel("value")

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

柱状图
```
import matplotlib.pyplot as plt

data = [5, 20, 15, 25, 10]

plt.bar(range(len(data)), data)
plt.show()

# 设置柱体颜色
plt.bar(range(len(data)), data, fc='g')

# 设置不同的柱体颜色
plt.bar(range(len(data)), data, color='rgb') # or `color=['r', 'g', 'b']`

# 设置label
data = [5, 20, 15, 25, 10]
labels = ['Tom', 'Dick', 'Harry', 'Slim', 'Jim']

plt.bar(range(len(data)), data, tick_label=labels)
plt.show()
```

