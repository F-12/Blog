title: Java工程中使用Properties创建高可配置应用
tags: Java，架构
---
## 动机

## Properties类型
- 系统参数
- 国际化
- 环境配置
## Properties层次
- ###### System Properties（启动时的-D参数）
- ###### 用户配置参数：用户自定义Properties
- ###### 系统配置参数：开发者定义Properties

## Properties模块架构
借助实现工具：OWNER库
通过创建接口，屏蔽读取文件，解析文件的过程

定义获取参数的方法：实现获取方式

定义default值：实现默认配置

定义配置文件查找路径：实现层次化的配置文件

## Properties应用场景
- 数据库配置（username，password，url）
- 运行环境（ENV：production，dev，test等）
- 运行模式（DEBUG/NORMAL）
- 主题（颜色，字体，字号，LookAndFeel等）
- 特定路径配置（数据包存储路径，初始化数据加载路径，临时文件存储路径等）
- 底层库选择（同样接口不同底层实现的配置，JPA配置底层ORM实现vendor等）
