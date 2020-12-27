<!--
 * @Author: Li Zhiliang
 * @Date: 2020-12-26 09:38:26
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-12-26 09:59:26
 * @FilePath: /FE-Interview.git/javascript/this.md
-->
# this关键字

## this是什么？

this是一个指针，指向调用函数的对象。

概念：执行上下文，this一般存在于函数中，表示当前函数的执行上下文， 如果函数没有执行，那么this没有内容，只有函数在执行后this才有绑定。

> 划重点：this的指向只能是一个对象，不要忘记Array也是一类特殊的对象哦

## this的绑定规则

### 1. 默认绑定(非严格模式情况下，this 指向 window, 严格模式下，this指向 undefined。)

- 默认绑定，在不能应用其它绑定规则时使用的默认规则，通常是独立函数调用。

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var name = 'YvetteLau';
sayHi();
// 在调用Hi()时，应用了默认绑定，this指向全局对象（非严格模式下），严格模式下，this指向undefined，undefined上没有this对象，会抛出错误。

// 在浏览器环境中运行，那么结果就是 Hello,YvetteLau

// 在node环境中运行，结果就是 Hello,undefined.这是因为node中name并不是挂在全局对象上的。
```


### 2. 隐式绑定(如果函数调用时，前面存在调用它的对象，那么this就会隐式绑定到这个对象上)

- 函数的调用是在某个对象上触发的，即调用位置上存在上下文对象。典型的形式为 XXX.fun().我们来看一段代码：

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
person.sayHi(); //  Hello,YvetteLau.

// sayHi函数声明在外部，严格来说并不属于person，但是在调用sayHi时,调用位置会使用person的上下文来引用函数，隐式绑定会把函数调用中的this(即此例sayHi函数中的this)绑定到这个上下文对象（即此例中的person）
```

- 需要注意的是：对象属性链中只有最后一层会影响到调用位置。

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person2 = {
    name: 'Christina',
    sayHi: sayHi
}
var person1 = {
    name: 'YvetteLau',
    friend: person2
}
person1.friend.sayHi(); // Hello, Christina.

// 因为只有最后一层会确定this指向的是什么，不管有多少层，在判断this的时候，我们只关注最后一层，即此处的friend。
```

- 隐式绑定有一个大陷阱，绑定很容易丢失(或者说容易给我们造成误导，我们以为this指向的是什么，但是实际上并非如此).

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = person.sayHi;
Hi(); // Hello,Wiliam.

// Hi直接指向了sayHi的引用，在调用的时候，跟person没有半毛钱的关系，针对此类问题，我建议大家只需牢牢继续这个格式:XXX.fn();fn()前如果什么都没有，那么肯定不是隐式绑定，但是也不一定就是默认绑定
```

- 除了上面这种丢失之外，隐式绑定的丢失是发生在回调函数中(事件回调也是其中一种)，我们来看下面一个例子:

    ```js
    function sayHi(){
        console.log('Hello,', this.name);
    }
    var person1 = {
        name: 'YvetteLau',
        sayHi: function(){
            setTimeout(function(){
                console.log('Hello,',this.name);
            })
        }
    }
    var person2 = {
        name: 'Christina',
        sayHi: sayHi
    }
    var name='Wiliam';
    person1.sayHi();
    setTimeout(person2.sayHi,100);
    setTimeout(function(){
        person2.sayHi();
    },200);

    // 结果为
    // Hello, Wiliam
    // Hello, Wiliam
    // Hello, Christina
    ```

    - 第一条输出很容易理解，setTimeout的回调函数中，this使用的是默认绑定，非严格模式下，执行的是全局对象

    - 第二条输出我们可以这样理解: setTimeout(fn,delay){ fn(); },相当于是将person2.sayHi赋值给了一个变量，最后执行了变量，这个时候，sayHi中的this显然和person2就没有关系了。

    - 第三条虽然也是在setTimeout的回调中，但是我们可以看出，这是执行的是person2.sayHi()使用的是隐式绑定，因此这是this指向的是person2，跟当前的作用域没有任何关系。


### 3. 显式绑定（硬绑定）(函数通过 call()、apply()、bind()调用，this 指向被绑定的对象。)

- 显式绑定比较好理解，就是通过call,apply,bind的方式，显式的指定this所指向的对象。(注意:《你不知道的Javascript》中将bind单独作为了硬绑定讲解了)
    
    - 临时改变this指向   =>   call apply
    
    - 永久改变this指向   =>   bind

- call,apply和bind的第一个参数，就是对应函数的this所指向的对象。
    
    1. call和apply的作用一样，只是传参方式不同。(call() 和 apply() 都是 立即执行函数 ，但是它们接受的参数的形式不同)

    ```js
    call(this, arg1, arg2, ...)
    apply(this, [arg1, arg2, ...])
    ```

    2. call和apply都会执行对应的函数，而bind方法不会。 bind() 则是 返回一个新的包装函数，而不是立刻执行。bind()会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。
    ```js
    bind(this, arg1, arg2, ...)
    ```

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = person.sayHi;
Hi.call(person); //Hi.apply(person)
// 结果为Hello, YvetteLau. 
```

- 绑定丢失

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = function(fn) {
    fn();
}
Hi.call(person, person.sayHi);  // Hello, Wiliam

// 原因很简单，Hi.call(person, person.sayHi)的确是将this绑定到Hi中的this了。但是在执行fn的时候，相当于直接调用了sayHi方法(记住: person.sayHi已经被赋值给fn了，隐式绑定也丢了)，没有指定this的值，对应的是默认绑定。
```

- 解决绑定丢失问题: 调用fn的时候，也给它硬绑定

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = function(fn) {
    fn.call(this);
}
Hi.call(person, person.sayHi); // Hello, YvetteLau

// 因为person被绑定到Hi函数中的this上，fn又将这个对象绑定给了sayHi的函数。这时，sayHi中的this指向的就是person对象。

```


### 4. new绑定(函数被 new 调用，this 指向由 new 新构造出来的这个对象。)

- 在javaScript中，构造函数只是使用new操作符时被调用的函数，这些函数和普通的函数并没有什么不同，它不属于某个类，也不可能实例化出一个类。任何一个函数都可以使用new来调用，因此其实并不存在构造函数，而只有对于函数的“构造调用”。

- 使用new来调用函数，会自动执行下面的操作：
    1. 创建（或者说构造）一个全新的对象。
    
    2. 这个新对象会被执行 Prototype 连接。
    
    3. 将构造函数的作用域赋值给新对象，即this指向这个新对象
    
    4. 执行构造函数中的代码
    
    5. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

- 我们使用new来调用函数的时候，就会新对象绑定到这个函数的this上。

```js
function sayHi(name){
    this.name = name;
	
}
var Hi = new sayHi('Yevtte');
console.log('Hello,', Hi.name);
// Hello, Yevtte, 原因是因为在var Hi = new sayHi('Yevtte');这一步，会将sayHi中的this绑定到Hi对象上。
```

## 绑定优先级

- 四种绑定的优先级为:

new绑定 > 显式绑定(硬绑定) > 隐式绑定 > 默认绑定


## 绑定例外

- 如果我们将null或者是undefined作为this的绑定对象传入call、apply或者是bind,这些值在调用时会被忽略，实际应用的是默认绑定规则。

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = function(fn) {
    fn();
}
Hi.call(null, person.sayHi); 
// 输出的结果是 Hello, Wiliam，因为这时实际应用的是默认绑定规则。
```

## 箭头函数

- 箭头函数是ES6中新增的，它和普通函数有一些区别，箭头函数没有自己的this，它的this继承于外层代码库中的this。箭头函数在使用时，需要注意以下几点:

    1. 函数体内的this对象，继承的是外层代码块的this。
    
    2. 不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。
    3. 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。
    4. 不可以使用yield命令，因此箭头函数不能用作 Generator 函数。
    5. 箭头函数没有自己的this，所以不能用call()、apply()、bind()这些方法去改变this的指向.
    OK，我们来看看箭头函数的this是什么？

```js
var obj = {
    hi: function(){
        console.log(this);
        return ()=>{
            console.log(this);
        }
    },
    sayHi: function(){
        return function() {
            console.log(this);
            return ()=>{
                console.log(this);
            }
        }
    },
    say: ()=>{
        console.log(this);
    }
}
let hi = obj.hi();  //输出obj对象
hi();               //输出obj对象
let sayHi = obj.sayHi();
let fun1 = sayHi(); //输出window
fun1();             //输出window
obj.say();          //输出window
```

- 我们来分析一下上面的执行结果：

    1. obj.hi(); 对应了this的默认绑定规则，this绑定在obj上，所以输出obj，很好理解。
    
    2. hi(); 这一步执行的就是箭头函数，箭头函数继承上一个代码库的this，刚刚我们得出上一层的this是obj，显然这里的this就是obj.
    
    3. 执行sayHi();这一步也很好理解，我们前面说过这种隐式绑定丢失的情况，这个时候this执行的是默认绑定，this指向的是全局对象window.
    
    4. fun1(); 这一步执行的是箭头函数，如果按照之前的理解，this指向的是箭头函数定义时所在的对象，那么这儿显然是说不通。OK，按照箭头函数的this是继承于外层代码库的this就很好理解了。外层代码库我们刚刚分析了，this指向的是window，因此这儿的输出结果是window.
    
    5. obj.say(); 执行的是箭头函数，当前的代码块obj中是不存在this的，只能往上找，就找到了全局的this，指向的是window.

- 箭头函数没有自己的this，箭头函数中的this继承于外层代码库中的this.箭头函数的this不是静态的

```js
var obj = {
    hi: function(){
        console.log(this);
        return ()=>{
            console.log(this);
        }
    },
    sayHi: function(){
        return function() {
            console.log(this);
            return ()=>{
                console.log(this);
            }
        }
    },
    say: ()=>{
        console.log(this);
    }
}
let sayHi = obj.sayHi();
let fun1 = sayHi(); //输出window
fun1();             //输出window

let fun2 = sayHi.bind(obj)();//输出obj
fun2();                      //输出obj
```

## 判断this指向

- 在 ES5 中， this 的指向始终是一个原则：this 的指向并不是在创建的时候就可以确定的，在 es5 中， this 永远指向 最后调用它的那个对象 。

1. 函数是否在new中调用(new绑定)，如果是，那么this绑定的是新创建的对象。

2. 函数是否通过call,apply调用，或者使用了bind(即硬绑定)，如果是，那么this绑定的就是指定的对象。

3. 函数是否在某个上下文对象中调用(隐式绑定)，如果是的话，this绑定的是那个上下文对象。一般是obj.foo()

4. 如果以上都不是，那么使用默认绑定。如果在严格模式下，则绑定到undefined，否则绑定到全局对象。

5. 如果把Null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认绑定规则。

6. 如果是箭头函数，箭头函数的this继承的是外层代码块的this。

## 执行过程解析实例

```js
var number = 5;
var obj = {
    number: 3,
    fn: (function () {
        var number;
        this.number *= 2;
        number = number * 2;
        number = 3;
        return function () {
            var num = this.number;
            this.number *= 2;
            console.log(num);
            number *= 3;
            console.log(number);
        }
    })()
}
var myFun = obj.fn;
myFun.call(null);
obj.fn();
console.log(window.number);

```

1. 在定义obj的时候，fn对应的闭包就执行了，返回其中的函数，执行闭包中代码时，显然应用不了new绑定(没有出现new 关键字)，硬绑定也没有(没有出现call,apply,bind关键字),隐式绑定有没有？很显然没有，如果没有XX.fn()，那么可以肯定没有应用隐式绑定，所以这里应用的就是默认绑定了，非严格模式下this绑定到了window上(浏览器执行环境)。【这里很容易被迷惑的就是以为this指向的是obj，一定要注意，除非是箭头函数，否则this跟词法作用域是两回事，一定要牢记在心】

```js
window.number * = 2; //window.number的值是10(var number定义的全局变量是挂在window上的)

number = number * 2; //number的值是NaN;注意我们这边定义了一个number，但是没有赋值，number的值是undefined;Number(undefined)->NaN

number = 3;          //number的值为3
```

2. myFun.call(null);我们前面说了，call的第一个参数传null，调用的是默认绑定;

```js
fn: function(){
    var num = this.number;
    this.number *= 2;
    console.log(num);
    number *= 3;
    console.log(number);
}
// 执行时
var num = this.number; //num=10; 此时this指向的是window
this.number * = 2;     //window.number = 20
console.log(num);      //输出结果为10
number *= 3;           //number=9; 这个number对应的闭包中的number;闭包中的number的是3
console.log(number);   //输出的结果是9

```

3. obj.fn();应用了隐式绑定，fn中的this对应的是obj.

```js
var num = this.number;//num = 3;此时this指向的是obj
this.number *= 2;     //obj.number = 6;
console.log(num);     //输出结果为3;
number *= 3;          //number=27;这个number对应的闭包中的number;闭包中的number的此时是9
console.log(number);  //输出的结果是27

```

4. 最后一步console.log(window.number);输出的结果是20

```
10
9
3
27
20
```

## 手动实现call

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

- 简易版（不考虑context非对象情况，不考虑Symbol\BigInt 不能 new.constructor( context )情况）

```js
/**
 * context: 要改变的函数中的this指向，写谁就是谁
 * args：传递给函数的实参信息
 * this：要处理的函数 fn
 */
Function.prototype.call = function(context, ...args) {
	//  null，undefined，和不传时，context为 window
  context = context == null ? window : context;
  
  let result;
  context['fn'] = this; // 把函数作为对象的某个成员值
  result = context['fn'](...args); // 把函数执行，此时函数中的this就是
	delete context['fn']; // 设置完成员属性后，删除
  return result;
}
```

- 完善版（context必须对象类型，兼容Symbol等情况）

```js
/**
 * context: 要改变的函数中的this指向，写谁就是谁
 * args：传递给函数的实参信息
 * this：要处理的函数 fn
 */
Function.prototype.call = function(context, ...args) {
	//  null，undefined，和不传时，context为 window
  context = context == null ? window : context;
  
  // 必须保证 context 是一个对象类型
  let contextType = typeof context;
  if (!/^(object|function)$/i.test(contextType)) {
    // context = new context.constructor(context); // 不适用于 Symbol/BigInt
  	context = Object(context);
	}
  
  let result;
  context['fn'] = this; // 把函数作为对象的某个成员值
  result = context['fn'](...args); // 把函数执行，此时函数中的this就是
	delete context['fn']; // 设置完成员属性后，删除
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

## 手写apply

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

```js
/**
 * context: 要改变的函数中的this指向，写谁就是谁
 * args：传递给函数的实参信息
 * this：要处理的函数 fn
 */
Function.prototype.apply = function(context, args) {

  context = context == null ? window : context;
  
  let contextType = typeof context;
  if (!/^(object|function)$/i.test(contextType)) {
  	context = Object(context);
	}
  
  let result;
  context['fn'] = this; 
  result = context['fn'](...args); 
	delete context['fn'];
  return result;
}
```

## 手写bind

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

```js

/**
 * this: 要处理的函数 func
 * context: 要改变的函数中的this指向 obj
 * params：要处理的函数传递的实参 [10, 20]
 */
Function.prototype._bind = function(context, ...params) {
  
  let _this = this; // this: 要处理的函数
  return function anonymous (...args) {
    // args： 可能传递的事件对象等信息 [MouseEvent]
    // this：匿名函数中的this是由当初绑定的位置 触发决定的 （总之不是要处理的函数func）
    // 所以需要_bind函数 刚进来时，保存要处理的函数 _this = this
    _this.call(context, ...params.concat(args));
  }
}
```

