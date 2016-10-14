title: 知其所以然-Binding_System_of_Vue
date: 2016-09-19 22:42:00
author: F-12
tags:
- Vue
- Javascript
- Web
---


# 目标
- [x] 理解Vue如何根据data生成DOM及挂载到Document对象上
- [x] 理解Vue如何实现更新data属性值时自动更新DOM
- [ ] 理解Vue如何实现计算属性自动更新
- [ ] 理解Vue如何实现v-*指令
- [ ] 理解Vue如何实现事件系统
- [ ] 实现一个简单的单向绑定的Demo

# 引入
## DOM更新

浏览器构建DOM Tree，渲染视图，并提供DOM API更新视图。浏览器保证视图的渲染结果和DOM Tree中的数据同步。开发者经常会从服务器加载数据更新到视图上，或根据用户的交互改变视图。这些改变视图的方式可以抽象成改变视图状态数据（即数据驱动视图）。

我们一般都会使用data object保存从服务器加载到的数据或记录用户交互的状态数据，然后把这些data object里的数据同步到DOM对象的各个属性值上，触发视图的更新。

一般的做法是这样的：

```
<div>
  Hello,<span id="name"></span>
</div>

<script>
  var name = 'f-12';
  $('#name').text(name);
</script>
```
name代表data object（既可能是从服务器上加载的数据，也可能是用户交互产生的状态数据），当我们把值赋给DOM的属性（使用原生DOM API或jQuery)，修改视图后，data，object就和DOM的属性值没有关系了，假如我们再次更新name，DOM属性值并不会变化，因而视图也不会变化。我们必须再次修改DOM属性才可以更新视图，即每次状态数据发生变动时我们必须手动更新视图。

手动更新DOM显得很繁琐。前端工程更加复杂，用户交互更加复杂，单页应用更加流行，产生了更多的状态数据，这些因素也凸显了手动更新DOM的缺点。

## Binding
Binding的思想就是建立data object和DOM对象之前的关系，使得我们修改data object上的属性值时自动完成DOM对象的更新。

# 观察者模式
- 原理
- javascript实现的观察者模式Demo

# Vue的binding系统
## Overview
{% asset_img vue_binding_model.png %}
（图片来自vuejs.org)

Web开发中比较频繁出现的场景是：
- 有很多状态数据需要同步到DOM上（DOM操作，这也是jQuery流行的主要因素）
- 多个DOM对象用到同一个状态数据，状态发生变动时需要更新所有DOM

## 最简单的🌰
```html
<div id="app">
  This is a {{ message }}.
</div>
```
```javascript
import Vue from 'vue';
const vm = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue.js!'
  }
})
```
—— Reference [vuejs.org](http://vuejs.org/guide/)

这个代码段的效果是页面渲染时，message的值会自动同步到DOM中，当修改message时，DOM会同步更新。
接下来，我们来探究一下这一切是怎么发生的。

## import
`new Vue()`之前，我们首先得引入Vue的库，我们使用ES6的`import`方式。
`import Vue from 'vue'`执行时，会首先找到`vue`包，然后执行里面的`index.js`代码，将其中导出的对象绑定到`Vue`这个标识符上。这一行代码完成Vue类的定义，导出一个构造函数来供使用者实例化。
Vue实例化时做了很多事情。

## 实例化过程
### `el`选项
以下段落来自Vuejs官方文档对`el`选项的描述。
- **Type:** `string | HTMLElement`

- **Restriction:** only respected in instance creation via `new`.

- **Details:**

  Provide the Vue instance an existing DOM element to mount on. It can be a CSS selector string or an actual HTMLElement.

  After the instance is mounted, the resolved element will be accessible as `vm.$el`.

  If this option is available at instantiation, the instance will immediately enter compilation; otherwise, the user will have to explicitly call `vm.$mount()` to manually start the compilation.

  <p class="tip">The provided element merely serves as a mounting point. Unlike in Vue 1.x, the mounted element will be replaced with Vue-generated DOM in all cases. It is therefore not recommended to mount the root instance to `<html>` or `<body>`.</p>

- **See also:** [Lifecycle Diagram](/guide/instance.html#Lifecycle-Diagram)

`el`选项提供`Vue`实例对象的挂载点，最终`Vue`实例将编译出一个`DOM`对象替换`el`对象的`DOM`。

{% asset_img vue_el_option_init.png %}

### `data`选项
以下段落来自Vuejs官方文档对`data`选项的描述。
- **Type:** `Object | Function`

- **Restriction:** Only accepts `Function` when used in a component definition.

- **Details:**

  The data object for the Vue instance. Vue will recursively convert its properties into getter/setters to make it "reactive". **The object must be plain**: native objects such as browser API objects and prototype properties are ignored. A rule of thumb is that data should just be data - it is not recommended to observe objects with its own stateful behavior.

实例化过程中有一个过程是`this._initData()`，在这个过程的最后一步是调用`function observe (value, vm) {}`，即`observe(this.data, this)`。
结果是`data`对象的`__ob__`保存新建的`Observer`对象，且此`__ob__.value`属性保存`data`选项对象。同时会遍历`data`选项对象的每个键值，使用ES6的`Object.defineProperty`定义其Getter和Setter。如果键值也是对象，会递归调用`observe`函数，即`observe(key)`。
完成`observe`调用后的`this.$data`结构
```javascript
{
  __ob__: {
    dep: {
      subs: []
    }
    value: {} //即this.$data自身
    vms: [] //所有持有此data对象的vm实例
  },
  get message() {}
  set message(newVal) {}
}
```
因为`observe`过程是在`this._initData()`过程中发生，而这个过程又在`new Vue(options)`构造函数的执行中进行，所以一旦实例创建结束后，再往`data`上增加的新键值并不是响应式，因为没有经过`observe`过程转化为响应式属性。

### Text Node中的`mustache`编译
#### 提取
- **NodeType识别Text Node**
- **正则表达式提取tokens**
  safe: `\{\{((?:.|\\n)+?)\}\}`
  unsafe: `\{\{\{((?:.|\\n)+?)\}\}\}`

  上面例子中的 `This is a {{ message }}.`提取后形成下面的`tokens`列表：
```javascript
[
  {value: 'this is a '},
  {
    html: false,        // 是否由unsafe模式匹配到的
    oneTime: false,     // 稍后介绍
    tag: true,          // 是否由safe或unsafe模式匹配到的
    value: 'message'    // 匹配到的文本值
  },
  {value: '.'}
]
```

  Text中的每一个`mustache`匹配到的文本将生成类似列表中第二个对象的一个token对象。其他token是由其余简单文本生成的。

#### 创建Text Node
  根据每个`tag`属性为true的token使用`function processTextToken (token, options) {}`处理，创建对应的Text Node对象并加入到一个新创建的`DocumentFragment`对象中。此过程中将进一步对token标记`descriptor`。`mustache`内部可能包含的过滤器调用也是在`descriptor`中标记。`token.tag`为`false`直接生成简单的Text Node。
  此时的token结构如下：

```javascript
{
  html: false,
  oneTime: false,
  tag: true,
  value: 'message',
  descriptor: {
    name: 'text',           // 表示当前token代表一个text node
    // directives/public/text预先定义的directive
    def: {
      bind() {},
      update() {}
    },
    expression: 'message',  // 由token中的值生成的表达式
    filters: undefined      // 可能包含的过滤器调用
  }
}
```

#### Link
递归编译每个DOM节点时，会对每个node生成对应的linker函数，这些linker会最终组合成一个linker。编译过程结束后，调用这个函数会开始link过程。代码概括为`compile(el, options)(this, el)`。link过程会递归调用所有的linker。

每个linker调用会创建一个`Directive`对象并添加到Vue实例的`_directives`数组中。
```
this._directives.push(
  new Directive(
    descriptor, // 编译过程中token中标记的discriptor
    this,       // 当前Vue实例
    node,       // 创建的对应DOM node对象
    host,       // 宿主DOM node
    scope,
    frag
  )
);
```

当所有的`Directive`对象添加到Vue实例的`_directives`数组中后，会遍历新增加的所有`Directive`，调用其上的`_bind`方法。（由于Vue提供了通过自定义`Directive`复用代码的方式，这里为了防止在实例化过程中调用到这些`Directive`的`_bind`方法，Vue通个`Directive.discriptor.priority`中保存自身优先级， link过程中创建的`Directive`对象都具有默认优先级。然后遍历前会现根据优先级排序。）

如下代码所示：
```javascript
  var originalDirCount = vm._directives.length
  linker()
  var dirs = vm._directives.slice(originalDirCount)
  sortDirectives(dirs)
  for (var i = 0, l = dirs.length; i < l; i++) {
    dirs[i]._bind()
  }
```

至此link过程完成了一半，继续进行下一半前，我们需要引入`Directive`类和`Watcher`类。

一个directive对象将DOM Element和data中的属性链接起来，并在内部注册一个Watchre对象，在data属性更新时调用update函数更新DOM对象。
```javascript
// 内部结构如下：
{
  vm: Object,           // Vue实例
  el: Object,           // 根DOM
  descriptor: Object,   // discriptor
  name: String,         // discriptor.name
  expression: Object,   // discriptor.expression
  arg: Object,          // discriptor.arg
  modifiers: [],    // discriptor.modifiers
  filters: [],      // discriptor.filters
  literal: ,
  _locked: Boolean, //
  _bound: Boolean,   // intial bind后为true
  _listeners: [], //
  _host: Object,        // context信息
  _scope: Object,     // context信息
  _frag: Object     // context信息
}

/**
 * @constructor
 */
export default function Directive (descriptor, vm, el, host, scope, frag) {}

Directive.prototype._bind = function () {}

/** 其他一些方法 */

```

一个Watcher对象封装一个表达式，收集该表达式的依赖项，在表达式的值发生变化时调用回调函数。
```
// 内部数据结构如下
{
  vm： Vue, // Vue实例
  expression: String | Function,   // 被封装的表达式，来自discriptor.expression
  cb: Function,
  id: Number,                      // 内部自增全局变量uid，批处理使用
  active: Boolean,
  dirty: Boolean,                  // lazy watchers标志量
  deps: [],                        //
  newDeps: [],
  depIds: Set,
  newDepIds: Set,
  prevError: null, // for async error stacks
  getter: Function,
  setter: Function,
  queued: Boolean, // for avoiding false triggers for deep and Array
  shallow: Boolean // for avoiding false triggers for deep and Array
}
/**
 * @constructor
 */
export default function Watcher (vm, expOrFn, cb, options) {}

Watcher.prototype.get = function() {}           // 计算expression的值并重新计算依赖
Watcher.prototype.set = function(value) {}      // 先应用可能的filter然后设置对应属性值
Watcher.prototype.beforeGet = function() {}     // 设置全局Dep.target为当前Watcher实例，准备收集依赖
Watcher.prototype.afterGet = function() {}      //
Watcher.prototype.update = function(shallow) {} // Subscriber接口，被Publisher调用，Vue的Publisher由Dep实现
Watcher.prototype.run = function() {}           // Batch job接口，被batcher调用
Watcher.prototype.evalute = function() {}
Watcher.prototype.addDep = function() {}        //
Watcher.prototype.depend = function() {}        // 遍历调用deps上的depend
Watcher.prototype.teardown = function() {}      // 清楚一个watcher，遍历调用deps里的removeSub()移除对其的订阅
```

在directive对象bind的过程中， 会删除`el` DOM上声明的v-*指令，然后将discriptor上定义的属性mixin到自身，并调用`discriptor.bind()`完成initial bind。
然后会new一个Watcher对象，并初始化更新DOM，将data上的初始化值更新到DOM上（这里需要例外处理v-model，将input的inline value回写到data中）。

此时我们生成的`Directive`和`Watcher`对象如下:
```
{
    _bound: true,
    _frag: undefined,
    _host: undefined,
    _listeners: null,
    _locked: false,
    _scope: undefined,
    _update() {},
    // 每个directive对象都会持有一个watcher对象
    _watcher: {
        id: 1,
        active: true,
        cb(val, oldVal) {},
        deep: undefined,
        depIds: [1],
        deps: [Watcher],
        dirty: undefined,
        expression: "message",
        filters: undefined,
        getter() {}
        newDepIds: [],
        newDeps: [],
        postProcess: null,
        preProcess: null,
        prevError: null,
        queued: false,
        scope: undefined,
        setter: undefined,
        shallow: false,
        twoWay: undefined,
        value: "hello world"
    },
    arg: undefined,
    attr: "data",        // 因为是data选项中定义的属性，所以值为data，如果只是简单text则为textContent
    descriptor: Object,  // token.descriptor
    bind: bind(),        // token.descriptor.bind
    update(value) {},     // token.descriptor.update
    el: Text,             // DOM中的Text类实例
    expression: "message",
    filters: undefined,
    literal: undefined,
    modifiers: undefined,
    name: "text",         // 标明此directive对象注册时key为text，自定义directive时将由Vue.directive(name, Object)提供name
    vm : Vue
}
```
#### 依赖系统
前面讲解link过程时，提到了收集依赖。所以一并讲解下Vue的依赖系统。依赖系统本质上是一个发布-订阅模式的实现。`Watcher`实现订阅者职责，`Dep`实现发布者职责，发布的内容是自身值的改变，`Watcher`对应的响应是使用依赖重新计算自己的值并调用回调函数。

为此引入`Dep`类。
一个Dep对象是一个发布者，可以被多个directive（实际上是directive里面的watcher）订阅。
```javascript
// 内部结构
{
  id: Number,      // 自增的uid
  subs: []         // 实现了update方法的对象
}

export default function Dep () {}

Dep.target        // 全局变量，标志当前收集依赖的Watcher对象

Dep.prototyp.addSub = function() {}        // 增加订阅者
Dep.prototyp.removeSub = function() {}        // 移除订阅者
Dep.prototyp.depend = function() {}        // 将自身增加当Dep.target的依赖中
Dep.prototyp.notify   = function() {}        // 遍历调用subs里对象的update方法，即发布更新
```
#### Binding流程
下面使用流程图展示这个过程中的行为。
{% asset_img vue_binding_sequence.png %}

**workshop**
debug `vm.message = 'Goodbye World'`时的执行时序

### 编译模板
`Vue`会根据写在`template`里的模板编译出`DOM`对象。`template`模板本身是`html`代码，`Vue`的功能通过写在html代码里的指令，`mustache`代码以及自定义`tag`实现。
模板的编译本质上是手动创建`DOM`树的过程。`html`中我们重点关注`Element Node`,`Attribute Node`,`Text Node`。

#### `v-*`指令编译
#### 自定义标签编译

## 更高效阅读源码
### 确定目标问题
### 简化与忽略
### 关注状态
### 断点验证
### 输出
