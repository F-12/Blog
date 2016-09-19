title: 01_深入浅出Node
tags: Node, 读书笔记， Learning
---

## 目录
- Node特点及应用场景
- Javascrpt生态系统(W3C，浏览器，CommonJS)

## 模块机制
### 模块规范:CommonJS/CMD/AMD/ES6
### 模块加载
```
require('module-name');
require('./path/to/custom/module');
require('../path/to/custom/module');

require('./path/to/custom/module.js');
require('./path/to/custom/module.node');
require('./path/to/custom/module.json');
```
**路径分析**  
**文件定位**  
**编译执行**  

### 模块类型
**核心内建模块**  
**核心Javascript模块**  
**JS文件模块**  
**C++扩展模块**  
调用栈

### 包与npm

## 异步
### 异步I/O动机
- **异步消除UI阻塞**  
  js执行时异步请求资源，js执行不被打断，用户交互不被打断
- **异步资源请求并发执行**  
  一个资源请求发出后，立刻发出下一个资源请求
- **单线程同步编程中阻塞I/O**  
  I/O和CPU不分离，存在等待
- **多线程编程**  
  存在死锁，状态同步，资源耗尽的问题

### 异步I/O的实现
- **异步I/O**  
- **阻塞I/O**  
- **几种轮询**  
- **理想的异步IO**  
- **Windows的异步IO**  

### Node的异步
- **事件循环**  
- **观察者**  
- **请求对象**  
- **异步IO时序**  
- **非IO的异步**  

### 异步编程
