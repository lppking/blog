## new Vue做了什么？
--- ---
在VueJS中可以采用模板语法，声明式的将数据渲染为DOM：
```
<div id="app">
    {{message}}
</div>
```
```
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
});
```
最终页面上回渲染出`Hello Vue!`。为了解释Vue是如何将模板渲染为DOM，需要从`new Vue()`入手。
--- ---
在之前的笔记中，已经知道`new Vue()`位于`src/core/instance/index.js`：
```
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```
这段代码的意思是Vue必须使用new来调用，也就是作为构造函数，不能作为一个普通函数调用，这里判断是否通过new来调用的方法值得学习。通过上述代码，我们可以发现`new Vue()`其实执行的逻辑是`this._init(options)`，这个方法是通过`initMixin(Vue)`预先就绑定在Vue.prototype上的：
```
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    ... ...
  }
}
```
那么`_init()`做了哪些事情呢？通过阅读源码可以发现如下代码：
```
initLifecycle(vm)
initEvents(vm)
initRender(vm)
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```
如上所示，包括初始化生命周期，初始化事件，初始化渲染，调用生命周期方法`beforeCreate`，初始化injections，初始化state，初始化provide，调用生命周期方法`created`。
除此之外，还包括将options挂载到`vm.$options`：
```
if (options && options._isComponent) {
  // optimize internal component instantiation
  // since dynamic options merging is pretty slow, and none of the
  // internal component options needs special treatment.
  initInternalComponent(vm, options)
} else {
  vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
  )
}
```
在_init方法的最后，判断`vm.$options`上是否存在el属性（容器ID），如果存在，则将其传递给`vm.$mount(vm.$options.el)`：
```
if (vm.$options.el) {
  vm.$mount(vm.$options.el)
}
```
--- ---
综上所述，就是`new Vue()`做的事。