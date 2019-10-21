---
title: python cv2.imresize图像缩放
date: 2019-05-09 21:37:30
tags:
 - python
 - opencv
categories: python
---

## cv2.resize
cv2是python的opencv包，实现的功能是对一个图片进行缩放。
python3下安装命令：
~$:pip install opencv-python

### 示例代码
``` python
import cv2
import numpy as np
img = np.random.rand(210, 160 ,3)
print(img.shape)
img_scale = cv2.resize(img, (84, 84))
print(img_scale.shape)
```

## 参考文献

