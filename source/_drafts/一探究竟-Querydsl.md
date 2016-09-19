title: 一探究竟-Querydsl
tags: Java
---

# 引入目的

# 术语
- Query type: 根据JPA或hibernate配置文件生成的对象类型
- Query variable: Query type实例化变量,默认或自建
- Query implementation: 提供Query的具体实现类,比如JPAQuery,HibernateQuery等

# Query API
**本质上是如何构造type-safe的SQL命令的封装**  
## Expression和Predicate
**提供type-safe的表达式和谓词构造**  
- `Expression<T>`接口:
  - `Constant`接口: 常量表达式
  - `FactoryExpression`接口: - for row based result processing
  - `ParamExpression`接口: - for bindable query parameters
  - `SubQueryExpression`接口: - for subqueries
  - `TemplateExpression`接口: - for custom syntax
  - `Path`接口: - for member access
    - `EntityPath<T>`接口:
      - `RelationalPath<T>`接口: 抽象一个Entity中对其他Entity的引用
  - `Operation`接口: - 封装了`Operator`和作为参数的`Expression`
  - `Predicate`接口: `not`,封装`Expression<Boolean>`,
- `Operator`接口: 封装SQL中的操作符,比如+,-,>,like,limit等
- `XXXBuilder`: Expression Builder
## Query:
**提供SQL查询组件构造**  
- `FilteredClause`接口: `where`
  - `SimpleQuery`接口: `distinct`,`limit`,`offset`,`orderBy`及SQL内变量
    - `Query`: `groupBy`和`having`
      - `JPACommonQuery`接口: `join`,`full join`,`left join`,`inner join`
        - `JPQLQuery`接口: `fetch`和`fetch all`
      - `HibernateQuery`

- `Projectable`接口: 投影运算
# 用例
## 创建
> Query construction in Querydsl involves calling query methods with expression arguments  

### 静态创建
调用Query variable上的方法
### 动态创建
- `Expressions`的静态工厂方法
  - `Expressions.path()`
  - `Expressions.predicate()`
  - `Expressions.path()`
- `XXXBuilder`创建`XXXExpression`
- `PathBuilder`创建`Path<T>`

## Result handling
- `FactoryExpressions`接口:  for row based transformation
- `ResultTransformer`接口:  for aggregation.
