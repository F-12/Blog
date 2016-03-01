title: Rust学习笔记
tags: Rust,编程语言
---

# 词法与语法
## 输入
- 编码：UTF-8

## whitespace(空白)
- free-form
- space,tab,LF,CR没有区别

## identifiers（标识符）
标识符命名约束：
- 非空
- `<XID_start><XID_continue>` 或 `_<a-zA-Z0-9><XID_continue>`
- 非关键字

## Comments
注释分类：  
**按用途**  
行注释，块注释，文档注释

**按风格**  
- 注释following items
- 注释containing items

注释列表:
- 行内注释: //
- 块注释: /* ... */
- 文档注释: ///和 /** ... */
- doc attributes: #[doc=""]
- 行内模块注释: //!
- 级模块注释: /*!
- 模块doc attribute #![doc="..."]

## Tokens
### Literals(字面值)
- single token
- 立即求值
- 编译期constant expression
- 各种类型字面值的定义

### Symbols

## Paths
- x
- `x::y::z`
- `::a::foo()` 全局路径
- `super::a::foo()` 相对路径，父模块开始
- `self::foo()` 相对路径，本模块开始

## Syntax extensions（语法扩展）
两种途径：宏和编译器插件
- macro_rules 声明式定义的简单宏
- compiler plugins:插件实现的程序性宏
#### macro_rules
- $name:designator进行匹配
- $() * + 表达循环
- Nested repetitions

# Crates and source files（包和源码文件）
**语义的phase distinction**  
- static interpretation(CT)语义
- dynamic interpretation(RT)语义

**compilation model(编译模型)**  
- 以crate为编译单元，链接单元，一次编译处理一个源码crate产生一个二进制crate
- 一个create组织成树状module scopes，顶层是匿名module
- 编译从单个源码文件输入开始

# Items and attributes
## Items（语言构造）
**7种item类型**  
- extern crate declarations
- use declarations
- modules
- functions
- type definitions
- structs
- enumerations
- constant items
- static items
- traits
- implementations

### Type Parameters（类型参数）
-  modules, constants and statics不能类型参数化，其他四种构造均可以类型参数化
- 类型参数化表示方式：<T,E,X,...>
- 类型参数出现在item名字后定义前，属于名字的一部分，不属于类型的一部分
- 没有高阶类型参数化的类型，只有类型参数化的items

### Modules（模块）
- container for zero or more items
-  module item向crate的modules tree中增加一个module
- 可以随意嵌套
- Modules和types共享命名空间
- path attribute控制加载外部文件模块

### Extern crate declarations（外部包声明）
- 声明链接依赖
TBD

### Use declarations
- 本质上为其他路径创建一个本地name binding
- 不声明链接依赖
- `use p::q::r as x;`进行rebinding
-  `use a::b::{c,d,e,f};`binding a list
- `use a::b::*;`biding all
- `use a::b::{self, c, d};` binding直接父module
- binding后的名字默认为containing module私有
- pub use进行re-export
- use声明中的path必须从crate root开始写

### Functions（函数）
first-class value及相应的function type
#### 返回值
- 显式return
- 隐式return  
- 返回值为()
- diverging

#### 泛型函数
#### Diverging functions（发散函数）
- panic！或发散函数调用结束
- 从不返回任何值

#### Extern functions
- 被外部语言调用的函数
- extern "ABI" fn()类型

### Type aliases(类型别名)
- keyword type
- `type Point = (u8, u8);`

### Structs
- 名义上的struct type
- 名义上的tuple struct type
- 名义上的unit-like struc
- 内存layout默认不指定，可以通过repr attribute指定

### Enumerations
- 同时定义了名义上的enum type和一组constructor
- struct-like enum变量
- enum的判别式，默认从0开始，可以直接赋值，枚举变量被赋值则判别式失效

### Constants
- 命名常量
- 不占用内存空间
- 使用时inlined
- 同一常量的引用不一定是同一个内存地址
- 必须显式类型标注

### Statics
- 代表一个内存地址
- never "inlined"
- 所有引用指向同一个地址
- static lifetime
- 没有interior mutability时被存放在read-only内存区
- **UnsafeCell**来给static赋予interior mutability，使得此类static读取是安全的

**statics的限制**  
+ 没有destructors
+ ascribe to `Sync`
+ 只能通过应用包含其他static，不能通过值
+ constants不能引用statics

**Contants vs Statics**  
`Constants over statics`  
  例外：
  - 大块数据
  - 需要单个内存地址
  - mutability

**Mutable statics**  
`static mut `  
问题：会因为并发引起data race，导致潜在的bug   
解决方案：读写mutable statics时必须使用unsafe block

### Traits
**概念**  
- 接口抽象
- 关联的items：functions，constants，types
- 方法调用语法
- supertraits：Self类型参数的trait bounds，影响vtable中可见的方法列表
- 默认实现
- 静态方法：没有self参数
- trait inherit
**Self**  
- 所有traits隐式定义的一个type parameter
- 代实现这个trait的item的type
- 可以增加trait bounds，但是必须无环

**traits bounds**  
- 限定参数类型
- 实现中可以调用对应trait上的方法

**trait object**
- 每个trait都同时定义一个同名trait object
- 特定类型的pointer转换为trait类型时生成一个trait object
- 隐性转换：参数传递
- 显性转换：as语法

### Implementations
- 一般形式：impl <Trait> for <Struct>
- 省略trait的形式：impl <Struct>

### External blocks
- FFI基础
- 编译器自动在Rust ABI和其他语言的ABI间转换
- 声明方式：`extern "stdcall" { }`，默认为`cdecl`，可以通过`ABI string`指定

## Visibility and Privacy
**name resolution（名称解析）**  
- global hierarchy of namespaces
- `items`声明或定义本质上是向这个全局命名空间中插入一棵子树
- 默认私有，例外：pub enum成员也是pub

**访问控制**  
- public items 可以被通过public ancestors使用
- private item 只能被当前module和后代items使用
