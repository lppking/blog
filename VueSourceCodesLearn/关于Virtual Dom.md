## 关于Virtual Dom
--- ---
我们都知道Vue中使用了Virtual Dom，那么什么是Virtual Dom的？
Virtual DOM也叫虚拟DOM，一句话概括其实就是：**通过使用JS对象来表示DOM。**
那么为什么要这么做呢？因为**性能**。日常开发中免不了要操作DOM，所以我们知道DOM有很多的属性，同时DOM的变化在某些情况下会导致页面的重绘。所以通过虚拟DOM来操作DOM的好处在我来看有如下三点：

 - **通过JS对象表示DOM节点，不需要创建真实DOM节点**
 - **只关心关键属性，对于真实DOM中大量的属性不关心**
 - **通过算法，寻找到操作DOM的最优方法，将性能损耗降到最低**

Vue的虚拟DOM是借助一个叫[Snabbdom](https://github.com/snabbdom/snabbdom)的第三方虚拟DOM库实现的。在`src/core/vdom/vnode.js`中，Vue定义了一个`VNode`类，还包括三个方法`createEmptyVNode`、`createTextVNode`、`cloneVNode`。
其实Vue中的`VNode`就是那么表示DOM节点的**JS对象**，对比一下就可以发现，属性少了很多。其实对于一个DOM节点，我们需要关心的也就是标签名、数据、子节点、父节点等，而且我们还可以添加一些自己关心的属性，操作起来也更加的灵活。
那么从虚拟DOM到最终的真实DOM，中间要经历哪些步骤呢？包括virtual dom的建立，新旧virtual dom的对比，最后也仅是渲染差异部分。


