## Vue是从哪里来的
--- ---
为了更加全面的了解Vue的构建相关的流程，我们选择以**Runtime with Compiler**为切入点。打开`src/platforms/web/entry-runtime-with-compiler.js`：
```
import Vue from './runtime/index'
```
发现Vue实例是从runtime目录下的index.js引入的，打开这个文件：
```
import Vue from 'core/index'
```
继续打开core目录下的index.js：
```
import Vue from './instance/index'
```
继续打开instance目录下的index.js：
```
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

// 向Vue.prototype上绑定方法
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```
由此可以发现，其实Vue本身是一个Function实现的类，且export出去的Vue是一个构造函数，所以我们只能通过`new Vue`的方式获取Vue实例。
这里有一个特殊的点，就是Vue之所以使用一个Function而不是一个Class的方式，是因为Vue绑定一些全局的API和方法是通过向`Vue.prototype`进行扩展而实现的，通过Class很难做到。这是一种很值得学习的编程技巧。

看完这个文件，我们回到上一层，`core/index.js`，有一个方法叫做`initGlobalAPI`，位于`./global-api/index`，打开这个文件：
```
export function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
  }

  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  // 2.6 explicit observable API
  Vue.observable = <T>(obj: T): T => {
    observe(obj)
    return obj
  }

  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue

  extend(Vue.options.components, builtInComponents)

  initUse(Vue)
  initMixin(Vue)
  initExtend(Vue)
  initAssetRegisters(Vue)
}
```
这部分代码就给Vue上绑定一些全局API，同时注释中还特殊强调了，最好不要依赖Vue.util，因为这部分是很容易变化的。
--- ---
**所以总的来说，Vue本身是一个Function，然后通过各种init向Vue.protype上绑定各种方法。**
