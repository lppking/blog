# JS模拟类的前世今生
## 通过new的方式实现继承
JS和Java都是面向对象语言，都支持继承，但是在实现继承的方式上不一样，java是基于类实现的继承，js则是基于原型实现的继承。但是因为历史原因，js在es5及之前的版本是通过new（为了模仿java）的方式实现继承：
```
function c1 () {
  this.a = 1;
  this.b = function () {
    console.log(this.a);
  }
}

var c = new c1();
c.b();
```
除了上面所示的在构造器中添加属性的方式，还可以在构造器的原型上绑定属性：
```
function c2 () {}
c2.prototype.a = 1;
c2.prototype.b = function () {
  console.log(this.a);
}
var cc = new c2();
cc.b();
```

## 通过原型操作函数实现继承
在es6中，js新增了Object.create()、Object.setPrototypeOf()、Object.getPrototypeOf()支持对原型的操作。
借助Object.create()，我们可以给对象指定其原型，由此也可以实现继承：
```
var animal = {
  sayHi: function () {
    console.log('动物叫');
  }
};

var cat = Object.create(animal, {
  sayHi: {
    writable: true,
      enumerable: true,
      configurable: true,
      value: function () {
        console.log('猫叫');
      }
  }
});

var ani = Object.create(animal);
var cat1 = Object.create(cat);

ani.sayHi();
cat1.sayHi();
```
## 通过class实现继承
当然，无论是通过new的方式还是通过Object.create()的方式实现继承总归在语法上有点怪异，所以es6还新增了class来定义类，虽然class只是语法糖，在运行时还是基于原型的继承。
```
class Animal {
  constructor (name) {
    this.name = name;
  }
  
  get animalName () {
    console.log(this.name);
  }
  
  sayHi () {
    console.log('Hi, ' + this.name);
  }
}

class Cat extends Animal {
  constructor (name) {
    super(name);
  }
  climbTree () {
    console.log('猫会爬树！');
  }
}

var cat1 = new Cat('Tom');
cat1.animalName;
cat1.sayHi();
cat1.climbTree();
```