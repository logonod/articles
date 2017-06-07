> 原型是 JavaScript 中一个比较难理解的概念, 本文主要通过举例加图注的形式加深对原型及原型链概念的理解。如有不正之处，欢迎指出。

在 JavaScript 中，每个函数都有一个 prototype 属性，当这个函数被用作构造函数来创建实例时，这个函数的 prototype 属性值会被作为原型赋值给所有对象实例，也就是说，所有实例的原型引用的是函数的 prototype 属性。

为了证明这一点,我们可以在 Chrome 控制台输入：
```javascript
function Person() {

}
var person = new Person();
console.log(person.__proto__ === Person.prototype); // true
```
我们可以得到如下的关系图：
![](http://ww4.sinaimg.cn/large/006tNc79gy1fg89af3vdaj30g0082t8x.jpg)

既然实例对象和构造函数都可以指向原型，那么原型是否有属性指向构造函数或者实例呢？

原型指向实例倒是没有，因为一个构造函数可以生成多个实例。但是原型指向构造函数倒是有的，这就要讲到第三个属性：constructor，每个原型都有一个 constructor 属性指向关联的构造函数。

让我们来验证一下：

```javascript
function Person() {

}
console.log(Person === Person.prototype.constructor); // true
```

所以再更新下关系图：

![](http://ww2.sinaimg.cn/large/006tNc79gy1fg89eran0aj30g4082dg5.jpg)

#### 属性查找

当查找一个对象的属性时，JavaScript 会向上遍历原型链，直到找到给定名称的属性为止，到查找到达原型链的顶部（也就是 Object.prototype），如果仍然没有找到指定的属性，就会返回 undefined。

举个例子：

```javascript
function Person() {

}

Person.prototype.name = 'Kevin';

var person = new Person();

person.name = 'Daisy';
console.log(person.name) // Daisy

delete person.name;
console.log(person.name) // Kevin
```

在这个例子中，我们给实例对象 person 添加了 name 属性，当我们打印 person.name 的时候，结果自然为 Daisy。

但是当我们删除了 person 的 name 属性时，读取 person.name，从 person 对象中找不到 name 属性就会从 person 的原型，也就是 Person.prototype 中查找，幸运的是我们找到了 name 属性，结果为 Kevin。

#### 原型链

同样，person.prototype 对象也有 \_\_proto\_\_ 属性，它指向创建它的函数对象（Object）的 prototype

```javascript
console.log(person.prototype.__proto__ === Object.prototype) //true
```

继续，Object.prototype对象也有  \_\_proto\_\_ 属性，但它比较特殊，为 null

```javascript
console.log(Object.prototype.__proto__) //null
```

我们把这个由  \_\_proto\_\_ 串起来的直到 Object.prototype\._\_proto\_\_ 为 null 的链叫做原型链。

![](http://ww1.sinaimg.cn/large/006tNc79gy1fg89mi2hv1j30ge0elt9d.jpg)

#### 补充

最后，补充两点大家可能不会注意的地方：

**constructor**

```javascript
function Person() {

}
var person = new Person();
console.log(person.constructor === Person); // true
```

当获取 person.constructor 时，其实 person 中并没有 constructor 属性，当不能读取到constructor 属性时，会从 person 的原型也就是 Person.prototype 中读取，正好原型中有该属性，所以：

```javascript
person.constructor === Person.prototype.constructor
```

**\_\_proto\_\_**

其次是 \_\_proto\_\_ ，绝大部分浏览器都支持这个非标准的方法访问原型，然而它并不存在于 Person.prototype 中。实际上，它是来自于 Object.prototype ，与其说是一个属性，不如说是一个 getter/setter，当使用 obj.\_\_proto\_\_ 时，可以理解成返回了 Object.getPrototypeOf(obj)

