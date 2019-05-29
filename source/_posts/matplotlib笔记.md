---
title: matplotlib笔记
date: 2019-03-21 15:29:17
tags:
 - python
 - matplotlib
categories: python
mathajax: true
---

## show()
### 介绍
show()函数是一个阻塞函数，调用该函数，显示当前已经绘制的图像，然后需要手动关闭打开的图像，程序才会继续执行。

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/matplotlib/1_show.py)
``` python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(0,10,1)

y1 = x**2
y2 = 2*x +5

plt.plot(x,y1)
plt.savefig("0_1.png")
plt.show()  # 调用show()会阻塞，然后关掉打开的图片，程序继续执行

plt.plot(x,y2)
plt.show()
```

## savefig()
### 介绍
该文件接收一个参数，作为文件保存的路径。

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/matplotlib/2_savefig.py)
``` python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(0, 10, 1)

y1 = x**2
y2 = 2*x +5

plt.plot(x,y1)
plt.savefig("2.png") # 保存图像，名字为2.png
plt.show()
```

## figure()
### 介绍
figure()函数相当于生成一张画布。如果不显示调用的话，所有的图像都会绘制在默认的画布上。可以通过调用figure()函数将函数图像分开。figure()会接受几个参数，num是生成图片的序号，figsize指定图片的大小。

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/matplotlib/3_figure.py)
``` python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(0,10,1)

y1 = x**2
y2 = 2*x +5

# figure
plt.figure()
plt.plot(x,y1)

plt.figure(num=6,figsize=(10,10))
plt.plot(x,y2)
plt.show()
```

## imshow()
### 介绍
该函数用来显示图像，接受一个图像矩阵。调用完该函数之后还需要调用show()函数。

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/matplotlib/4_image.py)
``` python
import matplotlib.pyplot as plt
import numpy as np

img = np.random.randint(0, 255, [32, 32])
print(img.shape)

plt.imshow(img)
plt.show()
```

## subplot()
### 介绍
绘制$m\times n$个子图

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/matplotlib/5_subplot.py)
``` python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(0, 10, 1)
y1 = 2 * x
y2 = 3 * x
y3 = 4 * x
y4 = 5 * x

plt.figure()

plt.subplot(2, 2, 1)
plt.plot(x, y1, marker='s', lw=4)

plt.subplot(2, 2, 2)
plt.plot(x, y2, ls='-.')

plt.subplot(2, 2, 3)
plt.plot(x, y3, color='r')

plt.subplot(2, 2, 4)
plt.plot(x, y4, ms=10, marker='o')

plt.show()
```

## subplots()
### 介绍
将一张图分成$m\times n$个子图。

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/matplotlib/6_subplots.py)
``` python
import matplotlib.pyplot as plt
import numpy as np


figure,axes = plt.subplots(2, 3, figsize=[40,20])
axes = axes.flatten()

x = np.arange(0, 20) 
y1 = pow(x, 2)
axes[0].plot(x, y1) 

y5 = pow(x, 3)
axes[5].plot(x, y5) 

plt.show()
```

## ax()
### 介绍
获得当前figure的坐标轴，用来绘制。

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/matplotlib/7_axes.py)
``` python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(-3.5,3.5,0.5)
y1 = np.abs(2 * x)
y2 = np.abs(x)

plt.figure(figsize=(10,10))
ax = plt.gca() # gca = get current axis
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('red')
ax.xaxis.set_ticks_position("bottom")
ax.yaxis.set_ticks_position("left")
ax.spines['bottom'].set_position(('data',0))
ax.spines['left'].set_position(('data',0))

# both work
ax.plot(x,y1,lw=2,marker='-',ms=8)
plt.plot(x,y2,lw=3,marker='^',ms=10)

# xlim and ylim
# ax.xlim([-3.8, 3.3])
# AttributeError: 'AxesSubplot' object has no attribute 'xlim'
plt.xlim([-3.8, 3.3])
plt.ylim([0, 7.2])

# xlabel and ylabel
# ax.xlabel('x',fontsize=20)
# AttributeError: 'AxesSubplot' object has no attribute 'xlabel'
plt.xlabel('x',fontsize=20)
plt.ylabel('y = 2x ')

# xticklabel and yticaklabel
# ax.xticks(x,('a','b','c','d','e','f','g','h','i','j','k','l','m','n'),fontsize=20)
# AttributeError: 'AxesSubplot' object has no attribute 'xticks'
plt.xticks(x,('a','b','c','d','e','f','g','h','i','j','k','l','m','n'),fontsize=20)

# both work
ax.legend(['t1','t2'])
plt.legend(['y1','y2'])

plt.show()
```

## ion()和ioff()
### 介绍
交互式绘图，可以在一张图上不断的更新。

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/matplotlib/8_plt_ion_ioff.py)
``` ppython
import matplotlib.pyplot as plt
import numpy as np

count = 1
flag = True

plt.figure()
ax = plt.gca()
x = np.arange(20)
plt.figure()
ax2 = plt.gca()

while flag:
    plt.ion()
    y = pow(x[:count], 2)
    temp = x[:count]
    ax.plot(temp, y, linewidth=1)
    plt.pause(1)
    plt.ioff()

    ax2.plot(x, x+count)
    count += 1
    if count > 20:
        break

plt.show() 
```

## seanborn

### 介绍
对matplotlib进行了一层封装

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/matplotlib/9_seanborn.py)
``` python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

values = np.zeros((21,21), dtype=np.int)
fig, axes = plt.subplots(2, 3, figsize=(40,20))
plt.subplots_adjust(wspace=0.1, hspace=0.2)
axes = axes.flatten()

# cmap is the paramter to specify color type, ax is the parameter to specify where to show the picture
# np.flipud(matrix), flip the column in the up/down direction, rows are preserved
figure = sns.heatmap(np.flipud(values), cmap="YlGnBu", ax=axes[0])
figure.set_xlabel("cars at second location", fontsize=30)
figure.set_title("policy", fontsize=30)
figure.set_ylabel("cars at first location", fontsize=30)
figure.set_yticks(list(reversed(range(21))))

figure = sns.heatmap(np.flipud(values), ax=axes[1])
figure.set_ylabel("cars at first location", fontsize=30)
figure.set_yticks(list(reversed(range(21))))
figure.set_title("policy", fontsize=30)
figure.set_xlabel("cars at second location", fontsize=30)

plt.savefig("hello.pdf")
plt.show()
plt.close()
```


