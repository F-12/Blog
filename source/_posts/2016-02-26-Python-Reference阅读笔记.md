title: Python-Reference阅读笔记
tags: 'Python, Reference, Note'
date: 2016-02-26 14:27:17
---


# 数据模型
## 对象、值和类型
+ `id()`函数与对象ID
+ `type()`函数与对象类型
+ 对象创建后天生具有ID，type
+ 值可变为可变对象，值不可变为不可变对象，由type决定

## 标准类型的层次结构
+ `None`
+ `NotImplemented`
+ `Ellipsis`
+ Numbers有numbers模块定义类
  - numbers.Number
  - numbers.Complex
  - numbers.Real
  - numbers.Rational
  - numbers.Integral
+ Sequence
  **可变**  
  - Strings：`chr()`,`ord()`函数在字符和字节序数间转换
  - Unicode: `unichr()`, `ord()`在Unicode和序数间转换
  - Tuple:
  **不可变**  
  - Lists:
  - Bytearray: `bytearray()`创建
+ Sets
  - set: `set()`创建，可变容器
  - frozenset: `frozenset()`创建，不可变容器
+ Maps
  - dict: `{}`创建，键是不可变对象
+ Callable
+ User-defined functions
  **函数属性**  
  - `__doc__`
  - `__name__`
  - `__module__`
  - `__defaults__`
  - `__code__`
  - `__globals__`
  - `__dict__`
  - `__closure__`
+ User-defined methods
  **将class，instance和Callable对象结合起来**  
  - 绑定的User-defined method: 通过类获取，im_self指向None
  - 未绑定的User-defined method: 通过实例获取，im_self指向实例
  - im_class指向类对象，im_func指向原始函数对象
  - __doc__
  - __name__
+ 静态方法对象
+ 类方法对象
+ Generator
+ Built-in functions
  - 封装C函数
  - 特殊属性
+ Built-in methods
  - C函数的另一种封装，包含传递给C函数实例作为隐式参数
+ Module
  - `import`语句导入的对象
  - 包含一个字典实现的命名空间，属性的引用被转换成查询这个字典
  - `__name__`为模块的名字
  - `__doc__`指模块的文档字符串，如果没有则为None
  - `__file__`为模块加载的文件路径
+ Classes
  **Class type**:新式类，可调用，通常作为工厂创建它们自己的实例,调用参数传递给`__new__()`，然后在调用`__init__()`
  **Classic Classes**:类调用时，参数传递给`__init__()`方法
  - 用字典对象实现的命名空间,类属性的引用被转换为这个字典的查询
  - 类属性的赋值将更新该类的字典，永远不会更新其基类的字典。
+ Class instances
  - 具有`__call__()`方法时实例可以调用
  - 实例属性字典->类属性字典->`__getattr__()`方法
  - 属性的赋值和删除会更新实例字典，永远不会是类的字典。如果类有一个__setattr__() 或者__delattr__() 方法，那么会调用这个方法而不是直接更新实例的字典。
+ File
+ 内部类型
+ Code objects
+ Frame objects
  **执行帧**
+ Traceback objects
+ Slice objects
## 新式类和经典类
## 特殊方法的名字
+ **运算符重载**
  - `__getitem__()`下标
  - `object.__new__(cls[, ...])`构造
  - `object.__init__(self[, ...])`初始化，子类需要显式调用基类的__init__
  - `object.__del__(self)`del操作符
  - `object.__repr__(self)`对象正式字符串表示`repr()`函数调用
  - `object.__str__(self)`对象非正式字符串表示`str()`函数调用
  **比较操作符**  
  - `object.__lt__(self, other)`
  - `object.__le__(self, other)`
  - `object.__eq__(self, other)`
  - `object.__ne__(self, other)`
  - `object.__gt__(self, other)`
  - `object.__ge__(self, other)`
  - `object.__hash__(self)`
  - `object.__nonzero__(self)`
  - `object.__unicode__(self)`
## 自定义属性访问
  - `__getattr__()`
  - `__getattribute__()`
  - `__setattr__()`
  - `__delattr__()`
# 执行模型
TBD
