title: CloundFoundry使用笔记
tags: Notes
date: 2016-01-07 00:54:05
---


# CLI

# Application
Designing and Running an Application in the Cloud
### Design
- 12-factors Application
- .cfingore
- 多实例增加可用性
- Buildpacks：提供framework and runtime support

### Deploy
- 部署流程

  **应用无关的部署过程**  
  - 上传并保存应用文件
  - 检查并保存应用metadata
  - 创建应用的Droplet（执行单位）
  - 选择DEA运行Droplet
  - 应用启动可用

  **应用相关**  
- 部署准备：cloud-ready，resources，service instances，buildpack
- Credentials and Target
  - API endpoint（
  - username and password
  - workspace -> organization -> space
- 域名配置
- 部署选项：设置选项值，配置环境变量

# Services
## Terminology
+ **Services**  
  - cf集成
  - 交付及操作资源的系统
  - 实现了service broker API
  - cloud controller是其客户端
+ **Service Broker**  
  - service中实现service broker API的组件
+ **Service Instances**  
  - cf集成的
  - 被交付及操作的资源
+ **UPSI**  
  - User-Provided Service Instances
  - 不被cf管理的资源，比如其他地方部署的可通过url访问的一个数据库实例

## Services vs Service Instances
- 管理服务实例
  - 显示服务列表：cf marketplace
  - 创建服务实例：`cf create-service SERVICE PLAN SERVICE_INSTANCE`
  - 显示服务实例列表 & 显示服务实例信息
  - rename服务实例
  - Delete服务实例
- 管理Service key：增删改查
Service key：手动维护的service实例凭证

## Service Instance Credentials
- 自动提供访问service instance的凭证
- Application Binding
- Service Keys

# 自定义Services
## Service Broker
### 架构图 ![架构图](http://docs.cloudfoundry.org/services/images/managed-services.png)
### [Service Broker API](http://docs.cloudfoundry.org/services/api.html)
- catalog
- bind
- unbind
- deprovision (delete)

### 管理Service Broker
**注册**  
- 部署实现了Service Broker API的应用
- 执行注册命令`cf create-service-broker mybrokername someuser somethingsecure http://mybroker.example.com/`
- 权限

**Make Plans Public**  
- 默认注册后为私有的
-

## Catalog Metadata
- Services Marketplace聚集各个Service
- Broker的catalog数据，展示给end users
- End users通过cloud controller clients获取catalog数据  
- 不同clients可以使用不同的catalog数据,通过检测metadata字段

## Binding
- 服务实例为bindable
- binding后分配的凭证写在VCAP_SERVICES环境变量中

## Dashboard Single Sign-On
TBD

# Buildpacks
## 概念
为应用提供framework and runtime support
- 确定需要下载的依赖
- 确定和服务通信的应用配置
- [Buildpack Packager](https://github.com/cloudfoundry/buildpack-packager)
- [System Buildpacks](http://docs.cloudfoundry.org/buildpacks/)

## Detection
- detection priority list
- 优先级一次检测，检测成功则启动，检测失败报错`Unable to detect a supported application type`

## 自定义
**structure**  
主要由三个脚本构成
```
- bin
---- detect
---- compile
---- release
```
- bin/detect:以app的build目录为参数调用，返回0(匹配)或1(不匹配)
- bin/compile:构建Droplet，以build目录和cache目录为参数
- bin/release: 以build位置为参数，必须生成yaml文件

**打包**  
- 使用buildpacks manager在线使用buildpack
- 上传buildpack，离线使用

**使用自定义buildpack部署应用**  
push时指定-b参数
