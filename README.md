# 设计模式
## 原型模式
### 示例
原型模式不关心对象的具体类型，而是找到一个对象，然后通过克隆来创建一个一摸一样的对象。
```javascript
class Plane = {
  constructor(blood = 100, attackLevel = -1, defenseLevel = -1) {
    this.blood = blood;
    this.attackLevel = attackLevel;
    this.defenseLevel = defenseLevel;
  }
}
const plane = new Plane(500)
```
```javascript
const Plane = function(blood = 100, attackLevel = -1, defenseLevel = -1) {
    this.blood = blood;
    this.attackLevel = attackLevel;
    this.defenseLevel = defenseLevel;
}
const plane = new Plane(500, 4, 5)
// 通过克隆创建一个一摸一样的对象
const clonePlane = Object.create(plane)
console.log(clonePlane.blood)  // 500
console.log(clonePlane.attackLevel)  // 4
console.log(clonePlane.defenseLevel)  // 5
```
原型模式的规则：
- 所有的数据都是对象。
- 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它。
- 对象会记住它的原型。
- 如果对象无法响应某个请求，他会吧这个请求委托给它自己的原型。
## 单例模式
> 单例模式的定义是：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

下面是一个普通单例的例子：
```javascript
var Person = function (name) {
    this.name = name
    this.instance = null
}
Person.prototype.getName = function () {
    return this.name
}
Person.getInstance = function (name) {
    if (!this.instance) {
        this.instance = new Person(name)
    }
    return this.instance
}
var a = Person.getInstance('sven')
var b = Person.getInstance('sven2')
console.log(a === b)  // true
```
上面单例每次都要靠 Person.getInstance() 来实现，使用的时候不怎么友好，因为我们创建实例的时候习惯直接使用new运算符，所以接下来出现一个改进版：
```javascript
var Person= (function () {
  var instance;
  var Person= function (name) {
    if (instance) {return instance}
    this.name = name
    return instance = this
  }
  Person.prototype.getName = function(){
    return this.name
  }
  return Person
})()
var a = new Person('yy')
var b = new Person('zz')
console.log(a === b)  // true
```
上面这个例子还有一点问题，就是这个函数只能是单例了，如果我以后需要这个函数实现多例，那就要将Person中控制创建唯一对象的相关代码删除，这种修改会存在很多的隐患。
为了避免上述的隐患，我们继续来改进这个例子：
```javascript
var Person = function(name) {
  this.name = name
}
Person.prototype.getName = function () {return this.name}
// 使用一个代理来完成唯一对象的创建
var proxySingletonPerson = (function () {
  var instance;
  return function (name) {
    if (!instance) {
      instance = new Person(name)
    }
    return instance
  }
})()
var a = new proxySingletonPerson('yy')
var b = new proxySingletonPerson('zz')
console.log(a === b)  // true
```
观察代理函数会发现主要用于变成单例的部分是这个模式
```javascript
var obj
if (!obj) {
  obj = xxxx
}
```
所以我们可以再抽象出一层：
```javascript
  var Person = function(name) {
    this.name = name
  }
  Person.prototype.getName = function() {
    return this.name
  }
  var getSingle = function(fn) {
    var result;
    return function() {
      return result || (result = new fn(arguments))
    }
  }
  var singletonPerson = getSingle(Person)
  var a = new singletonPerson('a')
  var b = new singletonPerson('bbb')
  console.log(a === b)  // true
```
## 策略模式
> 策略模式的定义是：定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。

下面是一个策略模式的例子：
表单提交的时候进行校验，使用策略类让校验类更容易维护
```javascript
  const strategies = {
    isNonEmpty(value, errmsg) {
      if(value === '') return errmsg
    },
    minLength(value, len, errmsg) {
      if (value.length < len) return errmsg
    },
    isMobile(value, errmsg) {
      if( !/(^1[3|5|8][0-9]{9}$)/.test(value)) return errmsg
    }
  }
  class Validator {
    constructor() {
      this.cache = []
    }
    add(dom, rule, errmsg) {
      this.cache.push(function() {
        // 单条校验过程
      })
    }
    start() {
      while(this.cache.length) {
        const rule = this.cache.shift()
        // 参数逐个进行校验
      }
    }
  }
```

## 代理模式
> 代理模式是为对象提供一个代用品或占位符，以便控制对它的访问。

代理模式有多种形式，保护代理、虚拟代理、缓存代理、防火墙代理、远程代理等等。在JavaScript中，一般比较常用的是虚拟代理和缓存代理。
**虚拟代理：把一些开销很大的对象，延迟到真正需要它的时候才去创建。**
下面看一个虚拟代理的例子：
```javascript
  const myImage = (function() {
    const imgNode = document.createElement('img')
    document.body.appendChild(imgNode)
    return {
      setSrc: function(src) {
        imgNode.src = src
      }
    }
  })()
  const proxyImage = (function() {
    const img = new Image
    img.onload = function() {
      myImage.setSrc(this.src)
    }
    return {
      setSrc: function(src) {
        myImage.setSrc('loading.gif')
        img.src = src
      }
    }
  })()
  proxyImage.setSrc('pucture.png')
```
上面这个例子中，使用的是图片预加载技术，先在proxyImage中加载需要的图片，然后让myImage显示loading图片，等需要的图片完全加载完毕后，再替换掉loading图片。
那为什么要将两个功能分开，它们应该可以合并为一个函数才对。这是因为单一职责原则（就一个类而言，应该仅有一个引起它变化的原因），面向对象设计中，一个对象承担的职责越多，那引起它变化的原因就会变得越多，当变化发生时，设计可能会遭到意外的破坏。
两个函数的使用都是setSrc接口，所以这样当不需要预加载的时候，可以直接请求对象本体，理解与修改也非常方便。
虚拟代理还有合并http请求、让文件实现惰性加载等能力。（惰性加载实例见JavaScript设计模式与开发实践6.7节）
**缓存代理可以为一些开销大的运算结果提供暂时的存储，在下次运算时，如果传递进来的参数跟之前一致，则可以直接返回前面存储的运算结果。**
```javascript
  var mult = function() {
    console.log('开始计算乘积')
    var a = 1
    for (var i = 0, l = arguments.length; i < l; i++) {
      a = a * arguments[i]
    }
    return a
  }
  var proxyMult = (function() {
    var cache = {}
    return function() {
      var args = Array.prototype.join.call(arguments, ',')
      if (args in cache) {
        return cache[args]
      }
      return cache[args] = mult.apply(this, arguments)
    }
  })()
  console.time('a')
  proxyMult(1, 2, 3, 4, 5)
  console.timeEnd('a')  // 3ms以上
  console.time('b')
  proxyMult(1, 2, 3, 4, 5)
  console.timeEnd('b')  // 1ms以下
```
上例中，计算结果被缓存，所以相同参数传入后不需要再进行计算直接读取就可以了，所以比较节约时间。

## 迭代器模式
> 迭代器模式是指提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。迭代器模式可以把迭代的过程从业务逻辑中分离出来，在使用迭代器模式之后，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素。

按照上面的定义，数组的forEach、reduce、reduceRight、filter这些应该都属于迭代函数，首先我们实现一个forEach函数
```javascript
  var each = function(arr, fn) {
    var l = arr.length,
      i = 0;
    for (; i < l; i++) {
      fn(arr[i], i, arr)
    }
  }
  each([1,2,3,4,5], function (e) {
    console.log(e)
  })
```
each属于内部迭代器，因为each函数的内部已经定义好了迭代规则，就是对一个数组内容进行遍历并分别调用函数fn。如果需要对两个数组进行处理，那么就需要对迭代规则进行修改。
下面来看一个外部迭代器：
```javascript
    /* 迭代器主体 */
    var Iterator = function (obj) {
      var current = 0
      var next = function () {
        current += 1
      }
      var getCurrItem = function () {
        return obj[current]
      }
      var isDone = function () {
        return current >= obj.length
      }
      return {
        next: next,
        getCurrItem: getCurrItem,
        isDone: isDone
      }
    }
    var each = function (iterator) {
      while(!iterator.isDone()) {
        /* 这里是循环遍历的主题，也就是一般each函数中自定义的fn */
        console.log(iterator.getCurrItem())
        iterator.next()
      }
    }
    var iterator1 = Iterator([1,2,3,4,5])
    each(iterator1)
```
## 发布-订阅模式（观察者模式）
> 发布-订阅模式又叫观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。在JavaScript中，我们一般用事件模型来替代传统的发布-订阅模式。

发布-订阅模式需要哪些东西：
- 订阅者列表
- 订阅的方法
- 发布的方法

```javascript
var event = {
  // 消息列表
  clientList: [],
  // 订阅消息
  $on: function(key, fn) {
    if (!this.clientList[key]) {
      this.clientList[key] = []
    }
    this.clientList[key].push(fn)
  },
  // 发布消息
  $emit: function() {
    var key = Array.prototype.shift.call(arguments),
      fns = this.clientList[key]
    if (!fns || fns.length === 0) {
      return false
    }
    for (var i = 0, fn; fn = fns[i++]) {
      fn.apply(this, arguments)
    }
  }
}
// 让对象拥有这些方法
function installEvent(obj) {
  for ( var i in event) {
    obj[i] = event[i]
  }
}

var demo = {}
installEvent(demo)
demo.$on('milk', function (num) {
  console.log('牛奶新到：' + num)
})
demo.$on('milk2', function (num) {
  console.log('牛奶2新到：' + num)
})
demo.$emit('milk', 100)
demo.$emit('milk2', 120)
```
上面是发布-订阅模式简单的示例，订阅者设置需要的消息和回调函数，发布者在发布消息的时候触发相应消息中的函数并将参数传入。

## 命令模式
> 命令模式是最简单和优雅的模式之一，命令模式种的命令指的是一个执行某些特定事情的指令。
命令模式最常见的场景是：有时候需要向某些对象发送请求，但是并不知道请求的接收者是谁也不知道被请求的操作是上面。此时希望用一种松耦合的方式来设计程序，使得请求发送者和请求接收者能够清除彼此之间的耦合关系。

```html
  <button id="execute">点击</button>
  <button id="undo">点击</button>
  <script>
  var Tv = {
    open: function() { console.log('打开电视机') },
    close: function() { console.log('关闭电视机') }
  }
  // 将命令封装成相应的对象
  var OpenTvCommand = function(receiver) { this.receiver = receiver; }
  OpenTvCommand.prototype.execute = function() { this.receiver.open() }
  OpenTvCommand.prototype.undo = function() { this.receiver.close() }

  // 设置命令的请求者需要执行的命令
  var setCommand = function(command) {
    execute.onclick = function() { command.execute() }
    undo.onclick = function() { command.undo() }
  }
  // 将命令的请求者和接收者结合在一起
  setCommand(new OpenTvCommand(Tv))
  </script>
```

**命令模式有时候和策略模式很像，区别是策略模式中所有策略的目标都是相同的，只是内部的“算法”有区别，但命令模式中的命令并不是针对一个目标，命令模式能完成更多的功能。**

## 组合模式
> 组合模式可以让我们使用树形方式创建对象的结构。我们可以把相同的操作应用在组合对象和单个对象上。在大多数情况下，我们都可以忽略掉组合对象和单个对象之间的差别，从而用一致的方式来处理它们。

```javascript
  // 普通命令
  var closeDoorCommand = {
    execute: function() {
      console.log('关门')
    }
  }
  var openPcCommand = {
    execute: function() {
      console.log('开电脑')
    }
  }
  var openQQCommand = {
    execute: function() {
      console.log('登录QQ')
    }
  }

  // 组合命令函数
  var MacroCommand = function() {
    return {
      commandsList: [],
      add: function(command) {
        this.commandList.push(command)
      },
      execute: function() {
        for (var i = 0, command; command = this.commandsList[i++];) {
          command.execute()
        }
      }
    }
  }

  // 组合命令
  var macroCommand = MacroCommand()

  macroCommand.add(closeDoorCommand)
  macroCommand.add(openPcCommand)
  macroCommand.add(openQQCommand)

  macroCommand.execute()
```
观察上例可以发现：我们通过一个函数，将多个命令组合成了对象，并且**单条命令的执行方式和组合命令的执行方式相同**；组合命令的结构类似于**树形结构**，而且**执行的顺序可以看做是对树深度优先的搜索**。

组合模式的使用场景一般有两种：
1. 表示对象的部分-整体层次结构。组合模式可以方便地构造一棵树来表示对象的部分-整体结构。
2. 客户希望统一对待树中的所有对象。

## 模板方法模式
> 模板方法模式由两部分结构组成，第一部分是抽象父类，第二部分是具体的实现子类。通常在抽象父类中封装了子类的算法框架，包括实现一些公共方法以及封装子类中所有方法的执行顺序。子类通过继承这个抽象类，也继承了整个算法结构，并且可以选择重写父类的方法。

抽象类：在Java中，类分为具体类和抽象类两种。具体类可以被实例化，抽象类不能被实例化。因为抽象类不能被实例化，所以抽象类一定是用来被某些具体类继承的。
标准的模板方法实现见JavaScript设计模式与开发实践11.2节。
适合JS的实现：
```javascript
  var Beverage = function(param) {
    var boilWater = function() {
      console.log('把水煮沸')
    }
    var brew = param.brew || function() {
      throw new Error('必须传递brew方法')
    }
    var pourInCup = param.pourInCup || function() {
      throw new Error('必须传递pourInCup方法')
    }
    var addCondiments = param.addCondiments || function() {
      throw new Error('必须传递addCondiments方法')
    }

    var F = function() {}
    F.prototype.init = function() {
      boilWater()
      brew()
      pourInCup()
      addCondiments()
    }
    return F
  }

  var Coffee = Beverage({
    brew: function() {
      console.log('用沸水冲泡咖啡')
    },
    pourInCup: function() {
      console.log('把咖啡倒进被子')
    },
    addCondiments: function() {
      console.log('加糖和牛奶')
    }
  })

  var Tea = Beverage({
    brew: function() {
      console.log('用沸水浸泡茶叶')
    },
    pourInCup: function() {
      console.log('把茶倒进被子')
    },
    addCondiments: function() {
      console.log('加柠檬')
    }
  })

  var coffee = new Coffee()
  coffee.init()

  var tea = new Tea()
  tea.init()
```
在上例中，Beverage 是抽象类，Coffee和Tea是具体类，Beverage中的F.init()封装了子类的算法框架，指导子类以何种顺序执行哪些方法，所以F.init()是模板方法。Beverage 将Coffee和Tea中都要执行的烧水过程确定，并定义好了Coffee和Tea中具体方法的执行顺序，也就是将两个类中的通用部分进行封装，提高了函数的扩展性。

## 享元模式
>     )模式是一种用于性能优化的模式，“fly”在这里是苍蝇的意思，意为蝇量级。享元模式的核心是运用共享技术来有效支持大量细粒度的对象。

使用享元模式的关键是如何区别内部状态和外部状态。可以被对象共享的属性通常被划分为内部状态，而外部状态取决于具体的场景，并根据场景而变化。

内部状态和外部状态：
- 内部状态存储与对象内部。
- 内部状态可以被一些对象共享。
- 内部状态独立于具体的场景，通常不会改变。
- 外部状态取决于具体的场景，并根据场景而变化，外部状态不能被共享。

```javascript
  var Upload = function(uploadType) {
    this.uploadType = uploadType
  }

  Upload.prototype.delFile = function(id) {
    uploadManager.setExternalState(id, this)

    if (this.fileSize < 3000) {
      return this.dom.parentNode.removeChild(this.dom)
    }
    if (window.confirm('确定要删除该文件吗？' + this.fileName)) {
      return this.dom.parentNode.removeChild(this.dom)
    }
  }
  // 工厂模式进行对象实例化，根据内部对象的数量来创建出新对象
  var UploadFactory = (function() {
    // 保存对象
    var createdFlyWeightObjs = {}
    return {
      create: function(uploadType) {
        if (createdFlyWeightObjs[uploadType]) {
          return createdFlyWeightObjs[uploadType]
        }
        return createdFlyWeightObjs[uploadType] = new Upload(uploadType)
      }
    }
  })()
  // 管理器封装外部状态
  var uploadManager = (function() {
    // 用于保存外部状态
    var uploadDatabase = {}
    return {
      add: function(id, uploadType, fileName, fileSize) {
        var flyWeightObj = UploadFactory.create(uploadType)

        var dom = document.createElement('div')
        dom.innerHTML = '<span>文件名称：' + fileName + '文件大小：' + fileSize + '</span>' + '<button class="delFile">删除</button>'
        dom.querySelector('.delFile').onclick = function() {
          flyWeightObj.delFile(id)
        }
        document.body.appendChild(dom)
        uploadDatabase[id] = {
          fileName: fileName,
          fileSize: fileSize,
          dom: dom
        }
        return flyWeightObj
      },
      // 通过id来返回对应的外部状态
      setExternalState: function(id, flyWeightObj) {
        var uploadData = uploadDatabase[id]
        for (var i in uploadData) {
          flyWeightObj[i] = uploadData[i]
        }
      }
    }
  })()

  var id = 0

  window.startUpload = function(uploadType, files) {
    for (var i = 0, file; file = files[i++];) {
      var uploadObj = uploadManager.add(++id, uploadType, file.fileName, file.fileSize)
    }
  }

  startUpload('plugin', [{
      fileName: '1.txt',
      fileSize: 1000
    },
    {
      fileName: '2.txt',
      fileSize: 3000
    },
    {
      fileName: '3.txt',
      fileSize: 5000
    },
  ])
  startUpload('flash', [{
      fileName: '4.txt',
      fileSize: 1000
    },
    {
      fileName: '5.txt',
      fileSize: 3000
    },
    {
      fileName: '6.txt',
      fileSize: 5000
    },
  ])
```
## 职责链模式
> 职责链模式的定义是：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系，将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。
职责链模式的名字非常形象，一系列可能会处理请求的对象被连接成一条链，请求在这些对象之间依次传递，直到遇到一个可以处理它的对象，我们把这些对象成为链中的节点。

举例：我口袋有一些钱，想买饮料，当在10块以下的时候，买矿泉水；当20块以下的时候，买奶茶；当30块以下的时候，买咖啡。
```javascript
  var drink = function(money) {
    if (money < 10) {
      console.log('矿泉水')
    } else if (money < 20) {
      console.log('奶茶')
    } else {
      console.log('咖啡')
    }
  }
```
通过上面这个函数，我们能得到正确的结果，但这段代码并不值得称赞，当条件变化的时候，我们需要不断修改这个函数，会导致越来越难维护。所以接下来我们使用职责链模式来处理这件事情：
```javascript
  /* 将不同条件分开 */
  var drinkWater = function(money) {
    if (money > 0 && money <= 10) {
      console.log('矿泉水')
    } else {
      return 'nextSuccessor'
    }
  }
  var drinkMilkTea = function(money) {
    if (money > 10 && money <= 20) {
      console.log('奶茶')
    } else {
      return 'nextSuccessor'
    }
  }
  var drinkCoffee = function(money) {
    if (money > 20 && money <= 50) {
      console.log('咖啡')
    } else {
      return 'nextSuccessor'
    }
  }

  /* 定义职责链传递的方式 */
  var Chain = function(fn) {
    this.fn = fn
    this.successor = null
  }
  Chain.prototype.setNextSuccessor = function(successor) {
    return this.successor = successor
  }
  Chain.prototype.passRequest = function() {
    var ret = this.fn.apply(this, arguments)
    if (ret === 'nextSuccessor') {
      return this.successor && this.successor.passRequest.apply(this.successor, arguments)
    }
    return ret
  }

  /* 开始使用 */
  var chainWater = new Chain(drinkWater)
  var chainMilkTea = new Chain(drinkMilkTea)
  var chainCoffee = new Chain(drinkCoffee)

  chainWater.setNextSuccessor(chainMilkTea)
  chainMilkTea.setNextSuccessor(chainCoffee)

  chainWater.passRequest(4) // 矿泉水
  chainWater.passRequest(14) // 奶茶
  chainWater.passRequest(24) // 咖啡
```
可以看出函数干净了很多，现在要修改饮料种类的话就不用管原来函数的内容是什么，直接增加一个函数然后添加到职责链里面就好。
接下来再看一个比较方便的实现方式：
```javascript
  Function.prototype.after = function(fn) {
    const self = this
    return function() {
      const ret = self.apply(this, arguments)
      if (ret === 'nextSuccessor') {
        return fn.apply(this, arguments)
      }
      return ret
    }
  }
  const drinkWater = function(money) {
    if (money > 0 && money <= 10) {
      console.log('矿泉水')
    } else {
      return 'nextSuccessor'
    }
  }
  const drinkMilkTea = function(money) {
    if (money > 10 && money <= 20) {
      console.log('奶茶')
    } else {
      return 'nextSuccessor'
    }
  }
  const drinkCoffee = function(money) {
    if (money > 20 && money <= 50) {
      console.log('咖啡')
    } else {
      return 'nextSuccessor'
    }
  }
  const drink = drinkWater.after(drinkMilkTea).after(drinkCoffee)
  drink(20) // 奶茶
```
after的实现使用了递归的思想，类似于二叉树搜索的深度优先，将需要执行的函数按顺序排列好，再开始逐个执行调用。

## 中介者模式
> 中介者模式的作用就是解除对象与对象之间的紧耦合关系。增加一个中介者对象后，所有的相关对象都通过中介者对象来通信，而不是互相引用，所以当一个对象发生改变时，只需要通知中介者对象即可。中介者使各对象之间耦合松散，而且可以独立地改变它们之间的交互。中介者模式使网状的多对多关系变成了相对简单的一对多关系。

中介者模式和发布-订阅模式很像，区别就是发布-订阅模式中发布者只能发布消息，订阅者只能接收消息，中介者模式中，每个模块都能够对中介者发布消息，也能从中介者中接收到消息。

下面看一个例子：
微信群聊中，所有人都能发送消息，也能接收消息，一个人发送消息的时候，其他人就会收到消息通知。
```javascript
  // 创建用户相关方法
  function User(name) {
    this.name = name
  }
  User.prototype.receiveMessage = function(people) {
    console.log(this.name + '收到消息：' + people.name + "说: " + people.msg)
  }
  User.prototype.sendMessage = function(msg) {
    this.msg = msg ? msg : ''
    wechart.ReceiveMessage('sendMessage', this)
  }
  User.prototype.leaveRoom = function() {
    wechart.ReceiveMessage('leaveRoom', this)
  }

  // 创建中介者
  var wechart = (function() {
    var users = [],
      operations = {}
    operations.addUser = function(user) {
      var isRepeat = users.some(function(u) {
        return u.name === user.name
      })
      if (!isRepeat) {
        users.push(user)
        console.log(user.name + " 加入群聊")
      }
    }
    operations.sendMessage = function(sender) {
      users.forEach(function(user) {
        if (user.name !== sender.name) {
          user.receiveMessage(sender)
        }
      })
    }
    operations.leaveRoom = function(user) {
      var i = 0,
        l = users.length
      for (; i < l; i++) {
        if (users[i].name === user.name) {
          users.splice(i, 1)
          console.log('系统：' + user.name + " 退出群聊")
          break
        }
      }
    }
    var ReceiveMessage = function() {
      var eventName = Array.prototype.shift.call(arguments)
      operations[eventName].apply(this, arguments)
    }
    return {
      ReceiveMessage: ReceiveMessage
    }
  })()

  //  通过工厂函数来创建用户
  var chartRoom = function(name) {
    var newUser = new User(name)
    wechart.ReceiveMessage('addUser', newUser)
    return newUser
  }

  var user1 = chartRoom('小明')
  var user2 = chartRoom('小红')
  var user3 = chartRoom('小强')
  var user4 = chartRoom('小刚')

  user1.sendMessage('hello')  //  小红收到消息：小明说：hello    
                              //  小强收到消息：小明说：hello   
                              //  小刚收到消息：小明说：hello    
  user1.leaveRoom()  //  系统：小明 退出群聊
```
在上例中，wechart就是中介者对象，它负责接收消息并分发给所有用户，它的存在，让用户和用户之间没有了耦合关系，用户要做什么，只要通知中介者，中介者就能处理完消息后把结果返回给其他的玩家对象。从这里可以明显看出，中介者模式和订阅发布模式有些类似，但还是不一样的。

中介者模式的优点是非常方便地对模块或者对象进行解耦，但有个缺点就是随着功能的逐渐复杂，中介者会变得越来越庞大，也就会变得越来越难以维护。

## 装饰者模式
> 装饰者模式能够在不改变对象自身的基础上，在程序运行期间给对象动态地添加职责。
```javascript
const before = function(fn, beforefn) {
  return function() {
    beforefn.apply(this, arguments)
    return fn.apply(this, arguments)
  }
}
```
### 示例
---
