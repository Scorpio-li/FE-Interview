<!--
 * @Author: Li Zhiliang
 * @Date: 2020-11-27 10:57:30
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-11-27 14:51:13
 * @FilePath: /FE-Interview.git/javascript/handleWrittenCode.md
-->
# 手写逻辑代码搜集

## 1. 实现继承

```js
// ES5
//实现一下继承
function Parent() {
    this.name = "大人"
    this.hairColor = "黑色"
}

function Child() {
    Parent.call(this)
    this.name = "小孩"
}

Child.prototype = Object.create(Parent.prototype)
//将丢失的构造函数给添加回来
Child.prototype.constructor = Child

let c1 = new Child()
console.log(c1.name, c1.hairColor) //小孩，黑色
console.log(Object.getPrototypeOf(c1))
console.log(c1.constructor) //Child构造函数

let p1 = new Parent()
console.log(p1.name, p1.hairColor) //大人，黑色
console.log(Object.getPrototypeOf(p1))
console.log(p1.constructor) //Parent构造函数

// ES6的继承
class Parent {
  constructor() {
    this.name = "大人"
    this.hairColor = "黑色"
  }
}

class Child extends Parent {
  constructor() {
    super()  //调用父级的方法和属性
    this.name = "小孩"
  }
}

let c = new Child()
console.log(c.name, c.hairColor) //小孩 黑色

let p = new Parent()
console.log(p.name, p.hairColor) //大人 黑色
```

## 2. 实现const

- 此处需要用到Object.defineProperty(Obj,prop,desc)这个API

```js
function _const (key, value) {
	const desc = {
        value,
        writable:false
    }
    Object.defineProperty(window,key,desc)
}

_const('obj',{a:1}) //定义obj
obj = {} //重新赋值不生效
```

## 3. 手写Call

原理：

- 给CONTEXT设置一个属性(属性名尽可能保持唯一,避免我们自己设置的属性修改默认对象中的结构,例如可以基于Symbol实现,也可以创建一个时间戳名字),属性值一定是我们要执行的函数(也就是THIS,CALL中的THIS就是我们要操作的这个函数)

- 接下来基于CONTEXT.XXX()成员访问执行方法，就可以把函数执行，并且改变里面的THIS(还可以把PARAMS中的信息传递给这个函数即可);都处理完了，别忘记把给CONTEXT设置的这个属性删除掉(人家之前没有你自己加，加完了我们需要把它删了)

```js
Function.prototype.call = function(context,...params){
  let key = Symbol('key'),//设置唯一值
      result;
  !/^(object|function)$/.test(typeof context) ? context = Object(context) :null;
  context !=null ? null : context = window;//如果context为null或者undefined，直接赋值为window

  context[key] = this;
  result = context[key](...params);//返回值
  delete context[key];
  return result;
}
```


```js
//手写call
let obj = {
  msg: "我叫王大锤",
}

function foo() {
  console.log(this.msg)
}

// foo.call(obj)
//调用call的原理就跟这里一样，将函数挂载到对象上，然后在对象中执行这个函数
// obj.foo = foo
// obj.foo()

Function.prototype.myCall = function (thisArg, ...args) {
  const fn = Symbol("fn") // 声明一个独有的Symbol属性, 防止fn覆盖已有属性
  thisArg = thisArg || window // 若没有传入this, 默认绑定window对象
  thisArg[fn] = this //this指向调用者
  const result = thisArg[fn](...args) //执行当前函数
  delete thisArg[fn]
  return result
}

foo.myCall(obj)
```


## 4. 手写apply

原理基本与call一样，但后面参数应该为数组

```js
Function.prototype.apply = function(context,params = []){
  let key = Symbol('key'),
      result;
  !/^(object|function)$/.test(typeof context) ? context = Object(context) :null;
  context !=null ? null : context = window;

  context[key] = this;
  result = context[key](...params);
  delete context[key];
  return result;
}
```

```js
// 手写apply (args传入一个数组的形式)，原理其实和call差不多，只是入参不一样
Function.prototype.myApply = function (thisArg, args = []) {
  const fn = Symbol("fn")
  thisArg = thisArg || window
  thisArg[fn] = this
  //虽然apply()接收的是一个数组，但在调用原函数时，依然要展开参数数组
  //可以对照原生apply()，原函数接收到展开的参数数组
  const result = thisArg[fn](...args)
  delete thisArg[fn]
  return result
}

foo.myApply(obj)
```

## 5. 手写bind

原理：

- bind() 函数会创建一个新函数（称为绑定函数），新函数与被调函数（绑定函数的目标函数）具有相同的函数体（在 ECMAScript 5 规范中内置的call属性）。

- 当目标函数被调用时 this 值绑定到 bind() 的第一个参数，该参数不能被重写。绑定函数被调用时，bind() 也接受预设的参数提供给原函数。

- 一个绑定函数也能使用new操作符创建对象：这种行为就像把原函数当成构造器。提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。

```js
Function.prototype.bind = function(context,...params){
	let self = this;
    return funtion(...innerArgs){
    	params = params.concat(...innerArgs);
        return self.call(context,...params);
    }
}
```

```js
Function.prototype.myBind = function (thisArg, ...args) {
  let self = this //这里的this是指向thisArg(调用者)
  let fnBound = function () {
    //this instanceof self ? this : thisArg 判断是构造函数还是普通函数
    //后面的args.concat(Array.prototype.slice.call(arguments))是利用函数柯里化来获取调用时传入的参数
    self.apply(this instanceof self ? this : thisArg, args.concat(Array.prototype.slice.call(arguments)))
  }
  // 继承原型上的属性和方法
  fnBound.prototype = Object.create(self.prototype)
  //返回已经绑定的函数
  return fnBound
}

//通过普通函数调用
// foo.myBind(obj, 1, 2, 3)()

//通过构造函数调用
function fn(name, age) {
  this.test = "测试数据"
}
fn.prototype.protoData = "原型数据"
let fnBound = fn.myBind(obj, "王大锤", 18)
let newBind = new fnBound()
console.log(newBind.protoData) // "原型数据"
```

## 6. 模拟实现一个new操作符

- 原理：
    
    1. new关键字会首先创建一个空对象
    
    2. 将这个空对象的原型对象指向构造函数的原型属性，从而继承原型上的方法
    
    3. 将this指向这个空对象，执行构造函数中的代码，以获取私有属性
    
    4. 如果构造函数返回了一个对象res，就将该返回值res返回，如果返回值不是对象，就将创建的对象返回

- 模拟实现一个new操作符,传入一个构造函数和参数

```js
function myNew(constructFn, ...args) {
  // 创建新对象,并继承构造方法的prototype属性,
  //把obj挂原型链上, 相当于obj.__proto__ = constructFn.prototype
  let obj = Object.create(constructFn.prototype)

  //执行构造函数，将args参数传入，主要是为了进行赋值this.name = name等操作
  let res = constructFn.apply(obj, args)

  //确保返回值是一个对象
  return res instanceof Object ? res : obj
}

function Dog(name) {
  this.name = name

  this.woof = function () {
    console.log("汪汪汪")
  }
  //构造函数可以返回一个对象
  //return { a: 1 }
}

let dog = new Dog("阿狸")
console.log(dog.name) //阿狸
dog.woof() //汪汪汪

let dog2 = myNew(Dog, "大狗")
console.log(dog2.name) //大狗
dog2.woof() //汪汪汪
```

```js
// ES5
function _new(target){
  var obj = {},
      params = [].splice.call(arguments,1),
      result;

  obj.__proto__ = target.prototype;
  result = target.apply(obj,params);

  if(result!=null && /(function|object)/.test(typeof result)){
    return result;
  }
  return obj;
}
```

```js
// ES6
function _new(target，...params){
  let obj = Object.create(target.prototype),
  	  result = target.call(obj,...params);
  if(result!=null && /^(function|object)$/.test(typeof result)){
    return result;
  }
  return obj;
}
```

## 7. 节流

节流可以控制事件触发的频率，节流就跟小水管一样，如果不加节流的话，水就会哗啦啦啦啦啦啦的流出来，但是一旦加了节流阀，你就可以自己控制水的流速了，加了节流后水可以从哗啦啦啦变成滴答滴答滴答，放到我们的函数事件里面说就是可以让事件触发变慢，比如说事件触发可以让它在每一秒内只触发一次，可以提高性能。

```js
function throttle(fn, wait) {
    let prev = new Date()
    return function() {
        let now = new Date()
        /*当下一次事件触发的时间和初始事件触发的时间的差值大于
			等待时间时才触发新事件 */
        if(now - prev > wait) {
            fn.apply(this, arguments)
            //重置初始触发时间
        	prev = +new Date()
        }
    }
}
```

```js
/*
      @params:
          func[function]:最后要触发执行的函数
          wait[number]:触发的频率
        @return
          可以被调用执行的函数
*/
function throttle(func,wait = 300){
      let timer = null,
          previous = 0;//记录上一次操作时间
      return function anonymouse(...params){
        let now = new Date(),//记录当前时间
            remaining = wait - (now - previous);//记录还差多久达到我们一次触发的频率
        if(remaining <= 0){
          //两次操作的间隔时间已经超过wait了
          window.clearInterval(timer);
          timer = null;
          previous = now;
          func.call(this,...params);
        }else if(!timer){
          //两次操作的间隔时间还不符合触发的频率
          timer = setTimeout(() => {
            timer = null;
            previous = new Date();
            func.call(this,...params);
          }, remaining);
        }
      }
}
```

## 8. 防抖

防抖就是可以限制事件在一定时间内不能多次触发，比如说你疯狂按点击按钮，一顿操作猛如虎，不加防抖的话它也会跟着你疯起来，疯狂执行触发的方法。但是一旦加了防抖，无论你点击多少次，他都只会在你最后一次点击的时候才执行。 防抖常用于搜索框或滚动条等的监听事件处理，可以提高性能。

```js
function debounce(fn, wait = 50) {
    //初始化一个定时器
    let timer
    return function() {
        //如果timer存在就将其清除
        if(timer) {
            clearTimeout(timer)
        }
        //重置timer
        timer = setTimeout(() => {
            //将入参绑定给调用对象
            fn.apply(this, arguments)
        }, wait)
    }
}
```

```js
/*
      防抖:
        @params:
          func[function]:最后要触发执行的函数
          wait[number]:频繁设定的界限
          immediate[boolean]:默认多次操作，我们识别的是最后一次，但是immediate=true，让其识别第一次
        @return
          可以被调用执行的函数
 */
function debounce(func,wait = 300,immediate = false){
      let timer = null;
      return function anonymous(...params){
        let now = immediate && !timer;

        //每次点击都把之前设置的定时器清除掉
        clearInterval(timer)
        //重新设置一个新的定时器监听wait事件内是否触发第二次
        timer = setTimeout(() => {
          timer = null;//垃圾回收机制
          //wait这么久的等待中，没有触发第二次
          !immediate ? func.call(this,...params) : null;
        }, wait);

        //如果是立即执行
        now ? func.call(this,...params) : null;
      }
}
```

## 9. 深拷贝和浅拷贝

**浅拷贝**： 顾名思义，所谓浅拷贝就是对对象进行浅层次的复制，只复制一层对象的属性，并不包括对象里面的引用类型数据 ， 当遇到有子对象的情况时，子对象就会互相影响 ，修改拷贝出来的子对象也会影响原有的子对象

**深拷贝**： 深拷贝是对对象以及对象的所有子对象进行拷贝，也就是说新拷贝对象的子对象里的属性也不会影响到原来的对象

```js
let obj = {
  a: 1,
  b: 2,
  c: {
    d: 3,
    e: 4
  }
}
```

### 浅拷贝

#### Object实现浅拷贝: 

- 使用Object.assign()

```js
let obj2 = Object.assign({}, obj)
obj2.a = 111
obj2.c.e = 555
console.log(obj)
console.log(obj2)
```

- 使用展开运算符

```js
let obj2 = {...obj}
obj2.a = 111
obj2.c.e = 555
console.log(obj)
console.log(obj2)
```

```js
function clone(obj){
      var type = toType(obj);
      if(/^(boolean|number|string|null|undefined|bigInt|symbol)$/.test(type)) return obj;
      if(/^(regExp|date)$/.test(type)) return new obj.constructor(obj);
      if(/^(function)$/.test(obj)) return function(){
        obj()
      };
      if(/^error$/.test(type))return new obj.constructor(obj.message);
      var target = {},
          keys = this.getOwnProperty(obj); 
      Array.isArray(target) ? target = [] : null;
      
     keys.forEach(item=>{
       target[item] = obj[item]
     })
      return target
}
```

#### Array的浅拷贝

```js
let arr = [1, 2, { a: 3 }]

// 使用Array.prototype.slice()

let arr2 = arr.slice()
arr2[0] = 222
arr[2].a = 333
console.log(arr)
console.log(arr2)

// 使用Array.prototype.concat()

let arr2 = arr.concat()
arr2[0] = 222
arr[2].a = 333
console.log(arr)
console.log(arr2)

// 使用展开运算符...

let arr2 = [...arr]
arr2[0] = 222
arr[2].a = 333
console.log(arr)
console.log(arr2)
```

#### 结合

```js
function shallowCopy(source) {
    //开头可以判断一下入参是不是一个对象
    let target = Array.isArray(source) ? [] : {}
    for(let key in source) {
        //使用 hasOwnProperty 限制循环只在对象自身，不去遍历原型上的属性
        if(source.hasOwnProperty(key)) {
            target[key] = source[key]
        }
    }
    return target
}
```

### 深拷贝

#### 1. 最简单的深拷贝方式

```js
let target = JSON.parse(JSON.stringify(source))
```

但是这种方法的话只支持**object、array、string、number、true、false、null**这几种数据或者值，其他的比如**函数、undefined、Date、RegExp**等数据类型都不支持。对于它不支持的数据都会直接忽略该属性。

#### 2. 递归的方式

```js
function deepCopy(source) {
  //开头这里可以判断入参是不是一个对象
  let target = Array.isArray(source) ? [] : {}
  for (let key in source) {
    if (source.hasOwnProperty(key)) {
      //这里我们再做一层判断看看是否有子属性
      //这里也可以直接调用上面写过的那个checkType函数进行判断，就不用写两个typeof了
      if (source[key] && typeof source[key] !== null && typeof source[key] === "object")
      {
        target[key] = Array.isArray(source[key]) ? [] : {}
        //递归调用
        target[key] = deepCopy(source[key])
      } else {
        target[key] = source[key]
      }
    }
  }
  return target
}
```

这里的深拷贝只能进行简单数据类型的拷贝，如果是复杂数据类型则拷贝的过程种会丢失掉数据，比如说拷贝的对象里含有正则表达式或者函数，日期，这些是无法拷贝的，如果需要拷贝这些的话就需要我们单独去对每一种类型做出判断，另外就是递归的版本还会存在循环引用的问题 

#### 优化

```js
function deepCopy(source, cache = new Map()) {
  if (cache.has(source)) {
    //如果缓存中已经有值则直接返回，解决循环调用的问题
    return cache.get(source)
  }
  //当入参属于Object复杂数据类型就开始做子类检测-> Function Array RegExp Date 都属于Object类型
  if (source instanceof Object) {
    let target
    if (source instanceof Array) {
      //判断数组的情况
      target = []
    } else if (source instanceof Function) {
      //判断函数的情况
      target = function () {
        return source.apply(this, arguments)
      }
    } else if (source instanceof RegExp) {
      //判断正则表达式的情况
      target = source
    } else if (source instanceof Date) {
      target = new Date(source)
    } else {
      //普通对象
      target = {}
    }
    // 将属性和拷贝后的值进行缓存
    cache.set(source, target)
    //开始做遍历递归调用
    for (let key in source) {
      if (source.hasOwnProperty(key)) {
        target[key] = deepCopy(source[key], cache)
      }
    }
    return target
  } else {
    //如果不是复杂数据类型的话就直接返回
    return source
  }
}
```

## 10. 实现一个sleep休眠函数

从Promise，async/await，generator等角度出发进行考虑

```js
//方法一：
function sleep(time) {
    return new Promise(resolve => {
        setTimeout(resolve, time)
    })
}

sleep(1000).then(() => {
    console.log('1秒过后执行这里')
})

//方法二：
//在函数中用async await调用上面封装好的sleep方法
async foo() {
    await sleep(1000)
    console.log('这里在一秒后打印')
}
foo()

//方法三：
//generator实现
function* generatorSleep(time){
  yield new Promise(reslove => {
      setTimeout(reslove, time)
  })
}

generatorSleep(1000).next().value.then(()=>{
  console.log('generator实现方式')
})

//方法四：
//通过回调函数来调用 cb->callback
function sleepCb(cb, time) {
    if(typeof cb !== 'function') return
    setTimeout(cb, time)
}

function foo() {
    console.log('1秒后打印这个回调函数')
}
sleepCb(foo, 1000)
```

## 11. 实现一个repeat重复执行函数

传入一个方法，然后每隔一段时间执行一次，执行n次

```js
//每隔2s输出一次helloworld，共输出4次 const repeatFunc = repeat(console.log, 4, 2000); //repeatFunc("helloworld")

function repeat(fn, n, interval) {
  return (...args) => {
    let timer
    let counter = 0
    timer = setInterval(() => {
      counter++
      fn.apply(this, args)
      if (counter === n) {
        clearInterval(timer)
      }
    }, interval);
  }
}

const repeatFn = repeat(console.log, 4, 2000)
repeatFn('helloworld')
```

## 12. 实现一个函数将下划线命名转化成驼峰命名法

```js
function formatHump(str) {
  if (typeof str !== "string") return false
  //将str分割成数组
  str = str.split("_") // ["get", "element", "by", "id"]
  if (str.length > 1) {
    // 从1开始for循环遍历,因为数组第一个字符串的首字母不需要转大写
    // 将数组里的每一个字符串第一个字母变成大写
    for (let i = 1; i < str.length; i++) {
      str[i] = str[i][0].toUpperCase() + str[i].substr(1)
    }
    //将数组拼接回字符串
    return str.join("")
  }
}

console.log(formatHump("get_element_by_id"))  //getElementById
```

## 13. 实现一个带并发限制的异步调度器Scheduler，最多同时运行两个任务

```js
//异步调度器
class Scheduler {
  constructor(maxNum) {
    //等待执行的任务队列
    this.taskList = []
    //当前任务数
    this.count = 0
    //最大任务数
    this.maxNum = maxNum
  }
    
  async add(promiseCreator) {
     //当当前任务数超出最大任务数就将其加入等待执行的任务队列
    if (this.count >= this.maxNum) {
      await new Promise(resolve => {
        this.taskList.push(resolve)
      })
    }
    this.count++
    const result = await promiseCreator()
    this.count--
    //当其它任务执行完任务队列中还有任务没执行就将其出队并执行
    if (this.taskList.length > 0) {
      this.taskList.shift()()
    }
    return result
  }
}

const timeout = time => {
  return new Promise(resolve => {
    setTimeout(resolve, time)
  })
}

const scheduler = new Scheduler(2)
const addTask = (time, value) => {
  scheduler.add(() => {
    return timeout(time).then(() => {
      console.log(value)
    })
  })
}

addTask(1000, "1")
addTask(500, "2")
addTask(300, "3")
addTask(400, "4")

//此处输出2 -> 3 ->1 -> 4
//一开始1、2两个任务进入队列
//500ms时，2完成，输出2，任务3进入队列
//800ms时，3完成，输出3，任务4进入队列
//1000ms时，1完成，输出1
//1200ms时，4完成，输出4
```



## 14. instanceof

```js
function _instanceof(L,R){    
    // 验证如果为基本数据类型，就直接返回false
    const baseType = ['string', 'number','boolean','undefined','symbol']
    if(baseType.includes(typeof(L))) { return false }
    
    let RP  = R.prototype;  //取 R 的显示原型
    L = L.__proto__;       //取 L 的隐式原型
    while(true){           // 无线循环的写法（也可以使 for(;;) ）
        if(L === null){    //找到最顶层
            return false;
        }
        if(L === RP){       //严格相等
            return true;
        }
        L = L.__proto__;  //没找到继续向上一层原型链查找
    }
}
```

## 15. 数组扁平化的方法

```js
//使用ES6中的Array.prototype.flat方法
arr.flat(Infinity)

let arr = [1,2,[3,4,[5,[6]]]]
console.log(arr.flat(Infinity))//flat参数为指定要提取嵌套数组的结构深度，默认值为 1
```

```js
//使用reduce的方式
function arrFlat(arr) {
  return arr.reduce((pre, cur) => {
    return pre.concat(Array.isArray(cur) ? arrFlat(cur) : cur)
  }, [])
}
```

```js
//使用递归加循环的方式
function arrFlat(arr) {
  let result = []
  arr.map((item, index) => {
    if (Array.isArray(item)) {
      result = result.concat(arrFlat(item))
    } else {
      result.push(item)
    }
  })
  return result
}
```

```js
//将数组先变成字符串，再复原 toString()
//这种方法存在缺陷，就是数组中元素都是Number或者String类型的才能展开
function arrFlat(arr) {
    return arr.toString().split(',').map(item=> +item)
}
```

## 16. 事件绑定

1. 直接在html中定义事件

```html
<button onclick="console.log(1)">点我</button>
```

> 缺点：违反了内容与行为相分离的原则，在html中写js代码，强耦合，不利于复用代码。不推荐

2. DOM0级事件

```js
let btn = document.querySelector(".btn");
btn.onclick = function(){
	console.log(1);
}
```

> 缺点:解决了强耦合性，但只能给一个事件对象绑定一个事件类型。 同样的事件类型，绑定两次，以 后一次 为准。

3. -DOM2级事件

```js
let btn = document.querySelector(".btn");
btn.addEventListener("click",function(){
	console.log(1);
},false)//默认值为false，可不写，false表示冒泡
```

> 同样的事件类型，绑定多次，每个都会执行，可以设置时捕获还是冒泡。

## 10.
## 10.
## 10.
## 10.
## 10.
## 10.