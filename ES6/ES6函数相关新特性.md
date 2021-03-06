﻿## ES6函数相关新特性
ES6函数相关的新特性可以概括为两个方面，包括函数形式上的新特性，函数参数上的新特性。
--- ---
#### 1. 箭头函数
箭头函数是使用`=>`定义函数的新语法，与传统的函数的区别在于以下几点：

 - 没有属于自己的this,super,arguments和new.target绑定。这些属性均继承外围最近一层的传统函数。
 - 不能通过new来调用箭头函数。也就是**箭头函数不能被用作构造函数**。
 - 没有原型。也就是**没有prototype属性**，因为不能被用作构造函数，所以也就没有存在原型的必要。
上述这三点特性中，在我看来最重要的就是没有自己的this。这一点在很大程度上解决了函数嵌套导致的this值容易混乱的问题。
箭头函数和传统函数并不是互斥关系，而是互补关系，因情况而定使用哪种函数，会写出更加简洁优雅不易错的代码。
使用箭头函数实现一个IIFE：
```
let p = ((name) => {
    return {
        getName: function() {
            return name;
        }    
    }
})("Tom");
```
--- ---
#### 2. 默认参数
默认参数其实就是设置函数参数的默认值，在ES6之前我们也会有这种需要，一般的做法是通过typeof判断参数是否存在，如果存在就是用参数值，不存在就是用默认值，如下所示：
```
function test(value_1, value_2) {
    value_1 = typeof value_1 === "undefined" ? 1 : value_1; // 注意区分null和undefined
    value_2 = typeof value_2 === "undefined" ? 2 : value_2;
    ...
}
```
上述代码的含义是设置value_1和value_2的默认值分别为1和2。可以设置默认值其实比较繁琐。ES6支持设置参数的默认值，大大简化了这一步骤：
```
function test(value_1 = 1, value_2 = 2) {
    ...
}
```
这两种形式功能上是等价的，只是ES6的方式更加简便。参数的默认值除了可以直接是一个具体的值，也可以是函数的返回值，如：
```
function getValue() {
    return 1;
}
function test(value = getValue()) {
    ...
}
```
还有一点需要注意，参数也有`临时死区(TDZ)`的概念。看如下两种声明默认参数的方式：
```
function test1(a, b = a) {
    ...
}

function test2(a = b, b) {
    ...
}
```
test1的方式是可以的，因为在给b赋值的时候，a已经声明过了。而test2的方式不正确，因为在给a赋值时，b还没有声明。
--- ---
#### 3. 处理无命名参数
JS函数理论上允许输入不限量个参数。但是这种情况在实际工作中应该不容易遇到，一般情况函数参数都是人为约定的，应该尽量避免出现多个参数，如果参数的数量确实不固定，最好将所有参数都声明为一个对象的属性，然后把这个对象作为参数传进来。
在ES6之前，如果事先无法确定参数个数，可以通过arguments来获取所有参数。在ES6新增了**不定参数**来处理这种情况。
首先，在命名参数前加三个点"..."就是一个不定参数，每个函数只能有一个不定参数，且这个不定参数是最后一个参数。基本形式如下所示：
```
function test(a, b, ...args) {
    ...
}
```
不定参数是一个数组，包含了除指定参数外其余所有参数（这一点和arguments不同，arguments是所有参数）。