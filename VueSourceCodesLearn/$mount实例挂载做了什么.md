## $mount实例挂载做了什么？
--- ---
接着前一部分[new Vue做了什么？]()，在将`vm.$options.el`传递给`vm.$mount()`方法后，`$mount`又做了哪些事。因为$mount方法的实现是和平台、构建方式相关的。所以在多个文件中都有对此方法的定义。为了能够详细了解Vue在**Compiler with Runtime**下的工作原理，我们选择一个Compiler版本的$mount实现。
打开`src/platform/web/entry-runtime-with-compiler.js`文件，找到对`$mount`方法的定义部分：
```
const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)

  /* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  const options = this.$options
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      template = getOuterHTML(el)
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }

      const { render, staticRenderFns } = compileToFunctions(template, {
        outputSourceRange: process.env.NODE_ENV !== 'production',
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  return mount.call(this, el, hydrating)
}
```
此处的`$mount`方法其实是针对需要**在线编译**的版本实现的。
通过阅读源码，大致梳理`$mount`的内容，首先将Vue原型上的`$mount`存储在`mount`中。然后重新定义了`Vue.prototype.$mount`方法：首先获取`el`，不允许el是body标签或html标签，然后判断`$options`上是否存在`render`，如果不存在，则通过`template`生成一个`render`绑定在`$options`上。所以无论是通过哪种方法写Vue代码，最终都是转化为render函数。由`template`转为`render`是通过`compileToFunctions`函数实现的。最后，通过原型存储的`$mount`方法挂载，这个原先存储的`$mount`是定义在`src/platform/web/runtime/index.js`，是一个公共方法，如果不是在线编译的Vue，其实使用公共方法就足够了。公共的`$mount`代码如下：
```
// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```
执行完了`compiler`的`$mount`，现在走到了公共的`$mount`，在公共部分，调用了`mountComponent`方法，这个方法定义在`core/instance/lifecycle`：
```
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }
  callHook(vm, 'beforeMount')

  let updateComponent
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name
      const id = vm._uid
      const startTag = `vue-perf-start:${id}`
      const endTag = `vue-perf-end:${id}`

      mark(startTag)
      const vnode = vm._render()
      mark(endTag)
      measure(`vue ${name} render`, startTag, endTag)

      mark(startTag)
      vm._update(vnode, hydrating)
      mark(endTag)
      measure(`vue ${name} patch`, startTag, endTag)
    }
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```
在这个方法中，第一个要求就是必须有`render`方法，如果没有，会提示你用`compiler`的在线编译版本。然后就是调用生命周期方法`callHook(vm, 'beforeMount')`，然后就是声明了一个`updateComponent`方法，紧接着`new Wactcher`监听渲染，在`updateComponent`方法中通过`const vnode = vm._render()`获取到虚拟DOM，通过`vm._update(vnode, hydrating)`更新DOM。
Watcher的作用有两个：

 - 初始化的时候执行回调beforeUpdate
 - 当检测的数据发生变化时执行回调beforeUpdate

最后判断`vm._isMounted = true`，表示实例已经挂载，执行回调`mounted`。`vm.$vnode == null`其中`vm.$vnode`表示Vue的父虚拟节点，为null表示当前是根Vue实例。
