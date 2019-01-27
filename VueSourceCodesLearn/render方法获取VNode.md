## render方法获取VNode
--- ---
在`updateComponent`中，了解到了获取虚拟DOM的方式是通过挂载在实例上的`_render()`方法实现的。提出问题：Vue是如何获取到虚拟DOM的？
在`src/core/instance/render.js`方法中，有`_render`方法的定义：
```
Vue.prototype._render = function (): VNode {
    const vm: Component = this
    const { render, _parentVnode } = vm.$options

    if (_parentVnode) {
      vm.$scopedSlots = normalizeScopedSlots(
        _parentVnode.data.scopedSlots,
        vm.$slots
      )
    }

    // set parent vnode. this allows render functions to have access
    // to the data on the placeholder node.
    vm.$vnode = _parentVnode
    // render self
    let vnode
    try {
      vnode = render.call(vm._renderProxy, vm.$createElement)
    } catch (e) {
      handleError(e, vm, `render`)
      // return error render result,
      // or previous vnode to prevent render error causing blank component
      /* istanbul ignore else */
      if (process.env.NODE_ENV !== 'production' && vm.$options.renderError) {
        try {
          vnode = vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e)
        } catch (e) {
          handleError(e, vm, `renderError`)
          vnode = vm._vnode
        }
      } else {
        vnode = vm._vnode
      }
    }
    // if the returned array contains only a single node, allow it
    if (Array.isArray(vnode) && vnode.length === 1) {
      vnode = vnode[0]
    }
    // return empty vnode in case the render function errored out
    if (!(vnode instanceof VNode)) {
      if (process.env.NODE_ENV !== 'production' && Array.isArray(vnode)) {
        warn(
          'Multiple root nodes returned from render function. Render function ' +
          'should return a single root node.',
          vm
        )
      }
      vnode = createEmptyVNode()
    }
    // set parent
    vnode.parent = _parentVnode
    return vnode
}
```
为了探究render原理，我们只关注render相关的。这个方法的核心在`vnode = render.call(vm._renderProxy, vm.$createElement)`，这个`render`方法是之前经过`template`编译的，因为实际开发中通常是写`template`，很少自己写`render`。
结合一个例子，了解一下`template`编译为`render`是什么样子：
```
// template
<div id="app">
    {{message}}
</div>

// render
{
    render: function (createElement) {
        return createElement('div', {
            attrs: {
                id: 'app'
            }
        }, this.message);
    }
}
```
结合`render.call(vm._renderProxy, vm.$createElement)`，可以了解到`createElement`参数就是`vm.$createElement`。那么`vm.$createElement`上的createElement是怎么来的呢？其实在本文件的`initRender`方法中有定义：
```
// bind the createElement fn to this instance
// so that we get proper render context inside it.
// args order: tag, data, children, normalizationType, alwaysNormalize
// internal version is used by render functions compiled from templates
vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
// normalization is always applied for the public version, used in
// user-written render functions.
vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
```
在这部分代码中，为实例上绑定了`_c`和`$createElement`两个方法，却别在于`_c`是被template编译成的render方法调用，而`$createElement`是被用户手写的render方法调用的。
那么关于`createElement`方法的定义，是在`import { createElement } from '../vdom/create-element'`中实现的。