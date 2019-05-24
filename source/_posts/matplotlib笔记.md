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
[代码地址]()
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
[代码地址]()
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
[代码地址]()
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
[代码地址]()
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
[代码地址]()
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
获得当前figure的坐标轴，可以用来绘制。

### 代码示例
[代码地址]()
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
[代码地址]()
``` ppython
import matplotlib.pyplot as plt
import numpy as np

plt.figure()
plt.ion()
x = np.arange(20)
for i in range(20):
  # x[i] = x[i] + i 
  y = pow(x[:i], 2)
  temp = x[:i]*100
  print(temp)
  plt.plot(temp, y, linewidth=1)
  plt.pause(0.1)

plt.ioff()
plt.show() 
```
