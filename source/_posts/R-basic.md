---
title: R basic
date: 2018-10-21 19:50:55
tags:
 - 其他编程语言
categories: 其他编程语言
---


## 矩阵索引
``` R
# 下标都是从0开始
A=matrix(1:16, 4, 4)
A
# 矩阵下标
A[2,3]
# 不包含
A[-c(1,3)] # 不包含第一行和第三行
A[,-c(2)] # 不包含第二行
# 矩阵维度
dim(A)
```

## 常用命令
### 赋值
``` R
x <- c(1, 3, 2,5)
# c是连接的意思，括号中的数被连在一起
# 也可以用等号
x = c(1, 6, 2)
```

### 长度
```R
x = c(1, 6, 2)
length(x)
```

### 删除varaibles
``` R
rm(x)   # 删除x
rm(list=ls())   # 
```

### 查看variables
``` R
ls()
```

### 矩阵
``` R
#matrix()    # 创建一个矩阵
matrix(data=c(1,2,3,4), nrow=2, ncol=2,byrow=TRUE) # byrow=TRUE表示先行后列
```

### 转置
``` R
a = matrix(c(1,2,3,4), 4, 4)
at = t(a)
```

### 幂和开方
``` R
x^3
poly(x, 4)
sqrt(x)
```

### 正态分布
``` R
# 默认的均值和方差是0,1
x = rnorm(n=10, mean=50, std=0.1)
y = x + rnorm(50)
cor(x,y) # 相关系数
```

### 均匀分布
``` R
x = runif(100, -100, 100)
```

### 随机数种子
``` R
set.seed(1303)
# 设置完随机数之后，产生的随机数都是可以复现的
```

### 均值和方差，标准差
``` R
set.seed(3)
y=rnorm(100)
mean(y)
var(y)
sd(y)
```

### seq创建序列
```
# seq(a,b)
a= seq(1, 3, 1)
3:11 = seq(3, 11)
```

## 绘图
### plot二维
``` R
x = rnorm(100)
y = rnorm(100)
plot(x, y)
plot(x, y, xlab="x-axis", ylab="y-axis", main="x-y")
# 保存在主目录下
pdf("Figure.pdf")
plot(x,y, col="green")
dev.off()
```

### contour等高线和image heatmap，persp()三维图
``` R
x = seq(-pi,pi)
y = x
f = outer(x, y, function(x,y)cos(y)/(1+x^2))
contour(x, y, f)
fa = (f - t(f)) /2
contour(x, y, fa)
image(x, y, fa)
persp(x, y, fa)
```

## 加载数据 
``` R
# 1.header=TRUE 表示第一行是变量名
# na.string表示缺失符号
auto_data = read.table("Auto.data", header = T, na.strings = "?")

# 2.使用fix查看数据
fix(auto_data)

# 3. 查看数据维度
dim(auto_data)


# 4.忽略不规则数据
na.omit(auto_data)
dim(auto_data)

# 5.查看变量名字
names(auto_data)
```

## 其他操作
``` R
auto_data = read.table("Auto.data", header = T, na.strings = "?")
auto_data = na.omit(auto_data)
names(auto_data)

# 1. 访问dataset中的变量
plot(auto_data$cylinders, auto_data$mpg)
# or
attach(auto_data)
plot(cylinders, mpg)

# 2. as.factor() and boxplot
cylinders = as.factor(cylinders)
plot(cylinders, mpg)

# 3.绘制直方图
hist(mpg)

# 4. pairs()绘制任意两个变量之间的散点图矩阵
pairs(auto_data)
pairs(~mpg+weight, auto_data)

# 5. 识别图中鼠标点击的点的坐标
plot(horsepower, mpg)
identify(horsepower, mpg, name)

# 6.summary()给出指定变量的描述信息
summary(auto_data)
```

## 库
### 安装package
``` R
install.packages("ISLR")
```

### 加载package
``` 
library(ISLR)
```

## 分支

### for循环
``` 
for (val in sequence)
{
statement
}

# 示例
x <- c(2,5,3,9,8,11,6)
count <- 0
for (val in x) {
if(val %% 2 == 0)  count = count+1
}
print(count)

```

### while 循环
```
while (TRUE)
{
    print("hello")
}
```

## if 语句
```
if (3 > 2 && 2 < 3):
    print(
```

## 函数
```
rand_interval = function(min, max, number)
{
  sd = (max.numeric()-min.numeric())/3
  count = 0
  res = 1:number
  while(TRUE)
  {
    x = rnorm(1, mean=0,sd=sd)
    if(x >= min && x <= max)
    {
      count = count+1
      res[count] = x
    }
  }
  return (res)
}

min = -100
max = 100
number = 20
x1 = rand_interval(min, max, number)

```

## 类型转换
### 字符转数值
```
as.numeric(str(3))
```

## 写入文件
```
library("xlsx")
data = matrix = c()
write.xlsx(data, file = "myworkbook.xlsx", append = FALSE)
```


##

qqplot
ggplot2
lm





## 参考文献
1.https://www.datamentor.io/r-programming/for-loop/
