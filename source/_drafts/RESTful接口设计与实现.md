title: RESTful接口设计与实现
tags: RESTful

---

## 问题设计
假设base url为:`http://example.com/api/v1/`
### 单个实体资源的Restful API
存在下面的资源：
```
# /users
User(id:uuid,name:string,age:integer)
```
GET /users: 获取User列表
- 如何进行分页控制

GET /users/<id>: 获取单个User

POST /users: 创建User实体
- POST的数据应该有哪些字段?所有字段必须具有吗?
- 对于id如何生成,由客户端生成还是服务端生成?
- 数据的服务端验证何时进行,验证出错应该怎么返回?

PUT /users/<id>: 修改User实体
- 不同登录用户允许修改不同的字段,怎么控制?

DELETE /users/<id>: 删除User实体
- 只有管理权限的登录用户才允许删除

对于需要登录才能访问的接口,如何进行身份验证?

### 具有一对多映射关系的资源的Rest API
现在有以下资源:
```
# /merchant
Merchant(id:uuid,name:string,documents:[Document])

# /<merchant_id>/documents/<document_id>
Document(id:uuid,content:text,merchant_id:uuid)
```
一个Merchant具有多个Document,Document实体具有指向Merchant实体的外键. Document必须依赖Merchant存在,不能存在一个指向不存在Merchant的Document实体.

`GET /<merchant_id>/documents/`
- 不存在merchant_id对应的Merchant实体,此时应该404
- 存在merchant_id对应的Merchant实体,但是没有Document实体与之关联,返回空列表

`POST /<merchant_id>/documents/`
- 不存在merchant_id对应的Merchant实体,此时应该404
- 存在merchant_id对应的Merchant实体,但是没有Document实体与之关联,返回空列表
`GET /<merchant_id>/documents/<document_id>`
`PUT /<merchant_id>/documents/`
`DELETE /<merchant_id>/documents/`


## RESTful架构
### 术语
#### Resource(资源)
- 物理资源: txt,jpeg,html等文件,数据库中的数据, 磁盘的空间等
- 虚拟资源: 计算能力, 某一段逻辑的执行等
- 资源实体: 某个抽象资源的实例

#### URI
- URI代表资源实体,相当于资源实体的指针
- URL与URN的区别

#### Represention(表现层)
资源的表示, 一般是格式化后的表现形式, 比如json格式,xml格式,或其他自定义格式
#### State Transfer(状态转移)
资源应该包含后续转换路径的表示形式,给出下一跳的路径

#### HTTP
- 动词
- URL
- 返回码
- Headers

#### HATEOAS(Hypermedia As The Engine Of Application State)

#### RESTful服务成熟度模型
- 0级: 不符合,只是将资源通过网络暴露出来
- 1级: URI, 使用了URI表示资源
- 2级: HTTP动词的使用
- 3级: 符合HATEOAS


## RESTful API设计
### 资源实体定义
- id设计: UUID/整型,关系到ORM对实体的管理
- 关联实体: 1-1, 1-N, N-1, N-N关联
- 值域定义
- 考虑HATEOAS原则

### URI设计
- entry point
- hypermedia化
- path/query参数

### Headers与返回码设计
- Content-Type
- Accept
- Location

### Serialization
- 资源实体序列化机制
- 关联实体与加载策略
- 分页与排序
- hypermedia属性

### Deserialization与Validation
- POST, PUT, PATCH接受载荷的定义
- 反序列化时取值的验证,400返回码,服务端验证

## API设计的描述
参见[swagger](http://swagger.io/)

## API实现(Python版本)
### 工具选择
- python
- Flask
- SQLAlchemy
- mashmallow库

### 资源定义与数据库映射
**ORM方案: SQLAlchemy**  

### URI定义
**Werkzeug路由系统**  
**Flask Web框架**  

### Serialization与Deserialization及Validation
[mashmallow库](http://marshmallow.readthedocs.org/en/latest/index.html)

## 参考
- [REST-维基](https://www.google.co.jp/url?sa=t&rct=j&q=&esrc=s&source=web&cd=3&cad=rja&uact=8&ved=0ahUKEwj_4rWy_p7LAhUCp5QKHShZBkIQFggwMAI&url=https%3A%2F%2Fzh.wikipedia.org%2Fzh%2FREST&usg=AFQjCNEi7AuRxIjZxGS5JjwyNSJzUZS4xA)
- [<<REST in Practice>>](http://www.amazon.com/REST-Practice-Hypermedia-Systems-Architecture/dp/0596805829)

## Next
- RESTful API中的用户认证与授权
- RESTful API的自动化测试
