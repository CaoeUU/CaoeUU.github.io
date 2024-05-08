---
title: "分布式计算整理-1"
date: 2024-05-08T19:53:38+08:00
draft: false
tags:
- Distributed Computing
- CS Notes
---

> 该笔记整理是基于[邱怡轩老师]()在2024春季面向本科生开设的分布式计算课程的课堂讲解和讲义，整理的目的是巩固并加深本人对相关内容的理解，同时让读者在相对更短的时间内对该课程的关键内容有较为全面的概览。
>
<!--more-->
> *注：这份整理略去了python编程基础相关的内容，侧重点是分布式计算及其涉及的数学知识。*

- *目录更新中...*

### 分布式计算与并行计算
分布式计算与并行计算最主要的区别在于出发点：前者的目的是扩大计算规模（通过不断整合计算资源），而后者是提升计算效率（通过同时进行多个子任务）。

对于并行计算，**Amdahl定律**可衡量其计算效率： 
$$S = \frac{1}{(1-p) + \frac{p}{s}}$$
其中 $p$  为可并行部分的计算时间占比，$s$为这部分能够被加速的倍率。那么整个系统在使用并行计算之后的时间会变化为 ***不能并行部分所用时间 + 能并行从而被加速的部分所用时间***。加速比则使用比率去表示时间的变化再取倒数。

### 函数式编程
不再一次性将数据读入到内存中，而是通过少次遍历来处理。主要内容是迭代器(*iterator*)及其变种，配合一些必要的例子。

- iterator
  
  "To access the elements of a lot of different containers." Container就是以其他类型数据作为元素一类的数据，比如`list`就是一个*container*。对于迭代器的讲解，可以参见CS61A中的[*Iterators*](https://www.youtube.com/watch?v=On-kFyFp8HY)。

  - 例1：计算文件行数
    
    ```python
    count = 0
    # 文件本身可作为迭代器
    with open(path_of_data) as file:
        for line in file:
            count += 1
    ```
  - 例2：计算样本方差
    
    拆解样本方差即可：
    $$S = \frac{1}{n-1}\sum_{i=1}^{n}(X_i-\bar{X})^2 = \frac{1}{n-1}(\sum_{i=1}^{n}X_i^2-n\bar{X}^2)$$
    ```python
    t = iter(samples)
    count = 0
    for i in t:
        count += 1
        quad_sum += i**2
        linear_sum += i
    sample_var = (quad_sum - count*linear_sum**2) / (count-1)
    ```
  - 例3：按给定概率抽样
    
    $w=(a_1,\dots,a_n)$，$v=(v_1,\dots,v_n)$，以 $\frac{a_i}{\sum_j a_j}$ 的概率抽取$v_i$。

    算法伪代码如下：
    [!sampling-algorithm](/images/sampling-algorithm.png)
    
    证明思路：

    - 若最终取样结果为$v_k$，则需保证第k步迭代时成功取到$a_k$，且其后要保持这一取样不变，而不论此前所取样本。因此，该算法所取的样本$v_{i^*} = v_k$的概率为：
    
    $$p = \frac{a_k}{\sum_{i=1}^{k}a_i} \cdot (1 - \frac{a_{k+1}}{\sum_{i=1}^{k+1}a_i})\dots(1 - \frac{a_n}{\sum_{i=1}^{n}a_i}) = \frac{a_k}{\sum_{i=1}^{n}a_i}$$

- islice
  
  需要载入`itertools`包，传入迭代器和截断数量进行切片。

*以下三类函数均为higher order function，即以函数作为参数传入的函数。*
- reduce
  
  需要载入`functools`包。接收的参数是函数操作和迭代对象，主要功能是让迭代器中的元素逐个进行二元运算，在迭代器所迭代的维度上降维至一维。传入的函数操作应该包含两个参数，分别是迭代器的当前对象和下一个对象。需要注意的是python本身的加法运算符进行的加法运算是和数据类型有关的。

- filter
  
  接收的函数需要返回布尔值，并且筛选迭代器中函数值为真的元素。在生成迭代器的时候并没有真的执行对应的筛选函数，只有在使用`next()`返回结果时才会执行。

- map

  将迭代器中所有元素都按照某个函数映射成另一个迭代器，同样在返回结果时才会执行函数。

使用这些函数，且不出现显式`for`循环来实现例子如下：
1. 计算文件行数：
   ```python
      import functools
      with open(path_of_file) as file:
      functools.reduce(lambda x: bool(x)+1, file)
   ```
2. 计算样本均值
   ```python
      import functools
      t = iter(vector)
      functools.reduce(lambda x, y: (x[0]+y[0], x[1]+y[1]), map(lambda x: (1, x),t))
   ```
### Ray基础
Ray是基于python编写的分布式计算框架，之后涉及的分布式计算内容都会基于这个框架。下面是笔者认为使用Ray需要掌握和理解的内容：

- **初始化和结束**
  
  在搭建好ray的环境并载入包后，需要使用`ray.init()`对其进行初始化（且仅能进行一次初始化）；程序结束时用`ray.shutdown()`来关闭。

- **`ray.remote()`生成的任务及提取**
  
  `@ray.remote`作为函数装饰器，使得函数被调用时生成计算任务（`ObjectRef`类型），并分发给各个计算资源，分发后立即进行计算。计算结果需要用`ray.get()`来获取。也就是说`ray.get()`并不是一个启动任务的函数，而只是“被动地”收结果。task在被分发之后就立马被执行了，如果在get的时候任务还没有执行完，那么就得在这个任务上等，才能执行下一句代码。因此说ray.get()是具有阻挡性的。基于这种阻挡性，在取结果的时候应该优先取那些耗时更短的，以便于对先出的结果先进行后续操作。

- `ray.get()`

  需要注意的是，`ray.get()`是将数据读取到本地。而进行计算时，并不需要在本地进行，因此在用`ray.remote`修饰的函数操作中不必要使用`ray.get()`将数据读取出来，直接进行计算即可\**这点可能需要核实*。 而且它并不接受迭代器作为参数，需要先用`list`把任务提取出来再传入。

- **`ray.put()`让计算资源共享数据**
  
  该函数的结果同样是`ObjectRef`类型，这个结果可以直接作为参数传入到要调用的函数中去，也可以使用`ray.get()`重新获取。

- **使用ray进行文件读取**
  考虑到数据的传输效率，单行的传输可能过于低效，因此需要将数据分为更大一些大的批次进行传输。下面是一个1000,000行数据批量读取的示例，需要用到`more_itertools`包：
  ```python
  import ray
  import more_itertools
  ray.init()
  batch_size = 10000
  def txt_to_mat(txt):
    return np.loadtxt(txt)
  def mat_to_obj(mat):
    return ray.put(mat)
  with open(path_to_file_txt) as file:
    it = more_itertools.chunked(file, batchsize)
    it_obj = map(mat_to_obj, map(txt_to_mat, it))
    batches = list(it_obj)
  ```
  需要注意的是，最后一行以`list`的方式提取出迭代器的元素。由于外层的`map`已经将原始的矩阵数据传递到了远端，所以这里的元素实际上是`ObjectRef`对象，直接提取不会有太大问题。另外，由于`map`的运算是在调用迭代器元素的时候才生效，而文件关闭后不能再通过`file`读取文件内容，所以提取操作应该在`with`代码块当中。

### 数值计算
数值计算的原则出发点是计算的效率，用时间复杂度来衡量。下面先给出矩阵运算的时间复杂度，再相应地给出计算原则。

**矩阵乘法**的时间复杂度可由向量乘法出发。$n$ 维向量乘法$\textbf{xy} = \sum_{i=1}^{n}x_iy_i$，时间复杂度可视为 $O(n)$。类似地可以得到 $A_{n \times p}\textbf{x}_{p\times1}$ 的时间复杂度为 $O(np)$， $A_{n \times p} B_{p \times m}$ 的时间复杂度为$O(mnp)$。

**矩阵求逆**的基本计算方式是高斯消元法，即在矩阵右侧添加相同维度的单位阵进行初等行变换。分析方式和矩阵乘法类似，以化为行阶梯型矩阵为例，需要操作的次数是 $n[(n-1)+(n-2)+\dots+1]$，即复杂度为 $O(n^3)$。因此矩阵求逆的时间复杂度为 $O(n^3)$。而以解线性方程组的形式求逆时（尤其是含含有逆矩阵乘以向量），由于增广矩阵右半部分是一个向量，所以时间复杂度会低于直接求逆。

因此，有计算原则：

1. 多个矩阵相乘优先进行维度更小的运算。
2. 将矩阵求逆运算转化为解线性方程组。
3. 分析矩阵结构简化运算（如与单位矩阵相乘时可以利用广播机制来简化）
4. 将多次矩阵与向量的乘法转化为矩阵与矩阵的乘法。
5. 避免重复计算相同的内容。

