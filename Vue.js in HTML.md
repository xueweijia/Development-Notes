# Vue.js in HTML

## Background and Introduction

### Why JavaScript Frameworks

Thanks to JavaScript, web pages has become more dynamic and vivid. As thousand lines of JavaScript connected to millions of HTML, CSS files without formal organization, JavaScript frameworks become essential to develop and maintain codes eventually in larger scale project.

### What is Vue.js

Vue is a progressive and performant **JavaScript framework** used for building UIs and front-end apps. Vue is an *Approachable*, *Versatile* and *Performant* framework to develop *maintainale* and *testable* code base.

> *Progressive*? If you have an exisiting server side app, you can just **plug** Vue as just one part of application.

### Why Vue.js

* Makes creating UIs and front-end apps much easier.
* Organize JavaScript files
* Fast and lightweight
* Progressive, working together with old codes
* Less learning curve than other frameworks
* `Virtual DOM?`
* Growing in the industry

### Virtual DOM

> The real benefit from Virtual DOM is it allows calculation the different between each changes and make make minimal changes into the HTML document.

When updating DOM, raw way is to delete old element, and rebuild a new one, on another word, rewrite `innerHTML` of its parent node, which is super time-consuming.  
For example, if most data change in a data sheet, resetting the whole innerHTML is rational. However, if only single data changes, rerendering all is just resource wasting. (E.g., front-end just-in-time search in a table)  
Virtual DOM strategy is maintaing a virtual DOM tree as a JavaScript object in memory. When init, the algorithm will abstract the real DOM tree to a JS object, the virtual one. If data changes, actions for Virtual DOM include these following steps:

1. Comparing Virtual DOM tree with real DOM tree(by `diff` algo, `batch` alogo).
2. Update the real DOM(by `patch` algo).

Comparing efficiency between innerHTML and Virtual DOM:

* innerHTML: rendering HTML string for `O(template size)` + rerendering *all* DOM elements `O(DOM size)`(including string parsing).
* Virtual DOM: rendering Virtual DOM + diff comparison for `O(template size)` + rerendering *necessary* DOM elements for `O(DOM change)`.

Virtual DOM rendering + diff is slower than simply string rendering, however, they are still JS calculations and for every operations after that, price is much lower.

In overall aspect, the most value for Virtual DOM is not performance, but it encapsulize direct DOM operations and JS codings to an abstract layer, so that the developers no longer need to operate DOM directly but through Virtual DOM. As Vue's creater Evan You regards, Virtual DOM

* Introduces functional programming.
* Render data to backends besides DOM, like ReactNative.

[Virtual DOM原理浅易详解](https://www.tangshuang.net/3678.html)  
[Why is React's virtual DOM so much faster than the real DOM?](https://www.quora.com/Why-is-Reacts-virtual-DOM-so-much-faster-than-the-real-DOM)  
[理解 Virtual DOM #5](https://github.com/y8n/blog/issues/5)  
(Partial Misleading) ~~[网上都说操作真实 DOM 慢，但测试结果却比 React 更快，为什么？](https://www.zhihu.com/question/31809713)~~

*************

## Basic and Core Function

Starting from rendering data to the web page

*************

### Declarative Rendering 声明式渲染

At the core of Vue.js is a system that enables us to declaratively render data to the DOM using straightforward template syntax:

Vue.js 的核心是一套允许使用简洁的模板语法来*声明式*地将数据渲染到 DOM 的系统。

#### Text Interpolation 文本插值法

```html
<!-- HTML 中创建一套元素，id 为 app -->
<div id="app">
    {{ message }}
</div>
```

``` JavaScript
// 创建 Vue 的实例
var app = new Vue({
    // el: element. 将该实例注册到 app 元素中
    el: '#app',
    // 将数据放入 data 对象
    data: {
        message: 'Hello Vue!'
    }
})
```

现在*数据*和*DOM*已经建立了关联，所有东西都是*响应式*的。

> *Reactive*: When data changes, Vue will take care and update all the places where using the data in the webpage.

#### Binding data with *Directive*: `v-bind`

> *Directive*: prefixed with `v-` to indicate that they are special attributes provided by Vue. They apply special reactive behavior to the rendered DOM.  
*指令*：带有前缀 `v-`，以表示它们是 Vue 提供的特殊属性。它们会在渲染的 DOM 上应用特殊的响应式行为。

`v-bind` 是用于绑定 HTML 属性的指令，可以绑定已有属性，也可以绑定自定义属性。

```html
<div id="app-2">
    <!-- 将这个元素的 title 特性和 Vue 实例的 message 属性保持一致 -->
    <span v-bind:title="message">
    悬停查看信息
    </span>
</div>
```

```JavaScript
var app2 = new Vue({
    el: '#app-2',
    data: {
        message: `Our page is loaded at ${new Date().toLocaleString()}`
    }
})
```

*************

### Conditionals and Loops 条件与循环

数据不仅可以绑定到 *DOM 文本*或*属性*，还能绑定到 *DOM 结构*。

#### Conditionals: Directive `v-if`

```HTML
<div id="app-3">
    <p v-if="seen">Now you see me.</p>
</div>
```

```JavaScript
var app3 = new Vue({
    el: '#app-3',
    data: {
        seen: true
    }
})
```

#### Loops: Directive `v-for`

```html
<div id="app-4">
    <ol>
        <li v-for="todo in todos">
            {{ todo.text }}
        </li>
    </ol>
</div>
```

```JavaScript
var app4 = new Vue({
    el: '#app-4',
    data: {
        todos: [
            { text: 'Learning JavaScript' },
            { text: 'Learning Vue' },
            { text: 'Build something awesome' }
        ]
    }
})

// In terminal
// app4.todos.push({text: 'New project'})
```

*************

### Handling User Input 处理用户输入

#### `v-on` Directive

Use `v-on` directive to attach event listeners that invoke methods on our Vue instances.

使用 `v-on` 指令添加事件监听器，通过这个事件监听器调用 Vue 实例中定义的方法。

```html
<div id="app-5">
    <p>{{ message }}</p>
    <button v-on:click="reverseMessage">Reverse</button>
</div>
```

```JavaScript
var app5 = new Vue({
    el: '#app-5',
    data: {
        message: 'Hello Vue.js!'
    },
    methods: {
        reverseMessage: function () {
            this.message = this.message.split('').reverse().join('')
        }
    }
})
```

#### `v-model` Directive

Two-way binding between form input and app state.  
实现表单输入和应用状态之间的绑定。

```html
<div id="app-6">
    <p>{{ message }}</p>
    <input v-model="message">
</div>
```

```JavaScript
var app6 = new Vue({
    el: '#app-6',
    data: {
        message: 'Hello Vue!'
    }
})
```

*************

### Composing with Components 组件化应用构建

As a developer, we want to make best the code reuse, so custom *component structure* is needed. Component system is another important concept in Vue, because it's an abstraction that allow us to build large-scale applications composed of small, self-contained, and oftern reusable components.  
作为开发者，我们想极尽所能使代码*重用*，*自定义标记结构*是有必要的。在 Vue 中，*组件系统*是另一重要概念，因为它是允许我们将*小型*、*独立*而且通常*可复用*的*组件*构建为*大型应用程序*的一种*抽象*。

[Reference to Web Components](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)

In Vue, a component is essentially is a *Vue instance* with *predefined options*.  
在 Vue 中，组件本质上就是一个 Vue 实例，这个 Vue 实例具有一些预设的选项。

```HTML
<ol>
    <!-- 创建 todo-item 组件的一个实例 -->
    <todo-item></todo-item>
</ol>
```

```JavaScript
// 定义名为 todo-item 的新组件
Vue.component('todo-item', {
    template: '<li>This is a todo item.</li>'
})
```

但这样会为每个待办事项渲染同样的文本。修改组件定义(pre-define)，使之能够接受一个 prop(属性).

```JavaScript
Vue.component('todo-item', {
    // 添加 todo 作为 prop，以将数据从父作用域传到子组件
    props: ['todo'],
    // 模板，将自定义组件实例中默认的元素写在此
    template: '<li>{{ todo.text }}</li>'
})
```

```html
<div id="app-7">
    <ol>
        <!-- 为每个 todo-item 提供 todo 对象，在此将 Vue 实例中的 item 绑定到 todo 对象中，而 item 由 v-for 循环产出 -->
        <!-- 同时，需要为每个组件提供一个 key -->
        <todo-item
            v-for="item in groceryList"
            v-bind:todo="item"
            v-bind:key="item.id"
        ></todo-item>
    </ol>
</div>
```

子单元 prop 接口与父单元 item in groceryList 进行了良好的解耦.

> `key`: 主要用在 Vue 的 Virtual DOM 算法，以在新旧 nodes 对比是表示 VNodes. *`v-bind:key` 是为了在 `v-for` 循环中给 Vue 提示，以便它能够跟踪每个节点的身份，从而重用和重新排序现有元素。每项的 key 是唯一的，如果相同，会发生渲染错误。  
~~使用 key，算法会基于 key 的变化重新排列元素顺序，而且会移除 key 不存在的元素。如果不适用 key, Vue 会使用一种最大限度减少动态元素并且尽可能地尝试修复/再利用相同类型元素的算法。~~  
[Official Documentation](https://cn.vuejs.org/v2/guide/list.html#%E7%BB%B4%E6%8A%A4%E7%8A%B6%E6%80%81)

*************

## The Vue Instance

### Data and Methods

When a Vue instance is created, it adds all the properties found in its `data` object to Vue's *reactivity system*. When the values of those propertie change, the view will "react", updating to match the new values.  
When the data changes, the view will re-render. **It should be noted that properties in `data` are only *reactive* if they existed when the instance was created.** If you know you'll need a property later, but it starts out empty of non-existent, you'll need to set some initial value. For example:

```JavaScript
// Set some initial value
data: {
    newTodoText: '',
    visitCount: 0,
    todos: [],
    error: null
}

// Object.freeze() is the only exception that reactivity system can't track changes.
var obj = {
    foo: 'bar'
}

Object.freeze(obj)

new Vue({
    el: '#app',
    data: obj
})
```

In addition to data properties, Vue instances *expose* a number of useful instance properties and methods. These are prefixed with `$` to differentiate them from user-defined properties.

```JavaScript
var data = { a: 1 }
var vm = new Vue({
    el: '#example',
    data: data
})

vm.$data === data // => true
vm.$el === document.getElementById('example')   // => true

// Call back function: revoked when var a change
vm.$watch('a', function (newValue, oldValue) {
    alert('Data change detected.')
})
```

*************

### Lifecycle Hooks 实例生命周期钩子

[Hooking 钩子编程](https://zh.wikipedia.org/wiki/%E9%92%A9%E5%AD%90%E7%BC%96%E7%A8%8B)

> *钩子编程*：也称作“挂钩”，至通过*拦截*软件模块间的*函数调用*、*消息传递*、*事件传递*来修改或扩展操作系统、应用程序或其他软件组件行为的各种技术。处理被拦截的函数调用、事件或消息等的代码，被称为*钩子*。

Vue runs functions calld *lifecycle hooks*, giving users the opportunity to add their own code at specific stages. Hooks are: `beforeCreate`/`created`/`beforeMount`/`Mounted`/`beforeUpdate`/`updated`/`beforeDestroy`/`destroyed`.

```JavaScript
// For example, created hook can be used to run code after an instance is created.
new Vue({
    data: {
        a: 1
    },
    created: function () {
        console.log(`a is: ${this.a}`)
    }
})
// => "a is: 1"
```

For lifecycle of Vue instances, see diagram:
![./vue_demo/lifecycle.png](Lifecycle)

*************

## References

[MVVM 介绍 => Presentation Logic](https://objccn.io/issue-13-1/)

