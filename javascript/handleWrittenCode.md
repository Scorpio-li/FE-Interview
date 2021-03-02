<!--
 * @Author: Li Zhiliang
 * @Date: 2020-11-27 10:57:30
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-12-26 10:42:43
 * @FilePath: /FE-Interview.git/javascript/handleWrittenCode.md
-->
# 手写逻辑代码搜集（待验证）

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

> 寄生组合继承（ES5继承的最佳方式）

所谓寄生组合式继承，即通过借用构造函数来继承属性，通过原型链的形式来继承方法。

只调用了一次父类构造函数，效率更高。避免在子类.prototype上面创建不必要的、多余的属性，与其同时，原型链还能保持不变。

```js
function Parent(name) {
  this.name = name;
  this.colors = ['red', 'blue', 'green'];
}
Parent.prototype.getName = function () {
  return this.name;
}

function Child(name, age) {
  Parent.call(this, name); // 调用父类的构造函数，将父类构造函数内的this指向子类的实例
  this.age = age;
}

//寄生组合式继承
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

Child.prototype.getAge = function () {
    return this.age;
}

let girl = new Child('Lisa', 18);
girl.getName();
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

## 3. 模拟实现一个new操作符

- 原理：
    
    1. new关键字会首先创建一个空对象
    
    2. 将这个空对象的原型对象指向构造函数的原型属性，从而继承原型上的方法.这个对象会被执行 [[Prototype]] 连接，将这个新对象的 [[Prototype]] 链接到这个构造函数.prototype 所指向的对象
    
    3. 将this指向这个空对象，执行构造函数中的代码，以获取私有属性.这个新对象会绑定到函数调用的 this
    
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

- 简化版
  
  - 步骤1：创建一个Func的实例对象（实例.proto = 类.prototype）
  
  - 步骤2：把Func 当做普通函数执行，并改变this指向
  
  - 步骤3：分析函数的返回值

```js
/**
  * Func: 要操作的类（最后要创建这个类的实例）
  * args：存储未来传递给Func类的实参
  */
function _new(Func, ...args) {
  
  // 创建一个Func的实例对象（实例.____proto____ = 类.prototype）
  let obj = {};
  obj.__proto__ = Func.prototype;
  
  // 把Func当做普通函数执行，并改变this指向
  let result = Func.call(obj, ...args);
  
  // 分析函数的返回值
  if (result !== null && /^(object|function)$/.test(typeof result)) {
    return result;
	}
  return obj;
}
```

- 优化版： proto 在IE浏览器中不支持

```js
let x = { name: "lsh" };
Object.create(x);

{}.__proto__ = x;


function _new(Func, ...args) {
  
  // let obj = {};
  // obj.__proto__ = Func.prototype;
  // 创建一个Func的实例对象（实例.____proto____ = 类.prototype）
  let obj = Object.create(Func.prototype);
  
  // 把Func当做普通函数执行，并改变this指向
  let result = Func.call(obj, ...args);
  
  // 分析函数的返回值
  if (result !== null && /^(object|function)$/.test(typeof result)) {
    return result;
  }
  return obj;
}
```

## 4. 节流

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

- 【第一次触发：reamining是负数，previous被赋值为当前时间】

- 【第二次触发：假设时间间隔是500ms，第一次执行完之后，20ms之后，立即触发第二次，则remaining = 500 - ( 新的当前时间 - 上一次触发时间 ) = 500 - 20 = 480 】

```js
/**
 * 实现函数的节流 （目的是频繁触发中缩减频率）
 * @param {*} func 需要执行的函数
 * @param {*} wait 检测节流的间隔频率
 * @param {*} immediate 是否是立即执行 True：第一次，默认False：最后一次
 * @return {可被调用执行的函数}
 */
function throttle(func, wait) {
	let timer = null
  let previous = 0  // 记录上一次操作的时间点

  return function anonymous(... params) {
    let now = new Date()  // 当前操作的时间点
    remaining = wait - (now - previous) // 剩下的时间
    if (remaining <= 0) {
      // 两次间隔时间超过频率，把方法执行
      
      clearTimeout(timer); // clearTimeout是从系统中清除定时器，但timer值不会变为null
      timer = null; // 后续可以通过判断 timer是否为null，而判断是否有 定时器
      
      // 此时已经执行func 函数，应该将上次触发函数的时间点 = 现在触发的时间点 new Date()
      previous = new Date(); // 把上一次操作时间修改为当前时间
      func.call(this, ...params);
    } else if(!timer){ 
      
      // 两次间隔的事件没有超过频率，说明还没有达到触发标准，设置定时器等待即可（还差多久等多久）
      // 假设事件间隔为500ms，第一次执行完之后，20ms后再次点击执行，则剩余 480ms，就能等待480ms
      timer = setTimeout( _ => {
        clearTimeout(timer)
        timer = null // 确保每次执行完的时候，timer 都清 0，回到初始状态
        
        //过了remaining时间后，才去执行func，所以previous不能等于初始时的 now
        previous = new Date(); // 把上一次操作时间修改为当前时间
        func.call(this, ...params)；
      }, remaining)
    }
  }
}

function func() {
  console. log('ok')
}
btn. onclick = throttle(func, 500)
```

## 5. 防抖

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

- 以最后一次触发为标准

```js
/**
 * 实现函数的防抖（目的是频繁触发中只执行一次）
 * @param {*} func 需要执行的函数
 * @param {*} wait 检测防抖的间隔频率
 * @param {*} immediate 是否是立即执行  True：第一次，默认False：最后一次
 * @return {可被调用执行的函数}
 */
function debounce(func, wati = 500, immediate = false) {
  let timer = null
  return function anonymous(... params) {

    clearTimeout(timer)
    timer = setTimeout(_ => {
      // 在下一个500ms 执行func之前，将timer = null
      //（因为clearInterval只能清除定时器，但timer还有值）
      // 为了确保后续每一次执行都和最初结果一样，赋值为null
      // 也可以通过 timer 是否 为 null 是否有定时器
      timer = null
      func.call(this, ...params)
    }, wait)

  }
}
```

- 以第一次触发为标准

```js
/**
 * 实现函数的防抖（目的是频繁触发中只执行一次）
 * @param {*} func 需要执行的函数
 * @param {*} wait 检测防抖的间隔频率
 * @param {*} immediate 是否是立即执行 True：第一次，默认False：最后一次
 * @return {可被调用执行的函数}
 */
function debounce(func, wait = 500, immediate = true) {
  let timer = null
  return function anonymous(... params) {

    // 第一点击 没有设置过任何定时器 timer就要为 null
    let now = immediate && !timer
    clearTimeout(timer)
    timer = setTimeout(_ => {
      // 在下一个500ms 执行func之前，将timer = null
      //（因为clearInterval只能在系统内清除定时器，但timer还有值）
      // 为了确保后续每一次执行都和最初结果一样，赋值为null
      // 也可以通过 timer 是否 为 null 是否有定时器
      timer = null!immediate ? func.call(this, ...params) : null
    }, wait)
    now ? func.call(this, ...params) : null

  }
}

function func() {
  console. log('ok')
}
btn. onclick = debounce(func, 500)
```

## 6. 深拷贝和浅拷贝

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

- 浅克隆：只拷贝对象或数组的第一层内容

```js
const shallClone = (target) => {
  if (typeof target === 'object' && target !== null) {
    const cloneTarget = Array.isArray(target) ? [] : {};
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) { // 遍历对象自身可枚举属性（不考虑继承属性和原型对象）
        cloneTarget[prop] = target[prop];
    }
    return cloneTarget;
  } else {
    return target;
  }
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

```js
function deepClone(target)  {
  if (target === null) return null;
  if (typeof target !== 'object') return target;

  const cloneTarget = Array.isArray(target) ? [] : {};
  for (let prop in target) {
    if (target.hasOwnProperty(prop)) {
      cloneTarget[prop] = deepClone(target[prop]);
    }
  }
  return cloneTarget;
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

```js
const isObject = (target) =>
  (typeof target === "object" || typeof target === "function") &&
  target !== null;

function deepClone(target, map = new Map()) {
  // 先判断该引用类型是否被 拷贝过
  if (map.get(target)) {
    return target;
  }
  // 获取当前值的构造函数：获取它的类型
  let constructor = target.constructor;
  // 检测当前对象target是否与 正则、日期格式对象匹配
  if (/^(RegExp|Date)$/i.test(constructor.name)) {
    return new constructor(target); // 创建一个新的特殊对象(正则类/日期类)的实例
  }
  if (isObject(target)) {
    map.set(target, true); // 为循环引用的对象做标记
    const cloneTarget = Array.isArray(target) ? [] : {};
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
        cloneTarget[prop] = deepClone(target[prop], map);
      }
    }
    return cloneTarget;
  } else {
    return target;
  }
}
```

## 7. 实现一个sleep休眠函数

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

```js
/**
 *
 * @param {*} fn 要执行的函数
 * @param {*} wait 等待的时间
 */
function sleep(wait) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve();
    }, wait);
  });
}

let sayHello = (name) => console.log(`hello ${name}`);

async function autoRun() {
  await sleep(3000);
  let demo1 = sayHello("时光屋小豪");
  let demo2 = sayHello("掘友们");
  let demo3 = sayHello("aaa");
}

autoRun();
```

## 8. 实现一个repeat重复执行函数

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

## 9. 实现一个函数将下划线命名转化成驼峰命名法

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

## 10. 实现一个带并发限制的异步调度器Scheduler，最多同时运行两个任务

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



## 11. instanceof

1. 步骤1：先取得当前类的原型，当前实例对象的原型链

​2. 步骤2：一直循环（执行原型链的查找机制）

- 取得当前实例对象原型链的原型链（proto = proto.____proto____，沿着原型链一直向上查找）

- 如果 当前实例的原型链____proto__上找到了当前类的原型prototype，则返回 true

- 如果 一直找到Object.prototype.____proto____ ==  null，Object的基类(null)上面都没找到，则返回 false

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

```js
function _instanceof (instanceObject, classFunc) {
	let classFunc = classFunc.prototype; // 取得当前类的原型
  let proto = instanceObject.__proto__; // 取得当前实例对象的原型链
  
  while (true) {
    if (proto === null) { // 找到了 Object的基类 Object.prototype.__proto__
      return false;
		};
    if (proto === classFunc) { // 在当前实例对象的原型链上，找到了当前类
      return true;
    }
    proto = proto.__proto__; // 沿着原型链__ptoto__一层一层向上查找
  }
}
```

### 处理兼容问题

> Object.getPrototypeOf ( )：用来获取某个实例对象的原型（内部[[prototype]]属性的值，包含proto属性）

```js
function _instanceof (instanceObject, classFunc) {
	let classFunc = classFunc.prototype; // 取得当前类的原型
  let proto = Object.getPrototypeOf(instanceObject); // 取得当前实例对象的原型链上的属性
  
  while (true) {
    if (proto === null) { // 找到了 Object的基类 Object.prototype.__proto__
      return false;
		};
    if (proto === classFunc) { // 在当前实例对象的原型链上，找到了当前类
      return true;
    }
    proto = Object.getPrototypeOf(proto); // 沿着原型链__ptoto__一层一层向上查找
  }
}
```

## 12. 数组扁平化flat的方法

- 循环数组里的每一个元素

- 判断该元素是否为数组

  - 是数组的话，继续循环遍历这个元素——数组

  - 不是数组的话，把元素添加到新的数组中

```js
//使用ES6中的Array.prototype.flat方法
arr.flat(Infinity)

let arr = [1,2,[3,4,[5,[6]]]]
console.log(arr.flat(Infinity))//flat参数为指定要提取嵌套数组的结构深度，默认值为 1
```

```js
const myFlat = (arr) => {
  let newArr = [];
  let cycleArray = (arr) => {
    for(let i = 0; i < arr.length; i++) {
      let item = arr[i];
      if (Array.isArray(item)) {
        cycleArray(item);
        continue;
      } else {
        newArr.push(item);
      }
    }
  }
  cycleArray(arr);
  return newArr;
}

myFlat(arr); // [1, 2, 2, 3, 4, 5, 5, 6, 7, 8, 9, 11, 12, 12, 13, 14, 10]
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

```js
let arr = [
    [1, 2, 2],
    [3, 4, 5, 5],
    [6, 7, 8, 9, [11, 12, [12, 13, [14]]]], 10
];

function myFlat() {
  _this = this; // 保存 this：arr
  let newArr = [];
  // 循环arr中的每一项，把不是数组的元素存储到 newArr中
  let cycleArray = (arr) => {
    for (let i=0; i< arr.length; i++) {
      let item = arr[i];
      if (Array.isArray(item)) { // 元素是数组的话，继续循环遍历该数组
        cycleArray(item);
        continue;
      } else{
        newArr.push(item); // 不是数组的话，直接添加到新数组中
      }
    }
  }
  cycleArray(_this); // 循环数组里的每个元素
  return newArr; // 返回新的数组对象
}

Array.prototype.myFlat = myFlat;

arr = arr.myFlat(); // [1, 2, 2, 3, 4, 5, 5, 6, 7, 8, 9, 11, 12, 12, 13, 14, 10]
```

## 13. 事件绑定

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

## 14. 手动实现Object.create

```js
Object.create() = function create(prototype) {
  // 排除传入的对象是 null 和 非object的情况
	if (prototype === null || typeof prototype !== 'object') {
    throw new TypeError(`Object prototype may only be an Object: ${prototype}`);
	}
  // 让空对象的 __proto__指向 传进来的 对象(prototype)
  // 目标 {}.__proto__ = prototype
  function Temp() {};
  Temp.prototype = prototype;
  return new Temp;
}
```

## 15. 数组去重

- 思想一：数组最后一项元素替换掉当前项元素，并删除最后一项元素

```js
let arr = [12, 23, 12, 15, 25, 23, 16, 25, 16];

for(let i = 0; i < arr.length - 1; i++) {
  let item = arr[i]; // 取得当前数组中的每一项
  let remainArgs = arr.slice(i+1); // 从 i+1项开始截取数组中剩余元素，包括i+1位置的元素
  if (remainArgs.indexOf(item) > -1) { // 数组的后面元素 包含当前项
    arr[i] = arr[arr.length - 1]; // 用数组最后一项替换当前项
    arr.length--; // 删除数组最后一项
    i--; // 仍从当前项开始比较
  }
}

console.log(arr); // [ 16, 23, 12, 15, 25 ]
```

- 思想二：新容器存储思想——对象键值对

  - 把数组元素作为对象属性

  ​- 通过遍历数组，判断数组元素是否已经是对象的属性，如果对象属性定义过，则证明是重复元素，进而删除重复元素

```js
let obj = {};
for (let i=0; i < arr.length; i++) {
  let item = arr[i]; // 取得当前项
  if (typeof obj[item] !== 'undefined') {
    // obj 中存在当前属性，则证明当前项 之前已经是 obj属性了
    // 删除当前项
    arr[i] = arr[arr.length-1];
    arr.length--;
    i--;
  }
  obj[item] = item; // obj {10: 10, 16: 16, 25: 25 ...}
}
obj = null; // 垃圾回收
console.log(arr); // [ 16, 23, 12, 15, 25 ]
```

- 思想三：相邻项的处理方案思想——基于正则

```js
let arr = [12, 23, 12, 15, 25, 23, 16, 25, 16];

arr.sort((a,b) => a-b);
arrStr = arr.join('@') + '@';
let reg = /(\d+@)\1*/g,
    newArr = [];
arrStr.replace(reg, (val, group1) => {
 // newArr.push(Number(group1.slice(0, group1.length-1)));
 newArr.push(parseFloat(group1));
})
console.log(newArr); // [ 12, 15, 16, 23, 25 ]
```

## 16. 基于Generator函数实现async/await原理

> **核心：**传递给我一个Generator函数，把函数中的内容基于Iterator迭代器的特点一步步的执行

```js
function readFile(file) {
	return new Promise(resolve => {
		setTimeout(() => {
			resolve(file);
    }, 1000);
	})
};

function asyncFunc(generator) {
	const iterator = generator(); // 接下来要执行next
  // data为第一次执行之后的返回结果，用于传给第二次执行
  const next = (data) => {
		let { value, done } = iterator.next(data); // 第二次执行，并接收第一次的请求结果 data
    
    if (done) return; // 执行完毕(到第三次)直接返回
    // 第一次执行next时，yield返回的 promise实例 赋值给了 value
    value.then(data => {
      next(data); // 当第一次value 执行完毕且成功时，执行下一步(并把第一次的结果传递下一步)
    });
  }
  next();
};

asyncFunc(function* () {
	// 生成器函数：控制代码一步步执行 
  let data = yield readFile('a.js'); // 等这一步骤执行执行成功之后，再往下走，没执行完的时候，直接返回
  data = yield readFile(data + 'b.js');
  return data;
})
```

## 17. 基于Promise封装Ajax

- 返回一个新的Promise实例

- 创建HMLHttpRequest异步对象

- 调用open方法，打开url，与服务器建立链接（发送前的一些处理）

- 监听Ajax状态信息

- 如果xhr.readyState == 4（表示服务器响应完成，可以获取使用服务器的响应了）

  - xhr.status == 200，返回resolve状态

  - xhr.status == 404，返回reject状态

- xhr.readyState !== 4，把请求主体的信息基于send发送给服务器

```js
function ajax(url, method) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()
    xhr.open(url, method, true)
    xhr.onreadystatechange = function () {
      if (xhr.readyState === 4) {
        if (xhr.status === 200) {
          resolve(xhr.responseText)
        } else if (xhr.status === 404) {
          reject(new Error('404'))
        }
      } else {
        reject('请求数据失败')
      }
    }
    xhr.send(null)
  })
}
```

## 18. 手动实现JSONP跨域

- 创建script标签

- 设置script标签的src属性，以问号传递参数，设置好回调函数callback名称

- 插入到html文本中

- 调用回调函数，res参数就是获取的数据

```js
let script = document.createElement('script');

script.src = 'http://www.baidu.cn/login?username=JasonShu&callback=callback';

document.body.appendChild(script);

function callback (res) {
	console.log(res);
}
```

## 19. ES5手动实现数组reduce

- 初始值不传时的特殊处理：会默认使用数组中的第一个元素
  
  - 函数的返回结果会作为下一次循环的prev
  
  - 回调函数一共接受四个参数（arr.reduce(prev, next, currentIndex, array))）
  
    - prev：上一次调用回调时返回的值
  
    - 正在处理的元素
  
    - 正在处理的元素的索引
  
    - 正在遍历的集合对象

```js
Array.prototype.myReduce = function(fn, prev) {
  for (let i = 0; i < this.length; i++) {
    if (typeof prev === 'undefined') {
      prev = fn(this[i], this[i+1], i+1, this);
      ++i;
    } else {
      prev = fn(prev, this[i], i, this);
    }
  }
  return prev
}
```

```js
let sum = [1, 2, 3].myReduce((prev, next) => {
  return prev + next
});

console.log(sum); // 6
```

## 20. 手动实现通用柯理化函数

**柯理化函数含义：**是给函数分步传递参数，每次传递部分参数，并返回一个更具体的函数接收剩下的参数，这中间可嵌套多层这样的接收部分参数的函数，直至返回最后结果。

```js
// add的参数不固定，看有几个数字累计相加
function add (a,b,c,d) {
  return a+b+c+d
}

function currying (fn, ...args) {
  // fn.length 回调函数的参数的总和
  // args.length currying函数 后面的参数总和 
  // 如：add (a,b,c,d)  currying(add,1,2,3,4)
  if (fn.length === args.length) {  
    return fn(...args)
  } else {
    // 继续分步传递参数 newArgs 新一次传递的参数
    return function anonymous(...newArgs) {
      // 将先传递的参数和后传递的参数 结合在一起
      let allArgs = [...args, ...newArgs]
      return currying(fn, ...allArgs)
    }
  }
}

let fn1 = currying(add, 1, 2) // 3
let fn2 = fn1(3)  // 6
let fn3 = fn2(4)  // 10
```

## 21. 手动实现发布订阅

> 发布订阅的核心:： 每次event. emit（发布），就会触发一次event. on（注册）

```js
class EventEmitter {
  constructor() {
    // 事件对象，存放订阅的名字和事件
    this.events = {};
  }
  // 订阅事件的方法
  on(eventName,callback) {
    if (!this.events[eventName]) {
      // 注意数据，一个名字可以订阅多个事件函数
      this.events[eventName] = [callback];
    } else  {
      // 存在则push到指定数组的尾部保存
      this.events[eventName].push(callback)
    }
  }
  // 触发事件的方法
  emit(eventName) {
    // 遍历执行所有订阅的事件
    this.events[eventName] && this.events[eventName].forEach(cb => cb());
  }
}
```

- 测试用例

```js
let em = new EventEmitter();

function workDay() {
  console.log("每天工作");
}
function makeMoney() {
    console.log("赚100万");
}
function sayLove() {
  console.log("向喜欢的人示爱");
}
em.on("money",makeMoney);
em.on("love",sayLove);
em.on("work", workDay);

em.emit("money");
em.emit("love");  
em.emit("work");  
```

## 22. 手动实现观察者模式

> 观察者模式（基于发布订阅模式） 有观察者，也有被观察者

观察者需要放到被观察者中，被观察者的状态变化需要通知观察者 我变化了 内部也是基于发布订阅模式，收集观察者，状态变化后要主动通知观察者

```js
class Subject { // 被观察者 学生
  constructor(name) {
    this.state = '开心的'
    this.observers = []; // 存储所有的观察者
  }
  // 收集所有的观察者
  attach(o){ // Subject. prototype. attch
    this.observers.push(o)
  }
  // 更新被观察者 状态的方法
  setState(newState) {
    this.state = newState; // 更新状态
    // this 指被观察者 学生
    this.observers.forEach(o => o.update(this)) // 通知观察者 更新它们的状态
  }
}

class Observer{ // 观察者 父母和老师
  constructor(name) {
    this.name = name
  }
  update(student) {
    console.log('当前' + this.name + '被通知了', '当前学生的状态是' + student.state)
  }
}

let student = new Subject('学生'); 

let parent = new Observer('父母'); 
let teacher = new Observer('老师'); 

// 被观察者存储观察者的前提，需要先接纳观察者
student. attach(parent); 
student. attach(teacher); 
student. setState('被欺负了');
```

## 23. 手动实现Object.freeze(**有问题**)

Object.freeze冻结一个对象，让其不能再添加/删除属性，也不能修改该对象已有属性的可枚举性、可配置可写性，也不能修改已有属性的值和它的原型属性，最后返回一个和传入参数相同的对象。

```js
function myFreeze(obj){
  // 判断参数是否为Object类型，如果是就封闭对象，循环遍历对象。去掉原型属性，将其writable特性设置为false
  if(obj instanceof Object){
    Object.seal(obj);  // 封闭对象
    for(let key in obj){
      if(obj.hasOwnProperty(key)){
        Object.defineProperty(obj,key,{
          writable:false   // 设置只读
        })
        // 如果属性值依然为对象，要通过递归来进行进一步的冻结
        myFreeze(obj[key]);  
      }
    }
  }
}
```

## 24. 手动实现Promise.all

- Promise.all：有一个promise任务失败就全部失败

- Promise.all方法返回的是一个promise

```js
function isPromise (val) {
  return typeof val.then === 'function'; // (123).then => undefined
}

Promise.all = function(promises) {
  return new Promise((resolve, reject) => {
    let arr = []; // 存放 promise执行后的结果
    let index = 0; // 计数器，用来累计promise的已执行次数
    const processData = (key, data) => {
      arr[key] = data; // 不能使用数组的长度来计算
      /*
        if (arr.length == promises.length) {
          resolve(arr);  // [null, null , 1, 2] 由于Promise异步比较慢，所以还未返回
        }
      */
     if (++index === promises.length) {
      // 必须保证数组里的每一个
       resolve(arr);
     }
    }
    // 遍历数组依次拿到执行结果
    for (let i = 0; i < promises.length; i++) {
      let result = promises[i];
      if(isPromise(result)) {
        // 让里面的promise执行，取得成功后的结果
        // data promise执行后的返回结果
        result.then((data) => {
          // 处理数据，按照原数组的顺序依次输出
          processData(i ,data)
        }, reject)  // reject本事就是个函数 所以简写了
      } else {
        // 1 , 2
        processData(i ,result)
      }
    }
  })
}
```

- 测试用例

```js
let fs = require('fs').promises;

let getName = fs.readFile('./name.txt', 'utf8');
let getAge = fs.readFile('./age.txt', 'utf8');

Promise.all([1, getName, getAge, 2]).then(data => {
	console.log(data); // [ 1, 'name', '11', 2 ]
})
```

## 25. 手写题：在线编程，getUrlParams(url,key); 就是很简单的获取url的某个参数的问题，但要考虑边界情况，多个返回值等等


## 19.
## 19.
## 19.
## 19.
## 19.
## 19.