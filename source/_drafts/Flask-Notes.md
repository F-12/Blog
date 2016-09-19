title: Flask-Notes
tags: Flask, Python
---
# Overview
- API
- Patterns
- 依赖: Jinja2模板引擎和Werkzeug WSGI toolkit
- 微框架: 只实现核心功能，通过扩展实现其他功能，扩展容易替换
- 线程局部的对象 vs Request Context

# 路由
# assets
# WSGI 中间件
# request
# response
# session
# 消息闪现
# 日志
# 调试与测试
# 配置
# 信号机制（web组件的生命周期）
# 视图
## 模板与模板引擎
## plugable view
# 上下文
## 两种状态：应用配置状态 vs 请求处理状态
## App Context(应用上下文)
### 作用
- 设计目的：一个 Python 进程中拥有多个应用
- 应用识别问题：显式地传递应用 vs current_app代理对象
- 没有请求时创建请求上下文是没有必要的昂贵操作
### 创建
- 显式：app.app_context()
- 隐式：request context压栈时自动创建，可忽略
## 请求上下文
### 上下文作用域
- 请求上下文通过栈维护
- 使用 with 声明或是调用RequestContext对象的push()和pop()方法
### 回调，错误，保护机制
TBD
# Blueprint
# Flask扩展
- 发现扩展：[Flask Extension Registry](http://flask.pocoo.org/extensions/)
- 导入扩展：`from flask.ext import foo`
- 兼容包
```
import flaskext_compat
flaskext_compat.activate()
```
# Shell中使用Flask
- `import Flask`
- 创建app
- 创建RequestContext`ctx = app.test_request_context()`
- RequestContext压栈`ctx.push()``
- 请求前调用`app.preprocess_request()`
- 使用请求request等对象
- 关闭请求`app.process_response(app.response_class())`
- RequestContext出栈`ctx.pop()`
