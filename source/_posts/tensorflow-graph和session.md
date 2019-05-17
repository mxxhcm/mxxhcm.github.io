---
title: tensorflow graph和session
date: 2019-05-12 21:45:04
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.Graph
tf.Graph包含两类信息：
- Node和Edge，用来表示各个op如何进行组合。
- collections。使用tf.add_to_collection和tf.get_collection对collection进行操作。一个常见的例子是创建tf.Variable的时候，默认会将它加入到"global variables"和"trainable variables" collection中。
当调用tf.train.Saver和tf.train.Optimizer的时候，它会使用这些collection中的变量作为默认参数。
常见的定义在tf.GraphKeys上的collection:
VARIABLES, TRAINABLE_VARIABLES, MOVING_AVERAGE_VARIABLES, LOCAL_VARIABLES, MODEL_VARIABLE,SUMMARIES.
[关于collections的详细介绍可点击这里](https://mxxhcm.github.io/2019/05/13/tensorflow-collection/)

## 构建tf.Graph
调用tensorflow API就会构建新的tf.Operation和tf.Tensor，并将他们添加到tf.Graph实例中去。
- 调用 tf.constant(42.0) 创建单个 tf.Operation，该操作可以生成值 42.0，将该值添加到默认图中，并返回表示常量值的 tf.Tensor。
- 调用 tf.matmul(x, y) 可创建单个 tf.Operation，该操作会将 tf.Tensor 对象 x 和 y 的值相乘，将其添加到默认图中，并返回表示乘法运算结果的 tf.Tensor。
- 执行 v = tf.Variable(0) 可向图添加一个 tf.Operation，该操作可以存储一个可写入的张量值，该值在多个 tf.Session.run 调用之间保持恒定。tf.Variable 对象会封装此操作，并可以像张量一样使用，即读取已存储值的当前值。tf.Variable 对象也具有 assign 和 assign_add 等方法，这些方法可创建 tf.Operation 对象，这些对象在执行时将更新已存储的值。（请参阅变量了解关于变量的更多信息。）
- 调用 tf.train.Optimizer.minimize 可将操作和张量添加到计算梯度的默认图中，并返回一个 tf.Operation，该操作在运行时会将这些梯度应用到一组变量上。

## 获得默认图
用 tf.get_default_graph，它会返回一个 tf.Graph 对象：

``` python
# Print all of the operations in the default graph.
g = tf.get_default_graph()
```
## 清空默认图
tf.reset_default_graph()
``` python
# 清空当前session的默认图
tf.reset_default_graph()
```

## 命名空间
tf.Graph 对象会定义一个命名空间（为其包含的 tf.Operation 对象）。TensorFlow 会自动为图中的每个指令选择一个唯一名称，也可以指定描述性名称，让程序阅读和调试起来更轻松。TensorFlow API 提供两种方法来指定op名称：
- 如果API会创建新的op或返回新的 tf.Tensor，就可选 name 参数。例如，tf.constant(42.0, name="answer") 会创建一个新的 tf.Operation（名为 "answer"）并返回一个 tf.Tensor（名为 "answer:0"）。如果默认图已包含名为 "answer" 的操作，则 TensorFlow 会在名称上附加 "\_1"、"\_2" 等字符，以便让名称具有唯一性。
- 借助 tf.name_scope 函数，可以向在特定上下文中创建的所有op添加name_scope。当前name_scope是一个用 "/" 分隔的名称列表，其中包含所有活跃的 tf.name_scope 上下文管理器名称。如果某个name_scope已在当前上下文中被占用，TensorFlow 将在该作用域上附加 "\_1"、"\_2" 等字符。例如：
``` python
c_0 = tf.constant(0, name="c")  # => operation named "c"

# Already-used names will be "uniquified".
c_1 = tf.constant(2, name="c")  # => operation named "c_1"

# Name scopes add a prefix to all operations created in the same context.
with tf.name_scope("outer"):
  c_2 = tf.constant(2, name="c")  # => operation named "outer/c"

  # Name scopes nest like paths in a hierarchical file system.
  with tf.name_scope("inner"):
    c_3 = tf.constant(3, name="c")  # => operation named "outer/inner/c"

  # Exiting a name scope context will return to the previous prefix.
  c_4 = tf.constant(4, name="c")  # => operation named "outer/c_1"

  # Already-used name scopes will be "uniquified".
  with tf.name_scope("inner"):
    c_5 = tf.constant(5, name="c")  # => operation named "outer/inner_1/c"
```

请注意，tf.Tensor 对象以输出张量的op明确命名。张量名称的形式为 "\<OP_NAME\>:\<i\>"，其中：
- "\<OP_NAME\>" 是生成该张量的操作的名称。
- "\<i\>" 是一个整数，表示该张量在该op的输出中的索引。

## 获得图中的op
``` python
import tensorflow as tf

c_0 = tf.constant(0, name="c")  # => operation named "c"
# Already-used names will be "uniquified".  c_1 = tf.constant(2, name="c")  # => operation named "c_1"

# Name scopes add a prefix to all operations created in the same context.
with tf.name_scope("outer"):
  c_2 = tf.constant(2, name="c")  # => operation named "outer/c"

  # Name scopes nest like paths in a hierarchical file system.
  with tf.name_scope("inner"):
    c_3 = tf.constant(3, name="c")  # => operation named "outer/inner/c"

g = tf.get_default_graph()
print(g.get_operations())
# [<tf.Operation 'c' type=Const>, <tf.Operation 'c_1' type=Const>, <tf.Operation 'outer/c' type=Const>, <tf.Operation 'outer/inner/c' type=Const>]
```

## 类张量对象
许多 TensorFlow op都会接受一个或多个 tf.Tensor 对象作为参数。例如，tf.matmul 接受两个 tf.Tensor 对象，tf.add_n 接受一个具有 n 个 tf.Tensor 对象的列表。为了方便起见，这些函数将接受类张量对象来取代 tf.Tensor，并将它明确转换为 tf.Tensor（通过 tf.convert_to_tensor 方法）。类张量对象包括以下类型的元素：
- tf.Tensor
- tf.Variable
- numpy.ndarray
- list（以及类似于张量的对象的列表）
- 标量 Python 类型：bool、float、int、str

**注意** 默认情况下，每次使用同一个类张量对象时，TensorFlow 将创建新的 tf.Tensor。如果类张量对象很大（例如包含一组训练样本的 numpy.ndarray），且多次使用该对象，则可能会耗尽内存。要避免出现此问题，请在类张量对象上手动调用 tf.convert_to_tensor 一次，并使用返回的 tf.Tensor。

## tf.Session
### API
``` python
tf.Session.init(
		target, # 可选参数，指定设备。
		graph, #可选参数，默认情况下，新的session绑定到默认graph
		confi # 可选参数，常见的一个选择为gpu_options.allow_growth。将此参数设置为 True 可更改 GPU 内存分配器，使该分配器逐渐增加分配的内存量，而不是在启动时分配掉大多数内存。
)

```
### 创建session
#### 默认session
``` python
# Create a default in-process session.
with tf.Session() as sess:
  # ...

```
####
### 执行op
tf.Session.run 方法是运行 tf.Operation 或评估 tf.Tensor 的主要机制。传入一个或多个 tf.Operation 或 tf.Tensor 对象到 tf.Session.run，TensorFlow 将执行计算结果所需的操作。
tf.Session.run 需要指定一组 fetch，这些 fetch 可确定返回值，并且可能是 tf.Operation、tf.Tensor 或类张量类型，例如 tf.Variable。这些 fetch 决定了必须执行哪些子图（属于整体 tf.Graph）以生成结果：该子图包含 fetch 列表中指定的所有op，以及其输出用于计算 fetch 值的所有操作。例如，以下代码段说明了 tf.Session.run 的不同参数如何导致执行不同的子图：
``` python
x = tf.constant([[37.0, -23.0], [1.0, 4.0]])
w = tf.Variable(tf.random_uniform([2, 2]))
y = tf.matmul(x, w)
output = tf.nn.softmax(y)
init_op = w.initializer

with tf.Session() as sess:
  # 初始化w
  sess.run(init_op)

  # Evaluate `output`. `sess.run(output)` will return a NumPy array containing
  # the result of the computation.
  # 计算output
  print(sess.run(output))

  # Evaluate `y` and `output`. Note that `y` will only be computed once, and its
  # result used both to return `y_val` and as an input to the `tf.nn.softmax()`
  # op. Both `y_val` and `output_val` will be NumPy arrays.
  # 计算y和output
  y_val, output_val = sess.run([y, output])
```

tf.Session.run 也可以接受 feed dict，该字典是从 tf.Tensor 对象（通常是 tf.placeholder 张量），在执行时会替换这些张量的值（通常是 Python 标量、列表或 NumPy 数组）的映射。例如：
``` python
# Define a placeholder that expects a vector of three floating-point values,
# and a computation that depends on it.
x = tf.placeholder(tf.float32, shape=[3])
y = tf.square(x)

with tf.Session() as sess:
  # Feeding a value changes the result that is returned when you evaluate `y`.
  print(sess.run(y, {x: [1.0, 2.0, 3.0]}))  # => "[1.0, 4.0, 9.0]"
  print(sess.run(y, {x: [0.0, 0.0, 5.0]}))  # => "[0.0, 0.0, 25.0]"

  # Raises <a href="../api_docs/python/tf/errors/InvalidArgumentError"><code>tf.errors.InvalidArgumentError</code></a>, because you must feed a value for
  # a `tf.placeholder()` when evaluating a tensor that depends on it.
  sess.run(y)

  # Raises `ValueError`, because the shape of `37.0` does not match the shape
  # of placeholder `x`.
  sess.run(y, {x: 37.0})
```

tf.Session.run 也接受可选的 options 参数（允许指定与调用有关的选项）和可选的 run_metadata 参数（允许收集与执行有关的元数据）。例如，可以同时使用这些选项来收集与执行有关的跟踪信息：

``` python
y = tf.matmul([[37.0, -23.0], [1.0, 4.0]], tf.random_uniform([2, 2]))

with tf.Session() as sess:
  # Define options for the `sess.run()` call.
  options = tf.RunOptions()
  options.output_partition_graphs = True
  options.trace_level = tf.RunOptions.FULL_TRACE

  # Define a container for the returned metadata.
  metadata = tf.RunMetadata()

  sess.run(y, options=options, run_metadata=metadata)

  # Print the subgraphs that executed on each device.
  print(metadata.partition_graphs)

  # Print the timings of each operation that executed.
  print(metadata.step_stats)
```
## 访问当前sess的图。
``` python
sess = tf.Session()
sess.graph
```


## 可视化图
使用图可视化工具。最简单的方法是传递tf.Graph到tf.summary.FileWriter中。如下示例：
``` python
# Build your graph.
x = tf.constant([[37.0, -23.0], [1.0, 4.0]])
w = tf.Variable(tf.random_uniform([2, 2]))
y = tf.matmul(x, w)
# ...
loss = ...
train_op = tf.train.AdagradOptimizer(0.01).minimize(loss)

with tf.Session() as sess:
  # `sess.graph` provides access to the graph used in a <a href="../api_docs/python/tf/Session"><code>tf.Session</code></a>.
  writer = tf.summary.FileWriter("/tmp/log/...", sess.graph)

  # Perform your computation...
  for i in range(1000):
    sess.run(train_op)
    # ...

  writer.close()
```
然后可以在 tensorboard 中打开日志并转到“图”标签，查看图结构的概要可视化图表。

## 创建多个图
TensorFlow 提供了一个“默认图”，此图明确传递给同一上下文中的所有 API 函数。TensorFlow 提供了操作默认图的方法，在更高级的用例中，这些方法可能有用。
- tf.Graph 会定义 tf.Operation 对象的命名空间：单个图中的每个操作必须具有唯一名称。如果请求的名称已被占用，TensorFlow 将在操作名称上附加 "\_1"、"\_2" 等字符，以便确保名称的唯一性。通过使用多个明确创建的图，可以更有效地控制为每个op指定什么样的名称。
- 默认图会存储与添加的每个 tf.Operation 和 tf.Tensor 有关的信息。如果程序创建了大量未连接的子图，更有效的做法是使用另一个 tf.Graph 构建每个子图，以便回收不相关的状态。

### 创建两个图
``` python
g_1 = tf.Graph()
with g_1.as_default():
  # Operations created in this scope will be added to `g_1`.
  c = tf.constant("Node in g_1")

  # Sessions created in this scope will run operations from `g_1`.
  sess_1 = tf.Session()

g_2 = tf.Graph()
with g_2.as_default():
  # Operations created in this scope will be added to `g_2`.
  d = tf.constant("Node in g_2")

# Alternatively, you can pass a graph when constructing a <a href="../api_docs/python/tf/Session"><code>tf.Session</code></a>:
# `sess_2` will run operations from `g_2`.
sess_2 = tf.Session(graph=g_2)

assert c.graph is g_1
assert sess_1.graph is g_1

assert d.graph is g_2
assert sess_2.graph is g_2
```

## 参考文献
1.https://www.tensorflow.org/guide/graphs?hl=zh_cn
2.https://blog.csdn.net/shenxiaolu1984/article/details/52815641
