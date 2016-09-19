title: 一探究竟-Logback
tags: Java, Logger
---

# logback简介
## 引入目的
- 替代log4j  

## 基础用法
- 配置
- 获取Logger: `LoggerFactory.getLogger()`以当前类名或`class`对象为参数
- 打印: `debug()`,`error()`,`info()`

# Architecture
## Package和class
- logback-core
  - Appender
  - Layout
- logback-classic
  - Logger
- logback-access

# Logger
- 命名实体
- 命名层次及继承体系: 逆DNS命名惯例, 类似包结构的继承体系
- Level及Level的继承: TRACE < DEBUG < INFO <  WARN < ERROR

# Appenders
- Appender Additivity
# Encoders

# Layouts

# Filters

# Mapped Diagnostic Contexts

# Logging Separation

#  JMX Configurator

#  Joran

#  Groovy Configuration

#  Migration from log4j

#  Receivers

#  Using SSL
