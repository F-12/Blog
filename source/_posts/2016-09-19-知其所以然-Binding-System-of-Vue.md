title: 知其所以然-Binding_System_of_Vue
date: 2016-09-19 22:42:00
tags:
- Vue
- Javascript
- Web
---


# 目标
- [ ] 理解Vue的binding系统工作原理
- [ ] 理解单向绑定和双向绑定实现方法
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
  {{ message }}
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
`import Vue from 'vue'`执行时，会首先找到`vue`包，然后执行里面的`index.js`代码，将其中导出的对象绑定到`Vue`这个标识符上。
在这一步，定义Vue类，导出一个构造函数来实例化（Vue实例化时做了很多事情，我们稍后再深入）。

## 实例化过程
### el选项
```puml
@startuml
User -> Vue: new Vue(options)
note left of Vue
此时写在html里的Vue指令原样输出到html
浏览器解析html时跳过元素属性里的vue指令
text节点的vue指令按照字符串输出
end note

Vue -> vm: construct

Vue -> vm: this._initProps()
note left of vm
el选项通过document.querySelector(el)转化为DOM对象
compileAndLinkProps
end note

Vue -> vm: this.$mount(el)

Vue -> vm: this._compile(el)

Vue -> vm: this._initElement(el)
note left of vm
this.$el.__vue__链接到vm实例
hook: beforeCompile
end note
vm -> Vue: el DOM初始化this.$el

Vue -> vm: compileRoot(el, options, null)
note left of vm
处理node上声明的directive
end note
vm -> Vue: rootLinker

Vue -> vm: rootLinker()
vm -> Vue: rootUnlinkFn

Vue -> vm: compile(el, options)

Vue -> vm: compileNodeList(el.childNodes, options)

loop node in el.childNodes

  Vue -> vm: compileNode(node, options)
  note left of vm
  此时node为text node: {{message}}
  end note

  Vue -> vm: parseText('{{message}}')
  vm -> Vue: {{ }}和{{{ }}}匹配得到的tokens
  note left of vm: frag = document.createDocumentFragment()

  loop token in tokens
    alt token is tag
    Vue -> vm: processTextToken(token, options)

    Vue -> vm: parseDirective(token.value)
    note left of vm
    token.value为'message'
    进行词法分析
    需要处理的情况：
    字面量
    变量
    过滤器 |
    括号()
    索引[]
    花括号{}
    end note

    vm -> Vue: 解析出expression
    note left of vm
    解析得到对象{
      experssion: 'message'
    }
    设置token.descriptor = {
          name: 'text',
          def: directives['text'],
          expression: 'message',
          filters: null
    }
    end note


    end
  end

  vm -> Vue: textNodeLinkFn
end

vm -> Vue: childLinkFn

vm -> Vue: compositeLinkFn

Vue -> vm: compositeLinkFn(vm, el)
Vue -> vm: childLinkFn(vm ,textNode)
Vue -> vm: textNodeLinkFn(vm, textNode)

Vue -> vm: this._bindDir()
note left of vm
this._directives中增加新创建的Directive对象
end note
vm -> Vue

Vue -> vm: replace(el, fragClone)
note left of vm
使用新创建的空白DocumentFrag副本替换DOM中的el DOM
此时页面上应该短暂的空白
end note
vm -> Directive: _bind()
Directive -> Watcher: new
Watcher -> Directive: 设置this._watcher

Directive -> Directive: this.update(watcher.value)
Directive -> Directive: this.el[this.attr] = _toString(value)
note left
此时data.message设置到text node中
页面DOM属性和data中数据一致
end note

vm -> Vue: unlinkFn

vm -> Vue:
note left of vm
'hello world'同步到页面上
hook: compiled
hook: atached
hook: ready
end note

Vue --> User: 实例化的Vue对象vm

@enduml
```
### data选项
  <!-- end
  end
  end
  note left of vm
  checkTerminalDirectives
  checkElementDirectives
  checkComponent
  compileDirectives
  存在则返回linkFn
  不存在返回null -->
### `{{message}}`
