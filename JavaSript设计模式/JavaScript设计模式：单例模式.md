# JavaScript设计模式：单例模式

## 开篇语

一说起设计模式，对于很多JavaScript开发者来说就是“脑阔疼”，认为这种“高端”的东西是可望而不可及，甚至有人觉得就算没有这些设计模式，也一样不影响项目的开发。

其实，设计模式并非凭空出现的理论，GoF在《设计模式》一书中对设计模式的定义是：`在面向对象程序设计过程中针对特定问题的简洁而优雅的解决方案。`简言之，设计模式是能够在不同的场合中针对某种问题而使用的解决方案。因此，学习设计模式有以下几个好处：

（1）解决实际项目中遇到的问题

（2）提升程序设计的抽象能力

（3）提升代码阅读的能力

（4）提升甄别优劣代码的能力

最后，希望本文能够给予你一定的启发及帮助，也希望你以一种积极的态度拥抱设计模式，那将受益良多。

## 1. 什么是单例模式

单例模式是一种简单且常用的设计模式，该模式规定了一个类必须保证只有一个一个实例，这意味着第二次使用同一个类创建新对象的时候，应该与第一个实例对象完全相同。因此也就不难明白为什么该模式名为单例：单一（唯一）的实例。

### 1.1. 使用“类”的方式

JavaScript是弱类型语言，所以并没有“类”的概念，但是我们经常会用new关键字操作构造函数来创建一个对象：

```javascript
var Person = function (name) {
  this.name = name;
};

var p1 = new Person('Neil-1');
var p2 = new Person('Neil-2');
console.log(p1 === p2);    // false
```

以上就是一个简单的实例创建过程，分别通过构造函数new了p1和p2两个实例，这两个实例都有一个name的属性。但是这两个实例是不相等的，这是因为它们引用的地址是不相等的，因此也就没有实现单例的目标（一个类返回的对象永远是同一个引用）。

那么为了实现这一目标，我们需要对这个构造函数进行改造：

```javascript
var Person = function (name) {
  // 判断是否有这个实例
  if (typeof Person.instance === 'object') {
    return Person.instance;
  }

  this.name = name;
  this.instance = null
  // 缓存实例
  Person.instance = this;
}

var p1 = new Person('Neil-1');
var p2 = new Person('Neil-2');
console.log(p1 === p2);   // true
```

在以上的代码中，构造函数Person新增了instance属性，并在每次新建实例的时候都会先判断是否存在这个属性，最后在每次创建的实例都会指向同一个引用，无论创建了多少个实例都是会返回第一个实例对象。这就实现了一个最简单的单例模式了。

#### 1.1.1. 使用闭包实现“透明”的单例模式

前面已经实现了一种简单的单例模式，但这种实现方式有一个不足，那就是instance属性是暴露在全局作用域上的，也就存在被无意修改的可能。那么，下面我们再一次优化一下这段代码：

```javascript
var Person = (function () {
  var instance;
  // 重写Person，这才是真正的构造函数
  var Person = function (name) {
    if (instance) {
      return instance;
    }
    this.name = name;
    return instance = this;
  };
  return Person;
})();

var p1 = new Person('Neil-1');
var p2 = new Person('Neil-2');
console.log(p1 === p2);   // true
```

在上述代码中，我们使用了自执行函数和闭包，让instance变成了私有变量，这样，调用者就无法直接修改instance。此处最核心的地方是在自执行函数中重写了构造函数，当调用者使用new操作符创建一个实例的时候，实际上是调用了内层的构造函数。

在这段代码中，仍存有两点不合理的地方：

（1）通用性差，当需要改写Person构造函数的时候，需要移除整块重写部分；

（2）可阅读性差，这种写法明显与我们传统的构造函数大相径庭。

#### 1.1.2. 使用代理类实现单例模式

此处，我们通过代理类（代理模式）的方式来完善上面的代码。

```javascript
var Person = function () {
  this.name = 'Neil';
};

var Singleton = (function (Fn) {
  var instance;
  return function (name) {
    if (!instance) {
      instance = new Fn(name);
    }
    return instance;
  }
})(Person);

var p1 = new Singleton('Neil-1');
var p2 = new Singleton('Neil-2');
console.log(p1 === p2);   // true
```

从上述的代码中，可以明显看出更加清晰简洁。Person构造函数放在处理单例的代码外，并作为实参传入自执行函数的形参中，这样，当需要修改构造函数Person的功能时，无须对单例代码进行改动，只需要修改或替换构造函数即可。

### 1.2. 使用对象的方式

前面提到JavaScript并没有“类”的概念，所谓的构造函数，无非就是使用约定俗成的大写字母开头（Person）作为函数名的方法，而创建的实例，则是借用了new操作符的功能特性，返回了一个引用对象。对于JavaScript开发者来说，使用“类”的方式来实现单例模式，并非我们的常规武器，那我们还能怎么去实现呢？我们不妨再从定义入手：

  `一个类必须保证只有一个实例`

实例即对象，那么，在JavaScript的世界里，我们就可以通过对象字面量的方式创建一个对象：

```javascript
var obj = {
  name: 'Neil'
};
var obj2 = {
  name: 'Neil'
};

console.log(obj === obj2);   // false
```

无论对象中的成员是否相同，两个对象一定不会相同，因此我们可以认为，每次通过对象字面量创建的对象都是一个单例。

虽然创建对象就是创建了单例，但是这种全局变量的方式会污染整个全局作用域，如果有多个全局变量的创建，那简直就是灾难级的。因此我们需要完善这种创建单例的方式。

#### 1.2.1. 使用命名空间

命名空间的方式也就是将多个对象放入到同一个对象里面，虽然使用命名空间也是在全局作用域创建了对象，但是这种代价已经大大地减少了污染的数量。以下是命名空间空间的基本使用：

```javascript
// 在使用该命名空间前，先判断这个名称是否被使用
if (typeof MYAPP !== 'undefined') {
  var MYAPP = {
    obj: function (name) {
      console.log(name);
    },
    obj2: function (name) {
      console.log(name);
    }
  };
}
```

上述代码在创建命名空间前进行了预检，保证了同名的命名空间不被污染。但如果我们存在多级命名空间时（如MYAPP.module1.module2），我们的预检代码就会变得很长很臃肿，这显然不是我们想要的，所以我需要实现一种可以动态创建命名空间的方法。

```javascript
// 对最外层的命名空间预检
var MYAPP = MYAPP || {}

MYAPP.namescape = function (name) {
  var parts = name.split('.');
  var current = MYAPP;
  for (var i in parts) {
    if (typeof current[parts[i]] === 'undefined') {
      current[parts[i]] = {};
    }
    current = current[parts[i]];
  }
};

MYAPP.namescape('Person.Man.Old');
```

#### 1.2.2. 使用闭包

这种方式和与前面的1.1.1种提到的方式类似，都是使用了自执行函数和闭包，建立私有变量，只暴露特定的方法供外部使用：

```javascript
var Person = (function () {
  // 定义私有属性
  var name = 'Neil';
  // 定义私有方法
  var showName = function (name) {
    console.log(name)
  }

  // 暴露外部的公共API
  return {
    getName: function () {
      showName(name)
    }
  }
})();
```

#### 1.2.3. 使用模块

我们还可以结合上面两种方法：

```javascript
// 假设定义了命名空间的函数
MYAPP.namescape('Person.Man.Old');
Person.Man.Old = (function () {
  // 私有属性
  var name = 'Neil';
  var age = 60;
  // 私有方法
  var getName = function () {
    console.log(name)
  }
  var getAge = function () {
    console.log(age)
  }
  // 公有API，唯一的出口
  return {
    geName: getName,
    getAge: getAge
  }
})();
```

也许乍看之下你会觉得这种方式好像跟前面的方式差异不大，但是在大型应用中或者库中，这种方式的优势就尤为明显。使用模块的方式可以在命名空间内定义私有属性和私有方法，并设定唯一的出口，这样就避免了命名重复（导致同名方法覆盖）等问题，同时让代码更加清晰整洁。

## 2. 惰性单例

在单例模式的应用中，大部分时候并不需要在页面加载的时候就创建好对象，而是等到需要的时候再来创建，那么此时我们就需要使用惰性单例。

```javascript
var Person = function (name) {
  this.name = name;
  this.instance = null;
};
Person.getInstance = function (name) {
  if (!this.instance) {
    this.instance = new Person(name);
  }
  return this.instance;
};

var p = Person.getInstance('Neil')
```

以上就是一个简单的惰性单例，只有当需要的时候才会去调用`Person.getInstance()`方法。当然，这个示例的实际作用并不大，具体的应用我们会在第四节中展示。

## 3. ES6中的单例模式

前面我们提到，JavaScript是没有“类”的概念的，但自从ES6出现后，我们也可以使用和其他静态类型语言一样的方法去创建单例。那要如果去实现呢？我们又双叒叕要从单例模式的定义入手了，以下是来自维基百科的定义：

```
实现单例模式的思路是：一个类能返回对象一个引用(永远是同一个)和一个获得该实例的方法（必须是静态方法，通常使用getInstance这个名称）；当我们调用这个方法时，如果类持有的引用不为空就返回这个引用，如果类保持的引用为空就创建该类的实例并将实例的引用赋予该类保持的引用；同时我们还将该类的构造函数定义为私有方法，这样其他处的代码就无法通过调用该类的构造函数来实例化该类的对象，只有通过该类提供的静态方法来得到该类的唯一实例。
```

由上可知，实现单例需要满足以下几个条件：

（1）一个类只能返回同一个引用；

（2）有一个getInstance的静态方法；

（3）构造函数必须为私有方法。

实现的思路已经非常清晰了，现在我们就可以实现ES6的单例模式：

```javascript
class Person {
  // 构造函数
  constructor (name) {
    this.name = name;
  }

  // 静态方法getInstance
  static getInstance = (name) => {
    if (!this.instance) {
      this.instance = new Person(name);
    }
    return this.instance;   // 返回一个引用，唯一出口
  };
}

let p1 = Person.getInstance('Neil-1');
let p2 = Person.getInstance('Neil-2');
console.log(p1 === p2)  // true
```

上述代码中完全满足了实现单例模式的3个条件，并在创建的时候返回了唯一的示例。

除了使用类的方式，我们同样也可以通过ES6的特性来实现：

```javascript
const Person = {
  name: 'Neil',
  getName: name => this.name = name
};

Object.freeze(Person);
export default Person; 
```

在上述的代码中，我们使用了几个来自ES6的小技巧，从而使得我们的代码变得简洁：

（1）使用了const，Person变量不能重新被覆盖；

（2）使用了Object.freeze()这个方法，冻结了Person对象，使其不能新增和改变现有的方法和属性；

（3）使用了ES6的模块化，能够准确地知道Person在哪里被调用了，并且降低了全局变量污染的可能。

## 4. 单例模式在项目中的应用

在前面的章节中，我们通过Person这个例子讲解了如何使用单例模式，但在实际的项目中并非最常用的，下面我将以一个常见的提示框为例，介绍单例模式在项目中的应用。

假设我们的项目中有个确认登录的弹窗，当用户没有登录的时候，就会弹出以下提示弹框。显然这个弹框在整个页面中有且只需要一个。

![https://github.com/Neil-WN/MarkdownPhotos/blob/master/images/singleton-example.png]()

通常我们会在页面加载完成的时候就先创建好这个弹窗，当用户点击请求数据按钮的时候，如果后端接口返回了未登录的信息，则这个弹窗就会显示，否则会一直隐藏。

```javascript
// 页面加载时就创建好弹窗
var warnModal = (function () {
  var div = document.createElement('div');
  div.innerHTML = '<p>你还未登录</p><button>确定</button>';
  div.style.display = 'none';
  document.body.appendChild(div);
  return div;
})();

// 我们借助jQuery库来发起ajax请求
$.get('http://www.stargraph.cn', function (data) {
  // 如果用户未登录，则显示弹窗
  if (!data.isLogin) {
    warnModal.style.display = 'block';
  }
});
```

在上面的这段代码中，我们可以看到弹窗是当页面加载完成后就创建好了，无论我们是否发起异步请求，这个弹窗都一直存在的，这样就会白白增加了DOM节点，如果页面中有很多类似的弹窗，那整个页面的初次加载有可能是灾难级的，因此我们可以改造一下代码：

```javascript
var createWarnModal = function () {
  var div = document.createElement('div');
  div.innerHTML = '<p>你还未登录</p><button>确定</button>';
  div.style.display = 'none';
  document.body.appendChild(div);
  return div;
};

// 我们借助jQuery库来发起ajax请求
$.get('http://www.stargraph.cn', function (data) {
  // 如果用户未登录，则创建并显示弹窗
  if (!data.isLogin) {
    var warnModal = createWarnModal();
    warnModal.style.display = 'block';
  }
});
```

我们此时已经完成了基本的业务需求，在后端接口返回未登录信息的时候创建并显示弹窗，但是此时并非我们想要的结果，即每次接口返回数据的时候，都会创建一个弹窗，这显然不是我们需要的单例，我们对以上代码再进行一次改造：

```javascript
var createWarnModal = (function () {
  var div = null;
  return function () {
    if (!div) {
      div = document.createElement('div');
      div.innerHTML = '<p>你还未登录</p><button>确定</button>';
      div.style.display = 'none';
      document.body.appendChild(div);
    }
    return div;
  }
})();

// 我们借助jQuery库来发起ajax请求
$.get('http://www.stargraph.cn', function (data) {
  // 如果用户未登录，则创建并显示弹窗
  if (!data.isLogin) {
    var warnModal = createWarnModal();
    warnModal.style.display = 'block';
  }
});
```

## 总结

在本文中，我们了解了单例模式的基本定义，并介绍了如何通过ES5、ES6实现单例模式，我们还通过一个弹窗的例子介绍了单例模式在项目中的简单应用。

单例模式是众多设计模式中最简单、最常用的模式之一，希望藉由本篇文章能够帮你消除对设计模式的恐惧，从最基础的模式带你入门设计模式。设计模式在我们的项目开发中也是扮演着重要的角色，灵活地运用单例模式能够让减少内存的开销，提升页面的性能和用户体验。