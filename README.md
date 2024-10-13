
如果你想弄明白什么怎样才可以实现JavaScript的原型链污染，那么你首先需要弄清楚两个东西，那就是`__proto__`和`prototype`。


### 到底什么才是`__proto__`和`prototype`?


那我们先来看看比较官方的说法吧：



```
__proto__:是每个对象的隐藏属性，指向创建该对象的构造函数的原型对象（prototype）。它是对象用于继承属性和方法的机制。它是一个对象。所有的对象都可以通过__proto__ 来访问它的原型，进而实现原型链查找。

```


```
prototype是函数（特别是构造函数）特有的属性，它用于定义由该构造函数创建的实例共享的属性和方法。prototype通常用来定义构造函数的实例方法。当我们创建一个新对象时，该对象会继承其构造函数 prototype上的属性和方法。

```


```
const obj = {};
console.log(obj.__proto__ === Object.prototype); // true

```

在这个例子中，`obj`是通过`Object`构造函数创建的，所以`obj.__proto__`指向`Object.prototype`。


那我们可以这样去理解，prototype是类通有的属性，当类被实例化为对象后，对象会拥有prototype中的属性和方法。当对象想去访问类的原型时用`__proto__`属性来访问类的原型。


### prototype链


了解了`__proto__`和`prototype`之后，那我们就要去深入了解JavaScript的继承链，在JavaScript中，每个对象都有一个指向其原型的内部链接（即`__proto__`），这个原型本身也是一个对象，通常还有自己的原型，这样就形成了一条原型链。当你访问一个对象的属性时，JavaScript 会沿着这条原型链逐级查找，直到找到该属性或者原型链的顶端（即Object.prototype）。



```
function Animal(name) {
  this.name = name;
}

function Something() {
  this.speak = console.log(this.name + ' makes a beautiful sound.');
};

const dog = new Animal('Dog');
dog.speak; // 输出: 'Dog makes a beautiful sound.'

```

实例化Animal类创建了dog对象，当访问speak属性时，在dog中寻找不到时，会在`dog.__proto__.__proto__`中寻找，发现`dog.__proto__.__proto__`中有speak属性，也就是`Something.prototype`中存在speak属性。也就是说JavaScript使用prototype链实现继承机制。


### prototype链污染（原型链污染）


那我们学习继承、prototype链不就是为了进行链子污染，然后得到我们想得到的东西吗？


既然`dog.__proto__.__proto__`指向创建该对象的构造函数的原型对象（`Something.prototype`）。那么我们修改`dog.__proto__.__proto__`的内容是不是可以进而实现了修改Something类。


示例：



```
function Animal(name) {
    this.name = name;
}

function Something() {
    this.speak = console.log(this.name + ' makes a beautiful sound.');
};

const dog = new Animal('Dog');

dog.__proto__.speak = console.log('修改成功');

dog.speak;

```

[![](https://img2024.cnblogs.com/blog/3262760/202410/3262760-20241012214215162-44737841.png)](https://img2024.cnblogs.com/blog/3262760/202410/3262760-20241012214215162-44737841.png)


通过这个结果我们可以看出我们轻易的将`dog.__proto__.__proto__`的speak值进行了更改。我们先将这个继承链给搞出来：



```
dog -> Animal.prototype -> Something.prototype -> Object.prototype -> null

```

那么攻击者如果通过一个入口点，控制并修改了一个对象的原型，那么将可能影响所有和这个对象来自同一个类、父类直到Object类的对象。


典型的便是merge操作导致原型链污染：



```
javascript
function merge(target, source) {
    for (let key in source) {
        if (key in source && key in target) {
            merge(target[key], source[key])
        } else {
            target[key] = source[key]
        }
    }
}

let object1 = {}
let object2 = JSON.parse('{"a": 1, "__proto__": {"b": 2}}')
merge(object1, object2)
console.log(object1.a, object1.b)

object3 = {}
console.log(object3.b)
//1 2
//2

```

在JSON解析的情况下，`__proto__`会被认为是一个真正的“键名”，而不代表“原型”，所以在遍历object2的时候会存在这个键。


  * [到底什么才是\_\_proto\_\_和prototype?](#%E5%88%B0%E5%BA%95%E4%BB%80%E4%B9%88%E6%89%8D%E6%98%AF__proto__%E5%92%8Cprototype)
* [prototype链](#prototype%E9%93%BE)
* [prototype链污染（原型链污染）](#prototype%E9%93%BE%E6%B1%A1%E6%9F%93%E5%8E%9F%E5%9E%8B%E9%93%BE%E6%B1%A1%E6%9F%93)

   \_\_EOF\_\_

   ![](https://github.com/weljoni)weljoni  - **本文链接：** [https://github.com/weljoni/p/18461552](https://github.com):[飞数机场](https://ze16.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
