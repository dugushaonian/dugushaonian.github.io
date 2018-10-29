# Python_learn_note_002

[TOC]

#1. copy

python的赋值语句不会进行复制对象，只是在目标和对象之间创建绑定。对于可变或者包含可变项的集合（collections）有时候需要一个副本，这样可以改变一个副本而同时保持另一个副本不变。这时候可以使用python的copy模块（copy模块的源码[copy.py](https://github.com/python/cpython/blob/3.7/Lib/copy.py)）。copy模块包含浅拷贝（shallow copy）和深拷贝（deep copy）操作，具体介绍如下。

###1.1 接口

copy.copy(x)

​        返回x的浅拷贝

copy.deepcopy(x)

​        返回x的深拷贝

expecttion copy.error

​        抛出特殊模块的异常

浅拷贝和深拷贝的之前的区别仅仅是对复合对象（如list）的拷贝不同

* 浅拷贝构造一个新的复合对象，然后（尽可能）将对它的引用插入到原始对象中。
* 深层复制构造一个新的复合对象，然后递归地将复制插入原始对象中的对象。

深拷贝操作通常存在两个浅拷贝不存在的问题：

* 递归对象（直接或间接包含对自身的引用的复合对象）可能会导致递归循环（死循环）。
* 因为深拷贝会复制它可能复制的所有内容，例如要在副本之间共享的数据。

deepcopy()函数通过以下方式避免了这些问题：

* 保留当前复制过程中已复制的对象的“备忘录”字典;
* 让用户定义的类覆盖复制操作或复制的组件集。

此copy模块不进行复制的类型，如模块，方法，堆栈跟踪，堆栈帧，文件，套接字，窗口，数组或任何类似类型。它通过返回原始对象来“复制”函数和类（浅和深）;这与pickle模块处理这些方式兼容。

可以使用dict.copy()创建字典的浅层副本，通过分配整个列表的切片来列出列表，例如，copied_list = original_list [:]。

类可以使用相同的接口来控制用于控制pickling的复制。有关这些方法的信息，请参阅模块pickle的说明。实际上，复制模块使用copyreg模块中注册的pickle函数。

为了让类定义自己的副本实现，它可以定义特殊方法\_\_copy \_\_()和\_\_deepcopy \_\_()。前者被称为实现浅拷贝操作;没有传递其他参数。调用后者来实现深拷贝操作;它传递了一个参数，即备忘录字典。如果\__deepcopy __()实现需要创建组件的深层副本，它应该调用\_\_deepcopy\_\_()函数，并将组件作为第一个参数，将备注字典作为第二个参数。

#### 1.2 示例

上述来自python官方文档，下面用实例来看看到底区别。

#####1.2.1 基本类型

```python
import copy

x = 1
ass_x = x
shallw_x = copy.copy(x)
deep_x = copy.deepcopy(x)

ass_x is x # True
shallw_xis x # True
deep_x is x # True

```

此时x, ass_x, shallw_x, deep_x依然是同一个对象。下面对x重新赋值：

```python
x = 2
ass_x is x # False
shallw_x is x # False
deep_x is x # False
ass_x is shallw_x # True
ass_x is deep_x # True

```

此时x的值是2，而ass_x, shallw_x, deep_x 的值是1，x对象和ass_x, shallw_x, deep_x指向了不同的对象，但是ass_x, shallw_x, deep_x依然指向同一个对象。下面继续对ass_x赋值：

```python
ass_x = 2
ass_x is x # True
shallw_x is ass_x # False
deep_x is ass_x # False
```

此时ass_x和x又指向了同一个对象。继续对shallw_x, deep_x赋值：

```python
shallw_x = 2
shallw_x is x # True
deep_x = 2
deep_x is x # True
```

从上述示例我们发现对于基本数据，如int，这些不可变的数据类型浅拷贝和深拷贝的作用和赋值操作的作用是相同的，这是因为对于这些类型，copy.copy(x)和copy.deepcopy(x)返回的是x对象本身(具体见源码)。至于为什么当把不同的变量指向同一个数字就成了同一个对象了呢？这个后续会有blog介绍。

#####1.2.2 可变数据类型 如，list

list是中的存储的数据是可变得，对于这样的数据类型，copy()和deepcopy()的差别如何呢？见下面示例：

```python
import copy
x = [1,2]
ass_x = x
shallw_x = copy.copy(x)
deep_x = copy.deepcopy(x)
ass_x is x # True
shallw_x is x # False
deep_x is x # False
shallw_x is deep_x # False
```

我们看到，这时的ass_x和x是指向同一个对象，而x、shallw_x和deep_x分别指向不同的对象。让我们改变x对象中的一个值：

```
x[1] = 6
x # [1,6]
ass_x # [1,6]
shallw_x # [1,2]
deep_x # [1,2]
ass_x is x # Tree
```

此时发现ass_x的值随着x的值改变而改变，这是因为他们指向的是同一个对象，而shallw_x和deep_x的值没有改变，因为他们指向的是不同的对象。这是浅拷贝和深拷贝对x进行复制得到了另外的不同于x的对象。让我们继续改变shallw_x和deep_x的值：

```
shallw_x[1] = 6
shallw_x is x # False
deep_x[1] = 6
deep_x is x # False
```

现在我们发现shallw_x和deep_x改成和x相同的值依然指向的是不同的对象，因为他们是通过对x进行拷贝得到的不同于x的对象。这不同于1.2.1所述的int类型，int类型的数据指向同一个数字时会指向同一个对象，而list却是不同的对象，这和Python本身实现有关，后续会有相关的blog进行介绍，请期待。

到目前为止，我们还未发现浅拷贝和深拷贝有什么任何的差别，别急，继续往下看。

#####1.2.3 含有可变类型的list

在1.2.2中我们介绍的list中包含的是基本类型，那当list的对象中的元素也是list会是如何的呢？

```python
import copy
x = [1,2,[3,4]]
ass_x = x
shallw_x = copy.copy(x)
deep_x = copy.deepcopy(x)
ass_x is x # Tree
shallw_x is x # False
deep_x is x # False
```

依然没差别，别急，坚持，马上就看到差别啦，让我们改变x的值看看。

```python
x[0] = 5
x # [5,2,[3,4]]
ass_x # [5,2,[3,4]]
shallw_x # [1,2,[3,4]]
deep_x # [1,2,[3,4]]
```

依然没差别，继续，让我们把x中的3改成6试试看

```
x[2][0] = 6
x # [5,2,[6,4]]
ass_x # [5,2,[6,4]]
shallw_x # [1,2,[6,4]]
deep_x # [1,2,[3,4]]
```

终于看到不同之处啦。没错，这么多的介绍，真正的差别就是这么点儿。但是这点差别却是很大的差别。赋值操作得到的ass_x和x指向的是同一个对象，所以x变时ass_x也变会变；而浅拷贝和深拷贝得到的对象都是对x复制得到一个副本，但是对于对象中的元素是可变的对象时，如list中的元素也是个list，浅拷贝是对子list对象本身进行拷贝，但是对子list的元素不进行拷贝，所以子list的元素依然是同一个对象，所以shallw_x中的子list的元素会跟着x进行变化，而shallw_x中的不可变元素（如1，2）不会跟着x进行变化；但是深拷贝是对list递归进行的拷贝，不仅对子list本身进行拷贝，对子list中的元素也进行拷贝，所以deep_x完全不会跟着x进行变化。

####1.3 总结

浅拷贝和深拷贝的差别主要体现在可变对象的元素依然是个可变对象时，如list中的元素也是list。浅拷贝只对list中的元素进行拷贝，而对list中的list元素的元素不进行拷贝，而深拷贝会递归的进行拷贝。