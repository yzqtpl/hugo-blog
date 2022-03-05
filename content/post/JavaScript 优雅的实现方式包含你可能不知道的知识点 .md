---
title: JavaScript 优雅的实现方式包含你可能不知道的知识点 
date: 2017-05-11 09:17:00
categories: [编程, 前端]
tags: [前端, js]
description:
---

# 前端

标签： 干货

# JavaScript 优雅的实现方式包含你可能不知道的知识点 

**有些东西很好用，但是你未必知道；有些东西你可能用过，但是你未必知道原理。**

实现一个目的有多种途径，俗话说，条条大路通罗马。很多内容来自平时的一些收集以及过往博客文章底下的精彩评论，收集整理拓展一波，发散一下大家的思维以及拓展一下知识面。

**茴字有四种写法，233333...， 文末有彩蛋有惊喜。**

### 1、简短优雅地实现 sleep 函数

很多语言都有 `sleep` 函数，显然 `js` 没有，那么如何能简短优雅地实现这个方法？

#### 1.1 普通版

```javascript
function sleep(sleepTime) {
	for(var start = +new Dat e; +new Date - start <= sleepTime;) {}
}
var t1 = +new Date()
sleep(3000)
var t2 = +new Date()
console.log(t2 - t1)
```

优点：简单粗暴，通俗易懂。
缺点：这是最简单粗暴的实现，确实 sleep 了，也确实卡死了，CPU 会飙升，无论你的服务器 CPU 有多么 Niubility。

#### 1.2 Promise 版本

```javascript
function sleep(time) {
  return new Promise(resolve => setTimeout(resolve, time))
}

const t1 = +new Date()
sleep(3000).then(() => {
  const t2 = +new Date()
  console.log(t2 - t1)
})
```

优点：这种方式实际上是用了 `setTimeout`，没有形成进程阻塞，不会造成性能和负载问题。
缺点：虽然不像 `callback `套那么多层，但仍不怎么美观，而且当我们需要在某过程中需要停止执行（或者在中途返回了错误的值），还必须得层层判断后跳出，非常麻烦，而且这种异步并不是那么彻底，还是看起来别扭。

#### 1.3 Generator 版本

```javascript
function sleep(delay) {
  return function(cb) {
    setTimeout(cb.bind(this), delay)
  };
}

function* genSleep() {
  const t1 = +new Date()
  yield sleep(3000)
  const t2 = +new Date()
  console.log(t2 - t1)
}

async(genSleep)

function async(gen) {
  const iter = gen()
  function nextStep(it) {
    if (it.done) return
    if (typeof it.value === "function") {
      it.value(function(ret) {
        nextStep(iter.next(ret))
      })
    } else {
      nextStep(iter.next(it.value))
    }
  }
  nextStep(iter.next())
}
```

优点：同 `Promise` 优点，另外代码就变得非常简单干净，没有 then 那么生硬和恶心。
缺点：但不足也很明显，就是每次都要执行 next() 显得很麻烦，虽然有 [**co**](https://github.com/tj/co)（第三方包）可以解决，但就多包了一层，不好看，错误也必须按 [**co**](https://github.com/tj/co) 的逻辑来处理，不爽。

**co 之所以这么火并不是没有原因的，当然不是仅仅实现`` sleep `这么无聊的事情，而是它活生生的借着generator/yield 实现了很类似 `async/await `的效果！这一点真是让我三观尽毁刮目相看。**

```javascript
const co = require("co")
function sleep(delay) {
  return function(cb) {
    setTimeout(cb.bind(this), delay)
  }
}

co(function*() {
  const t1 = +new Date()
  yield sleep(3000)
  const t2 = +new Date()
  console.log(t2 - t1)
})
```

#### 1.4 Async/Await 版本

```javascript
function sleep(delay) {
  return new Promise(reslove => {
    setTimeout(reslove, delay)
  })
}

!async function test() {
  const t1 = +new Date()
  await sleep(3000)
  const t2 = +new Date()
  console.log(t2 - t1)
}()
```

优点：同 Promise 和 Generator 优点。 Async/Await 可以看做是 Generator 的语法糖，Async 和 Await 相较于 * 和 yield 更加语义，另外各个函数都是扁平的，不会产生多余的嵌套，代码更加清爽易读。
缺点： ES7 语法存在兼容性问题，有 babel 一切兼容性都不是问题

至于 `Async/Await` 比 `Promise` 和 `Generator` 的好处可以参考这两篇文章：
[Async/Await 比 Generator 的四个改进点](https://juejin.im/post/59f9ce7a51882554f666220f)
[关于Async/Await替代Promise的6个理由](https://juejin.im/post/58ede4c1b123db43cc365551)

#### 1.5 不要忘了开源的力量

**在 javascript 优雅的写 sleep 等于如何优雅的不优雅，2333**

> 这里有 C++ 实现的模块：<https://github.com/ErikDubbelboer/node-sleep>

```javascript
const sleep = require("sleep")

const t1 = +new Date()
sleep.msleep(3000)
const t2 = +new Date()
console.log(t2 - t1)
```

优点：能够实现更加精细的时间精确度，而且看起来就是真的 sleep 函数，清晰直白。
缺点：缺点需要安装这个模块，`^_^`，这也许算不上什么缺点。

**从一个间简简单单的 sleep 函数我们就就可以管中窥豹，看见 JavaScript 近几年来不断快速的发展，不单单是异步编程这一块，还有各种各样的新技术和新框架，见证了 JavaScript 的繁荣。**

**你可能不知道的前端知识点：Async/Await是目前前端异步书写最优雅的一种方式**

### 2、获取时间戳

上面第一个用多种方式实现 `sleep` 函数，我们可以发现代码有 `+new Date()`获取时间戳的用法，这只是其中的一种，下面就说一下其他两种以及 `+new Date()`的原理。

#### 2.1 普通版

```javascript
var timestamp=new Date().getTime()
```

优点：具有普遍性，大家都用这个
缺点：目前没有发现

#### 2.2 进阶版

```javascript
var timestamp = (new Date()).valueOf()
```

> valueOf 方法返回对象的原始值(Primitive,'Null','Undefined','String','Boolean','Number'五种基本数据类型之一)，可能是字符串、数值或 bool 值等，看具体的对象。

优点：说明开发者原始值有一个具体的认知，让人眼前一亮。
缺点: 目前没有发现

#### 2.3 终极版

```javascript
var timestamp = +new Date()
```

优点：对 JavaScript 隐式转换掌握的比较牢固的一个表现
缺点：目前没有发现

现在就简单分析一下为什么 `+new Date()` 拿到的是时间戳。

**一言以蔽之，这是隐式转换的玄学，实质还是调用了 valueOf() 的方法。**

我们先看看 `ECMAScript` 规范对一元运算符的规范：

**一元+ 运算符**

一元 `+` 运算符将其操作数转换为 `Number` 类型并反转其正负。注意负的 `+0` 产生 `-0`，负的 `-0` 产生 `+0`。产生式 `UnaryExpression : - UnaryExpression` 按照下面的过程执行。

> 1. 令 expr 为解释执行 UnaryExpression 的结果 .
> 2. 令 oldValue 为 ToNumber(GetValue(expr)).
> 3. 如果 oldValue is NaN ，return NaN.
> 4. 返回 oldValue 取负（即，算出一个数字相同但是符号相反的值）的结果。

**``+new Date()`` 相当于`` ToNumber(new Date())``**

我们再来看看 `ECMAScript` 规范对 `ToNumber` 的定义：

[![img](https://camo.githubusercontent.com/496f833832f11f2260af76c7048bbef32b8c31dd/68747470733a2f2f706963342e7a68696d672e636f6d2f35302f76322d31336664393733303963383439653039396466303139363366383565323931335f68642e6a7067)](https://camo.githubusercontent.com/496f833832f11f2260af76c7048bbef32b8c31dd/68747470733a2f2f706963342e7a68696d672e636f6d2f35302f76322d31336664393733303963383439653039396466303139363366383565323931335f68642e6a7067)

我们知道 `new Date()` 是个对象，满足上面的 `ToPrimitive()`，所以进而成了 `ToPrimitive(new Date())`。

接着我们再来看看 `ECMAScript` 规范对 `ToPrimitive` 的定义，一层一层来，抽丝剥茧。

[![img](https://camo.githubusercontent.com/a9fbade449ef107129c103917c4af444717dbfd7/68747470733a2f2f706963342e7a68696d672e636f6d2f35302f76322d32396334376363623638323838336438623433616239663937303936623563335f68642e6a7067)](https://camo.githubusercontent.com/a9fbade449ef107129c103917c4af444717dbfd7/68747470733a2f2f706963342e7a68696d672e636f6d2f35302f76322d32396334376363623638323838336438623433616239663937303936623563335f68642e6a7067)

这个 `ToPrimitive` 刚开始可能不太好懂，我来大致解释一下吧：

**ToPrimitive(obj,preferredType)**

`JavaScript` 引擎内部转换为原始值 `ToPrimitive(obj,preferredType)` 函数接受两个参数，第一个 `obj` 为被转换的对象，第二个`preferredType` 为希望转换成的类型（默认为空，接受的值为 `Number` 或 `String`）

在执行 `ToPrimitive(obj,preferredType)` 时如果第二个参数为空并且 `obj` 为 `Date` 的实例时，此时 `preferredType` 会被设置为 `String`，其他情况下 `preferredType` 都会被设置为`Number` 如果 `preferredType`为 `Number`，`ToPrimitive` 执行过程如下：

> 1. 如果obj为原始值，直接返回；
> 2. 否则调用 obj.valueOf()，如果执行结果是原始值，返回之；
> 3. 否则调用 obj.toString()，如果执行结果是原始值，返回之；
> 4. 否则抛异常。

如果 `preferredType` 为 `String`，将上面的第2步和第3步调换，即：

> 1. 如果obj为原始值，直接返回；
> 2. 否则调用 obj.toString()，如果执行结果是原始值，返回之；
> 3. 否则调用 obj.valueOf()，如果执行结果是原始值，返回之；
> 4. 否则抛异常。

首先我们要明白 `obj.valueOf()` 和 `obj.toString()` 还有原始值分别是什么意思,这是弄懂上面描述的前提之一:

**toString 用来返回对象的字符串表示。**

```javascript
var obj = {};
console.log(obj.toString());//[object Object]

var arr2 = [];
console.log(arr2.toString());//""空字符串

var date = new Date();
console.log(date.toString());//Sun Feb 28 2016 13:40:36 GMT+0800 (中国标准时间)
```

**valueOf 方法返回对象的原始值，可能是字符串、数值或 bool 值等，看具体的对象。**

```javascript
var obj = {
  name: "obj"
}
console.log(obj.valueOf()) //Object {name: "obj"}

var arr1 = [1]
console.log(arr1.valueOf()) //[1]

var date = new Date()
console.log(date.valueOf())//1456638436303
如代码所示，三个不同的对象实例调用valueOf返回不同的数据
```

**原始值指的是 'Null','Undefined','String','Boolean','Number','Symbol' 6种基本数据类型之一，上面已经提到过这个概念，这里再次申明一下。**

最后分解一下其中的过程：`+new Date():`

> 1. 运算符 `new` 的优先级高于一元运算符 `+`，所以过程可以分解为：
>    var time=new Date();
>    +time
> 2. 根据上面提到的规则相当于：`ToNumber(time)`
> 3. `time` 是个日期对象，根据 `ToNumber` 的转换规则，所以相当于：`ToNumber(ToPrimitive(time))`
> 4. 根据 `ToPrimitive` 的转换规则：`ToNumber(time.valueOf())`，`time.valueOf()` 就是 原始值 得到的是个时间戳，假设 `time.valueOf()=1503479124652`
> 5. 所以 `ToNumber(1503479124652)` 返回值是 `1503479124652` 这个数字。
> 6. 分析完毕

**你可能不知道的前端知识点：隐式转换的妙用**

### 3、数组去重

注：暂不考虑`对象字面`量，`函数`等引用类型的去重，也不考虑 `NaN`, `undefined`, `null`等特殊类型情况。

> 数组样本：[1, 1, '1', '2', 1]

#### 3.1 普通版

`无需思考，我们可以得到 O(n^2) 复杂度的解法。定义一个变量数组 res 保存结果，遍历需要去重的数组，如果该元素已经存在在 res 中了，则说明是重复的元素，如果没有，则放入 res 中。`

```javascript
var a = [1, 1, '1', '2', 1]
function unique(arr) {
    var res = []
    for (var i = 0, len = arr.length; i < len; i++) {
        var item = arr[i]
        for (var j = 0, len = res.length; j < jlen; j++) {
            if (item === res[j]) //arr数组的item在res已经存在,就跳出循环
                break
        }
        if (j === jlen) //循环完毕,arr数组的item在res找不到,就push到res数组中
            res.push(item)
    }
    return res
}
console.log(unique(a)) // [1, 2, "1"]
```

优点： 没有任何兼容性问题，通俗易懂，没有任何理解成本
缺点： 看起来比较臃肿比较繁琐，时间复杂度比较高`O(n^2)`

#### 3.2 进阶版

```javascript
var a =  [1, 1, '1', '2', 1]
function unique(arr) {
    return arr.filter(function(ele,index,array){
        return array.indexOf(ele) === index//很巧妙,这样筛选一对一的,过滤掉重复的
    })
}
console.log(unique(a)) // [1, 2, "1"]
```

优点：很简洁，思维也比较巧妙，直观易懂。
缺点：不支持 `IE9` 以下的浏览器，时间复杂度还是`O(n^2)`

#### 3.3 时间复杂度为O(n)

```javascript
var a =  [1, 1, '1', '2', 1]
function unique(arr) {
    var obj = {}
    return arr.filter(function(item, index, array){
        return obj.hasOwnProperty(typeof item + item) ? 
        false : 
        (obj[typeof item + item] = true)
    })
}

console.log(unique(a)) // [1, 2, "1"]
```

优点：`hasOwnProperty` 是对象的属性(名称)存在性检查方法。对象的属性可以基于 `Hash` 表实现，因此对属性进行访问的时间复杂度可以达到`O(1)`;
`filter` 是数组迭代的方法，内部还是一个 `for` 循环，所以时间复杂度是 `O(n)`。
缺点：不兼容 `IE9` 以下浏览器，其实也好解决，把 `filter` 方法用 `for` 循环代替或者自己模拟一个 filter 方法。

#### 3.4 终极版

> 以 Set 为例，ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。

```javascript
const unique = a => [...new Set(a)]
```

优点：`ES6` 语法，简洁高效，我们可以看到，去重方法从原始的 `14` 行代码到 `ES6` 的 `1` 行代码，其实也说明了 `JavaScript` 这门语言在不停的进步，相信以后的开发也会越来越高效。
缺点：兼容性问题，现代浏览器才支持，有 `babel` 这些都不是问题。

**你可能不知道的前端知识点：ES6 新的数据结构 Set 去重**

### 4、数字格式化 1234567890 --> 1,234,567,890

#### 4.1 普通版

```javascript
function formatNumber(str) {
  let arr = [],
    count = str.length

  while (count >= 3) {
    arr.unshift(str.slice(count - 3, count))
    count -= 3
  }

  // 如果是不是3的倍数就另外追加到上去
  str.length % 3 && arr.unshift(str.slice(0, str.length % 3))

  return arr.toString()

}
console.log(formatNumber("1234567890")) // 1,234,567,890
```

优点：自我感觉比网上写的一堆 for循环 还有 if-else 判断的逻辑更加清晰直白。
缺点：太普通

#### 4.2 进阶版

```
function formatNumber(str) {

  // ["0", "9", "8", "7", "6", "5", "4", "3", "2", "1"]
  return str.split("").reverse().reduce((prev, next, index) => {
    return ((index % 3) ? next : (next + ',')) + prev
  })
}

console.log(formatNumber("1234567890")) // 1,234,567,890
```

优点：把 JS 的 API 玩的了如指掌
缺点：可能没那么好懂，不过读懂之后就会发出我怎么没想到的感觉

#### 4.3 正则版

```
function formatNumber(str) {
  return str.replace(/\B(?=(\d{3})+(?!\d))/g, ',')
}

console.log(formatNumber("123456789")) // 1,234,567,890
```

下面简单分析下正则`/\B(?=(\d{3})+(?!\d))/g`：

> 1. `/\B(?=(\d{3})+(?!\d))/g`：正则匹配边界`\B`，边界后面必须跟着`(\d{3})+(?!\d)`;
> 2. `(\d{3})+`：必须是1个或多个的3个连续数字;
> 3. `(?!\d)`：第2步中的3个数字不允许后面跟着数字;
> 4. `(\d{3})+(?!\d)`：所以匹配的边界后面必须跟着`3*n`（n>=1）的数字。

最终把匹配到的所有边界换成`,`即可达成目标。

优点：代码少，浓缩的就是精华
缺点：需要对正则表达式的位置匹配有一个较深的认识，门槛大一点

#### 4.4 API版

```
(123456789).toLocaleString('en-US')  // 1,234,567,890
```

如图，你可能还不知道 `JavaScript` 的 `toLocaleString` 还可以这么玩。

[![img](https://camo.githubusercontent.com/f2d3615a7c5c5c736a32f87cf21639551f86e882/68747470733a2f2f7773342e73696e61696d672e636e2f6c617267652f303036744e625277677931666d717434646f6d65346a3330783630636137376d2e6a7067)](https://camo.githubusercontent.com/f2d3615a7c5c5c736a32f87cf21639551f86e882/68747470733a2f2f7773342e73696e61696d672e636e2f6c617267652f303036744e625277677931666d717434646f6d65346a3330783630636137376d2e6a7067)

**还可以使用** [Intl对象 - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Intl)

> Intl 对象是 ECMAScript 国际化 API 的一个命名空间，它提供了精确的字符串对比，数字格式化，日期和时间格式化。Collator，NumberFormat 和 DateTimeFormat 对象的构造函数是 Intl 对象的属性。

```
new Intl.NumberFormat().format(1234567890) // 1,234,567,890
```

优点：简单粗暴，直接调用 API
缺点：[Intl兼容性](https://caniuse.com/#search=Intl)不太好，不过 toLocaleString的话 IE6 都支持

**你可能不知道的前端知识点：Intl对象 和 toLocaleString的方法。**

### 5、交换两个整数

let a = 3,b = 4 变成 a = 4, b = 3

#### 5.1 普通版

首先最常规的办法，引入一个 temp 中间变量

```
let a = 3,b = 4
let temp = a
a = b
b = temp
console.log(a, b)
```

优点：一眼能看懂就是最好的优点
缺点：硬说缺点就是引入了一个多余的变量

#### 5.2 进阶版

在不引入中间变量的情况下也能交互两个变量

```
let a = 3,b = 4
a += b
b = a - b
a -= b
console.log(a, b)
```

优点：比起楼上那种没有引入多余的变量，比上面那一种稍微巧妙一点
缺点：当然缺点也很明显，整型数据溢出，比如说对于32位字符最大表示有符号数字是2147483647，也就是Math.pow(2,31)-1，如果是2147483645和2147483646交换就失败了。

#### 5.3 终极版

利用一个数异或本身等于０和异或运算符合交换率。

```
let a = 3,b = 4
  a ^= b
  b ^= a
  a ^= b
console.log(a, b)
```

下面用竖式进行简单说明：(10进制化为二进制)

```
    a = 011
(^) b = 100
则  a = 111(a ^ b的结果赋值给a，a已变成了7)
(^) b = 100
则  b = 011(b^a的结果赋给b，b已经变成了3)
(^) a = 111
则  a = 100(a^b的结果赋给a，a已经变成了4)

```

从上面的竖式可以清楚的看到利用异或运算实现两个值交换的基本过程。

下面从深层次剖析一下：

> 1.对于开始的两个赋值语句，a = a ^ b，b = b ^ a，相当于b = b ^ (a ^ b) = a ^ b ^ b，而b ^ b 显然等于0。因此b = a ^ 0，显然结果为a。
> \2. 同理可以分析第三个赋值语句，a = a ^ b = (a ^ b) ^ a = b

注：

1. ^ 即”异或“运算符。

> 它的意思是判断两个相应的位值是否为”异“，为”异"(值不同)就取真（1）;否则为假（0）。

1. ^ 运算符的特点是与0异或，保持原值；与本身异或，结果为0。

优点：不存在引入中间变量，不存在整数溢出
缺点：前端对位操作这一块可能了解不深，不容易理解

#### 5.4 究极版

熟悉 `ES6` 语法的人当然不会对解构陌生

```
var a = 3,b = 4;
[b, a] = [a, b]
```

其中的解构的原理，我暂时还没读过 ES6的规范，不知道具体的细则，不过我们可以看看 `babel` 是自己编译的，我们可以看出点门路。

哈哈，简单粗暴，不知道有没有按照 ES 的规范，其实可以扒一扒 v8的源码，chrome 已经实现这种解构用法。

[![img](https://camo.githubusercontent.com/509c92224ea81a8a326a7a2e77189c63644a1c35/68747470733a2f2f7773322e73696e61696d672e636e2f6c617267652f303036744e633739677931666d6c3838376b7171676a3331686f306f7774646f2e6a7067)](https://camo.githubusercontent.com/509c92224ea81a8a326a7a2e77189c63644a1c35/68747470733a2f2f7773322e73696e61696d672e636e2f6c617267652f303036744e633739677931666d6c3838376b7171676a3331686f306f7774646f2e6a7067)

**这个例子和前面的例子编写风格有何不同**，你如果细心的话就会发现这两行代码多了一个**分号**，对于我这种编码不写分号的洁癖者，为什么加一个分号在这里，其实是有原因的，这里就简单普及一下，**建议大家还是写代码写上分号**。

#### 5.4 ECMAScript 自动分号;插入(作为补充，防止大家以后踩坑)

尽管 JavaScript 有 C 的代码风格，但是它不强制要求在代码中使用分号，实际上可以省略它们。

JavaScript 不是一个没有分号的语言，恰恰相反上它需要分号来就解析源代码。 因此 JavaScript 解析器在遇到由于缺少分号导致的解析错误时，会自动在源代码中插入分号。

##### 5.4.1例子

```
var foo = function() {
} // 解析错误，分号丢失
test()
```

自动插入分号，解析器重新解析。

```
var foo = function() {
}; // 没有错误，解析继续
test()
```

##### 5.4.2工作原理

下面的代码没有分号，因此解析器需要自己判断需要在哪些地方插入分号。

```
(function(window, undefined) {
    function test(options) {
        log('testing!')

        (options.list || []).forEach(function(i) {

        })

        options.value.test(
            'long string to pass here',
            'and another long string to pass'
        )

        return
        {
            foo: function() {}
        }
    }
    window.test = test

})(window)

(function(window) {
    window.someLibrary = {}
})(window)

```

下面是解析器"猜测"的结果。

```
(function(window, undefined) {
    function test(options) {

        // 没有插入分号，两行被合并为一行
        log('testing!')(options.list || []).forEach(function(i) {

        }); // <- 插入分号

        options.value.test(
            'long string to pass here',
            'and another long string to pass'
        ); // <- 插入分号

        return; // <- 插入分号, 改变了 return 表达式的行为
        { // 作为一个代码段处理
            foo: function() {}
        }; // <- 插入分号
    }
    window.test = test; // <- 插入分号

// 两行又被合并了
})(window)(function(window) {
    window.someLibrary = {}; // <- 插入分号
})(window); //<- 插入分号

```

解析器显著改变了上面代码的行为，在另外一些情况下也会做出错误的处理。

##### 5.4.3 ECMAScript对自动分号插入的规则

我们翻到7.9章节，看看其中插入分号的机制和原理，清楚只写以后就可以尽量以后少踩坑

**必须用分号终止某些 ECMAScript 语句 ( 空语句 , 变量声明语句 , 表达式语句 , do-while 语句 , continue 语句 , break 语句 , return 语句 ,throw 语句 )。这些分号总是明确的显示在源文本里。然而，为了方便起见，某些情况下这些分号可以在源文本里省略。描述这种情况会说：这种情况下给源代码的 token 流自动插入分号。**  [![img](https://camo.githubusercontent.com/d0dd45ca6894c705cb2e2444ac1107c6203be0fe/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f6136363063616232677931666461336367376472686a3231696f306c61746862)](https://camo.githubusercontent.com/d0dd45ca6894c705cb2e2444ac1107c6203be0fe/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f6136363063616232677931666461336367376472686a3231696f306c61746862)
[![img](https://camo.githubusercontent.com/de4c5635fd11dc26956ad2e0b5c5971ff178edae/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f613636306361623267793166646133636f646974616a32316477306969616731)](https://camo.githubusercontent.com/de4c5635fd11dc26956ad2e0b5c5971ff178edae/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f613636306361623267793166646133636f646974616a32316477306969616731)

还是比较抽象，看不太懂是不是，不要紧，我们看看实际例子，总结出几个规律就行，我们先不看抽象的，看着头晕，看看具体的总结说明， **化抽象为具体** 。

首先这些规则是基于两点：

1. 以换行为基础；
2. 解析器会尽量将新行并入当前行，当且仅当符合ASI规则时才会将新行视为独立的语句。

###### 5.4.3.1 ASI的规则

**1. 新行并入当前行将构成非法语句，自动插入分号。**

```
if(1 < 10) a = 1
console.log(a)
// 等价于
if(1 < 10) a = 1;
console.log(a);
```

**2. 在continue,return,break,throw后自动插入分号**

```
return
{a: 1}
// 等价于
return;
{a: 1};
```

**3. ++、--后缀表达式作为新行的开始，在行首自动插入分号**

```
a
++
c
// 等价于
a;
++c;
```

**4. 代码块的最后一个语句会自动插入分号**

```
function(){ a = 1 }
// 等价于
function(){ a = 1; }
```

###### 5.4.3.2 No ASI的规则

**1. 新行以 ( 开始**

```
var a = 1
var b = a
(a+b).toString()
// 会被解析为以a+b为入参调用函数a，然后调用函数返回值的toString函数
var a = 1
var b =a(a+b).toString()
```

**2. 新行以 [ 开始**

```
var a = ['a1', 'a2']
var b = a
[0,1].slice(1)
// 会被解析先获取a[1]，然后调用a[1].slice(1)。
// 由于逗号位于[]内，且不被解析为数组字面量，而被解析为运算符，而逗号运算符会先执
行左侧表达式，然后执行右侧表达式并且以右侧表达式的计算结果作为返回值
var a = ['a1', 'a2']
var b = a[0,1].slice(1)
```

**3. 新行以 / 开始**

```
var a = 1
var b = a
/test/.test(b)
// /会被解析为整除运算符，而不是正则表达式字面量的起始符号。浏览器中会报test前多了个.号
var a = 1
var b = a / test / .test(b)

```

**4. 新行以 + 、 - 、 % 和 \* 开始**

```
var a = 2
var b = a
+a
// 会解析如下格式
var a = 2
var b = a + a
```

**5. 新行以 , 或 . 开始**

```
var a = 2
var b = a
.toString()
console.log(typeof b)
// 会解析为
var a = 2
var b = a.toString()
console.log(typeof b)
```

到这里我们已经对ASI的规则有一定的了解了，另外还有一样有趣的事情，就是“空语句”。

```
// 三个空语句
;;;

// 只有if条件语句，语句块为空语句。
// 可实现unless条件语句的效果
if(1>2);else
  console.log('2 is greater than 1 always!');

// 只有while条件语句，循环体为空语句。
var a = 1
while(++a < 100);
```

##### 5.4.4 结论

建议绝对不要省略分号，同时也提倡将花括号和相应的表达式放在一行， 对于只有一行代码的 if 或者 else 表达式，也不应该省略花括号。 这些良好的编程习惯不仅可以提到代码的一致性，而且可以防止解析器改变代码行为的错误处理。
 [关于JavaScript 语句后应该加分号么？(点我查看)](https://www.zhihu.com/question/20298345)我们可以看看知乎上大牛们对着个问题的看法。

**你可能不知道的前端知识点：原来 JavaScript 还有位操作以及分号的使用细则**

### 6、将 argruments 对象(类数组)转换成数组

`{0:1,1:2,2:3,length:3}`这种形式的就属于类数组，就是按照数组下标排序的对象，还有一个 `length`属性，有时候我们需要这种对象能调用数组下的一个方法，这时候就需要把把类数组转化成真正的数组。

#### 6.1 普通版

```
var makeArray = function(array) {
  var ret = []
  if (array != null) {
    var i = array.length
    if (i == null || typeof array === "string") ret[0] = array
    else while (i) ret[--i] = array[i];
  }
  return ret
}
makeArray({0:1,1:2,2:3,length:3}) //[1,2,3]
```

优点：通用版本，没有任何兼容性问题
缺点：太普通

#### 6.2 进阶版

```
var arr = Array.prototype.slice.call(arguments);
```

这种应该是大家用过最常用的方法，至于为什么可以这么用，很多人估计也是一知半解，反正我看见大家这么用我也这么用，要搞清为什么里面的原因，我们还是从规范和源码说起。

照着规范的流程，自己看看推演一下就明白了：
[英文版15.4.4.10 Array.prototype.slice (start, end) ](http://es5.github.io/#x15.4.4.10)
[中文版15.4.4.10 Array.prototype.slice (start, end) ](http://yanhaijing.com/es5/#352)
如果你想知道 `JavaScript` 的 `sort` 排序的机制，到底是哪种排序好，用的哪种，也可以从规范看出端倪。

在官方的解释中，如[[mdn](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)]

> The slice() method returns a shallow copy of a portion of an array into a new array object.

**简单的说就是根据参数，返回数组的一部分的 copy。所以了解其内部实现才能确定它是如何工作的。所以查看 V8 源码中的 Array.js 可以看到如下的代码：**

方法 `ArraySlice`，[源码地址](https://github.com/v8/v8/blob/ad82a40509c5b5b4680d4299c8f08d6c6d31af3c/src/js/array.js)，第 `660` 行,直接添加到 `Array.prototype` 上的“入口”，内部经过参数、类型等等的判断处理，分支为 `SparseSlice` 和 `SimpleSlice` 处理。

`slice.call` 的作用原理就是，利用 `call`，将 `slice` 的方法作用于 `arrayLike`，`slice` 的两个参数为空，`slice` 内部解析使得 `arguments.lengt` 等于0的时候 相当于处理 `slice(0)` ： 即选择整个数组，`slice` 方法内部没有强制判断必须是 `Array` 类型，`slice` 返回的是新建的数组（使用循环取值）”，所以这样就实现了类数组到数组的转化，`call` 这个神奇的方法、`slice` 的处理缺一不可。

直接看 `slice` 怎么实现的吧。其实就是将 `array-like` 对象通过下标操作放进了新的 `Array` 里面：

```
// This will work for genuine arrays, array-like objects, 
    // NamedNodeMap (attributes, entities, notations),
    // NodeList (e.g., getElementsByTagName), HTMLCollection (e.g., childNodes),
    // and will not fail on other DOM objects (as do DOM elements in IE < 9)
    Array.prototype.slice = function(begin, end) {
      // IE < 9 gets unhappy with an undefined end argument
      end = (typeof end !== 'undefined') ? end : this.length;

      // For native Array objects, we use the native slice function
      if (Object.prototype.toString.call(this) === '[object Array]'){
        return _slice.call(this, begin, end); 
      }

      // For array like object we handle it ourselves.
      var i, cloned = [],
        size, len = this.length;

      // Handle negative value for "begin"
      var start = begin || 0;
      start = (start >= 0) ? start : Math.max(0, len + start);

      // Handle negative value for "end"
      var upTo = (typeof end == 'number') ? Math.min(end, len) : len;
      if (end < 0) {
        upTo = len + end;
      }

      // Actual expected size of the slice
      size = upTo - start;

      if (size > 0) {
        cloned = new Array(size);
        if (this.charAt) {
          for (i = 0; i < size; i++) {
            cloned[i] = this.charAt(start + i);
          }
        } else {
          for (i = 0; i < size; i++) {
            cloned[i] = this[start + i];
          }
        }
      }

      return cloned;
    };
```

优点：最常用的版本，兼容性较强
缺点：ie 低版本，无法处理 dom 集合的 slice call 转数组。（虽然具有数值键值、length 符合ArrayLike 的定义，却报错）搜索资料得到 ：因为 ie 下的 dom 对象是以 com 对象的形式实现的，js 对象与com对象不能进行转换 。

#### 6.3 ES6 版本

使用 `Array.from`, 值需要对象有 `length` 属性, 就可以转换成数组

```
var arr = Array.from(arguments);
```

扩展运算符

```
var args = [...arguments];
```

`ES6` 中的扩展运算符...也能将某些数据结构转换成数组，这种数据结构必须有便利器接口。
优点：直接使用内置 API，简单易维护
缺点：兼容性，使用 babel 的 profill 转化可能使代码变多，文件包变大

**你可能不知道的前端知识点：slice 方法的具体原理**

### 7、数字取整 2.33333 => 2

#### 7.1 普通版

```
const a = parseInt(2.33333)
```

`parseInt()` 函数解析一个字符串参数，并返回一个指定基数的整数 (数学系统的基础)。这个估计是直接取整最常用的方法了。
更多关于 `parseInt()` 函数可以查看 [MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt)

#### 7.2 进阶版

```
const a = Math.trunc(2.33333)
```

`Math.trunc()` 方法会将数字的小数部分去掉，只保留整数部分。
特别要注意的是：`Internet Explorer` 不支持这个方法，不过写个 `Polyfill` 也很简单：

```
Math.trunc = Math.trunc || function(x) {
  if (isNaN(x)) {
    return NaN;
  }
  if (x > 0) {
    return Math.floor(x);
  }
  return Math.ceil(x);
};
```

数学的事情还是用数学方法来处理比较好。

#### 7.3 黑科技版

##### 7.3.1 ~~number

双波浪线 ~~ 操作符也被称为“双按位非”操作符。你通常可以使用它作为代替 Math.trunc() 的更快的方法。

```
console.log(~~47.11)  // -> 47
console.log(~~1.9999) // -> 1
console.log(~~3)      // -> 3
console.log(~~[])     // -> 0
console.log(~~NaN)    // -> 0
console.log(~~null)   // -> 0
```

失败时返回0,这可能在解决 Math.trunc() 转换错误返回 NaN 时是一个很好的替代。
但是当数字范围超出 ±2^31−1 即：2147483647 时，异常就出现了：

```
// 异常情况
console.log(~~2147493647.123) // -> -2147473649 🙁
```

##### 7.3.2 number | 0

| (按位或) 对每一对比特位执行或（OR）操作。

```
console.log(20.15|0);          // -> 20
console.log((-20.15)|0);       // -> -20
console.log(3000000000.15|0);  // -> -1294967296 🙁
```

##### 7.3.3 number ^ 0

^ (按位异或)，对每一对比特位执行异或（XOR）操作。

```
console.log(20.15^0);          // -> 20
console.log((-20.15)^0);       // -> -20
console.log(3000000000.15^0);  // -> -1294967296 🙁
```

##### 7.3.4 number << 0

<< (左移) 操作符会将第一个操作数向左移动指定的位数。向左被移出的位被丢弃，右侧用 0 补充。

```
console.log(20.15 < < 0);     // -> 20
console.log((-20.15) < < 0);  //-20
console.log(3000000000.15 << 0);  // -> -1294967296 🙁
```

上面这些按位运算符方法执行很快，当你执行数百万这样的操作非常适用，速度明显优于其他方法。但是代码的可读性比较差。还有一个特别要注意的地方，处理比较大的数字时（当数字范围超出 ±2^31−1 即：2147483647），会有一些异常情况。使用的时候明确的检查输入数值的范围。

### 8、数组求和

#### 8.1 普通版

```
let arr = [1, 2, 3, 4, 5]
function sum(arr){
    let x = 0
    for(let i = 0; i < arr.length; i++){
        x += arr[i]
    }
    return x
}
sum(arr) // 15
```

优点：通俗易懂，简单粗暴
缺点：没有亮点，太通俗

#### 8.2 优雅版

```
let arr = [1, 2, 3, 4, 5]
function sum(arr) {
return arr.reduce((a, b) => a + b)
}
sum(arr) //15
```

优点：简单明了，数组迭代器方式清晰直观
缺点：不兼容 IE 9以下浏览器

#### 8.3 终极版

```
let arr = [1, 2, 3, 4, 5]
function sum(arr) {
return eval(arr.join("+"))
}
sum(arr) //15
```

优点：让人一时看不懂的就是"好方法"。
缺点：

> eval 不容易调试。用 chromeDev 等调试工具无法打断点调试，所以麻烦的东西也是不推荐使用的…

> 性能问题，在旧的浏览器中如果你使用了eval，性能会下降10倍。在现代浏览器中有两种编译模式：fast path和slow path。fast path是编译那些稳定和可预测（stable and predictable）的代码。而明显的，eval 不可预测，所以将会使用 slow path ，所以会慢。

更多关于 `eval` 的探讨可以关注这篇文章: [JavaScript 为什么不推荐使用 eval？](https://www.zhihu.com/question/20591877)

**你可能不知道的前端知识点：eval的使用细则**

### 最后

**祝大家圣诞快乐🎄，欢迎补充和交流。**
[![img](https://camo.githubusercontent.com/04fbd9fcac2bc85994dfebb6d74f15d8206c6085/68747470733a2f2f7773332e73696e61696d672e636e2f6c617267652f303036744e625277677931666d716f6e72756c74736a33306e6930736d6469682e6a7067)](https://camo.githubusercontent.com/04fbd9fcac2bc85994dfebb6d74f15d8206c6085/68747470733a2f2f7773332e73696e61696d672e636e2f6c617267652f303036744e625277677931666d716f6e72756c74736a33306e6930736d6469682e6a7067)
[![img](https://camo.githubusercontent.com/f61d61eaa203b60ecb1348ca1e1a1c7f7c17b2d6/68747470733a2f2f7773322e73696e61696d672e636e2f6c617267652f303036744e625277677931666d716f6e79707267396a3330763030747971676b2e6a7067)](https://camo.githubusercontent.com/f61d61eaa203b60ecb1348ca1e1a1c7f7c17b2d6/68747470733a2f2f7773322e73696e61696d672e636e2f6c617267652f303036744e625277677931666d716f6e79707267396a3330763030747971676b2e6a7067)
[![img](https://camo.githubusercontent.com/fd730490d0130d97a98b9695d73fde1c34825f59/68747470733a2f2f7773342e73696e61696d672e636e2f6c617267652f303036744e625277677931666d716f6f706b74646a67333064773037713777682e676966)](https://camo.githubusercontent.com/fd730490d0130d97a98b9695d73fde1c34825f59/68747470733a2f2f7773342e73696e61696d672e636e2f6c617267652f303036744e625277677931666d716f6f706b74646a67333064773037713777682e676966)