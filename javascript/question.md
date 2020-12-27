# Question

## 1. 下面代码的输出是什么?

```js
class Chameleon {
  static colorChange(newColor) {
    this.newColor = newColor;
  }

  constructor({ newColor = "green" } = {}) {
    this.newColor = newColor;
  }
}

const freddie = new Chameleon({ newColor: "purple" });
freddie.colorChange("orange");
```

> colorChange方法是静态的。 静态方法仅在创建它们的构造函数中存在，并且不能传递给任何子级。 由于freddie是一个子级对象，函数不会传递，所以在freddie实例上不存在freddie方法：抛出TypeError。

## 2. 下面代码的输出是什么?

```js
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}

const member = new Person("Lydia", "Hallie");
Person.getFullName = () => this.firstName + this.lastName;

console.log(member.getFullName());
```

**结果： TypeError**

> 您不能像使用常规对象那样向构造函数添加属性。 如果要一次向所有对象添加功能，则必须使用原型。 所以在这种情况下应该这样写：

```js
Person.prototype.getFullName = function () {
  return `${this.firstName} ${this.lastName}`;
}
```

这样会使member.getFullName()是可用的，为什么样做是对的？ 假设我们将此方法添加到构造函数本身。 也许不是每个Person实例都需要这种方法。这会浪费大量内存空间，因为它们仍然具有该属性，这占用了每个实例的内存空间。 相反，如果我们只将它添加到原型中，我们只需将它放在内存中的一个位置，但它们都可以访问它！

## 3. 下面代码的输出是什么?

```js
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}

const lydia = new Person("Lydia", "Hallie");
const sarah = Person("Sarah", "Smith");

console.log(lydia);
console.log(sarah);
```

**Person {firstName: "Lydia", lastName: "Hallie"} and undefined**

> 对于sarah，我们没有使用new关键字。 使用new时，它指的是我们创建的新空对象。 但是，如果你不添加new它指的是全局对象！

我们指定了this.firstName等于'Sarah和this.lastName等于Smith。 我们实际做的是定义global.firstName ='Sarah'和global.lastName ='Smith。 sarah本身的返回值是undefined。

## 4. 下面代码的输出是什么?

```js
function getPersonInfo(one, two, three) {
  console.log(one);
  console.log(two);
  console.log(three);
}

const person = "Lydia";
const age = 21;

getPersonInfo`${person} is ${age} years old`;
```

**["", "is", "years old"] Lydia 21**

> 如果使用标记的模板字符串，则第一个参数的值始终是字符串值的数组。 其余参数获取传递到模板字符串中的表达式的值！

## 5. 下面代码的输出是什么?

```js
const a = {};
const b = { key: "b" };
const c = { key: "c" };

a[b] = 123;
a[c] = 456;

console.log(a[b]);
```

**456**

对象键自动转换为字符串。我们试图将一个对象设置为对象a的键，其值为123。

但是，当对象自动转换为字符串化时，它变成了[Object object]。 所以我们在这里说的是a["Object object"] = 123。 然后，我们可以尝试再次做同样的事情。 c对象同样会发生隐式类型转换。那么，a["Object object"] = 456。

然后，我们打印a[b]，它实际上是a["Object object"]。 我们将其设置为456，因此返回456。

## 6. 单击按钮时event.target是什么?

```js
<div onclick="console.log('first div')">
  <div onclick="console.log('second div')">
    <button onclick="console.log('button')">
      Click!
    </button>
  </div>
</div>
```

**button**

> 导致事件的最深嵌套元素是事件的目标。 你可以通过event.stopPropagation停止冒泡

## 7. 单击下面的html片段打印的内容是什么?

```js
<div onclick="console.log('div')">
  <p onclick="console.log('p')">
    Click here!

</div>
```

**p div**

> 如果我们单击p，我们会看到两个日志：p和div。在事件传播期间，有三个阶段：捕获，目标和冒泡。 默认情况下，事件处理程序在冒泡阶段执行（除非您将useCapture设置为true）。 它从最深的嵌套元素向外延伸。

## 8. 下面代码的输出是什么?

```js
const person = { name: "Lydia" };

function sayHi(age) {
  console.log(`${this.name} is ${age}`);
}

sayHi.call(person, 21);
sayHi.bind(person, 21);
```

**Lydia is 21 function**

> 使用两者，我们可以传递我们想要this关键字引用的对象。 

- 但是，.call方法会立即执行！

- .bind方法会返回函数的拷贝值，但带有绑定的上下文！ 它不会立即执行。

## 9. 下面代码的输出是什么?

```js
(() => {
  let x, y;
  try {
    throw new Error();
  } catch (x) {
    (x = 1), (y = 2);
    console.log(x);
  }
  console.log(x);
  console.log(y);
})();
```

**1 undefined 2**

catch块接收参数x。当我们传递参数时，这与变量的x不同。这个变量x是属于catch作用域的。

之后，我们将这个块级作用域的变量设置为1，并设置变量y的值。 现在，我们打印块级作用域的变量x，它等于1。

在catch块之外，x仍然是undefined，而y是2。 当我们想在catch块之外的console.log(x)时，它返回undefined，而y返回2。

## 10. 冒泡排序算法和数组去重

**冒泡排序**

```js
function bubbleSort (arr) {
  for (let i = 0; i < arr.length; i++) {
    let flag = true;
    for (let j = 0; j < arr.length - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        flag = false;
        let temp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = temp;
      }
    }
    if (flag) break;
  }
  return arr;
}
```

> 冒泡排序总会执行(N-1)+(N-2)+(N-3)+..+2+1趟，但如果运行到当中某一趟时排序已经完成，或者输入的是一个有序数组，那么后边的比较就都是多余的，为了避免这种情况，我们增加一个flag，判断排序是否在中途就已经完成（也就是判断有无发生元素交换）

**数组去重**

定义去重数据

```js
let arr = [1, 1, "1", "1", null, null, undefined, undefined, /a/, /a/, NaN, NaN, {}, {}, [], []]
```

1. Array.from(new Set(arr))

```js
var arr = [1,1,2,5,6,3,5,5,6,8,9,8];
console.log(Array.from(new Set(arr)))
// console.log([...new Set(arr)])
```

2. [...new Set(arr)]: 引用数据类型并没有能成功去重，只能去除基本数据类型

```js
// 使用 Set
let res = [...new Set(arr)]
console.log(res)
```

  - 同理：filter 和 reduce 两种方法也和上面的方法一样，不能去掉引用数据类型。

  ```js
  //使用filter
  let res = arr.filter((item, index) => {
    return arr.indexOf(item) === index
  })
  console.log(res)

  //使用reduce
  let res = arr.reduce((pre, cur) => {
    return pre.includes(cur) ? pre : [...pre, cur]
  }, [])
  console.log(res)
  ```


3. for循环嵌套，利用splice去重

```js
function unique (origin) {
  let arr = [].concat(origin);
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] == arr[j]) {
        arr.splice(j, 1);
        j--;
      }
    }
  }
  return arr;
}
var arr = [1,1,2,5,6,3,5,5,6,8,9,8];
console.log(unique(arr))
```

4. 新建数组，利用indexOf或者includes去重

```js
function unique (arr) {
  let res = []
  for (let i = 0; i < arr.length; i++) {
    if (!res.includes(arr[i])) {
      res.push(arr[i])
    }
  }
  return res;
}
var arr = [1,1,2,5,6,3,5,5,6,8,9,8];
console.log(unique(arr))
```

5. 先用sort排序，然后用一个指针从第0位开始，配合while循环去重

```js
function unique (arr) {
  arr = arr.sort(); // 排序之后的数组
  let pointer = 0;
  while (arr[pointer]) {
    if (arr[pointer] != arr[pointer + 1]) { // 若这一项和下一项不相等则指针往下移
      pointer++;
    } else { // 否则删除下一项
      arr.splice(pointer + 1, 1);
    }
  }
  return arr;
}
var arr = [1,1,2,5,6,3,5,5,6,8,9,8];
console.log(unique(arr))
```

6. hasOwnProperty: 利用对象的hasOwnProperty方法进行判断对象上是否含有该属性，如果含有则过滤掉，不含有则返回新数组中(可以去除引用类型)

```js
let obj = {}
let res = arr.filter(item => {
  if (obj.hasOwnProperty(typeof item + item)) {
    return false
  } else {
    obj[typeof item + item] = true
    return true
  }
})
console.log(res)
```

## 11. 手写new

```js
function myNew (fn, ...args) {
  let instance = Object.create(fn.prototype);
  let result = fn.call(instance, ...args)
  return typeof result === 'object' ? result : instance;
}
```

## 12. 手写instanceof

instanceof它主要是用于检测某个构造函数的原型对象在不在某个对象的原型链上。

```js
function myInstanceof(left,right) {
    if(typeof left !== 'object' || left === null) return false
    //获取原型
    let proto = Object.getPrototypeOf(left)
    while(true){
        //如果原型为null，则已经到了原型链顶端，判断结束
        if(proto === null) return false
        //左边的原型等于右边的原型，则返回结果
        if(proto === right.prototype) return true
        //否则就继续向上获取原型
        proto = Object.getPrototypeOf(proto)
    }
}
```

## 13. 写一个防抖函数

## 14. for循环setTimeout打印输出

如果不采用立即执行函数或者let的形式就会直接打印出10个10，通过采取闭包或者let有了块级作用域之后就不会出现这样的问题

- 立即执行函数

```js
for (var i = 0; i < 10; i++) {
  (function (j) {
    setTimeout(() => {
      console.log(j)
    }, 1000)
  })(i)
}
```

- 闭包：给定时器传入第三个参数, 定时器可以传多个参数给定时器函数，此处将外层的i传递给了定时器中回调函数作为参数使用。

```js
for(var i = 1;i <= 5; i++){
  setTimeout(function timer(j){
    console.log(j)
  }, 0, i)
}
```

- 用let给定块级作用域

```js
for (let i = 0; i < 10; i++) {
  setTimeout(() => {
    console.log(i)
  }, 1000)
}
```

## 15. bind相关

```js
let obj = { a: 1 }
let obj2 = { a: 2 }
let obj3 = { a: 3 }
let obj4 = { a: 4 }

function foo() {
  console.log(this.a)
}

let boundFn = foo.bind(obj).bind(obj2).bind(obj3)
boundFn.call(obj4)  //打印结果为1
boundFn.apply(obj4) //打印结果为1
boundFn()  //打印结果为1
```

- 由此我们可以看出bind是永久绑定，往后的操作都不会再更改其指向

## 16. var arr =[[‘A’,’B’],[‘a’,’b’],[1,2]] 求二维数组的全排列组合 结果：Aa1,Aa2,Ab1,Ab2,Ba1,Ba2,Bb1,Bb2

### 实现方式一

```js
function foo(arr) {
  // 用于记录初始数组长度, 用于将数组前两组已经获取到全排列的数组进行截取标识
  var len = arr.length;
  // 当递归操作后, 数组长度为1时, 直接返回arr[0], 只有大于1继续处理
  if (len >= 2) {
    // 每次只做传入数组的前面两个数组进行全排列组合, 即arr[0]和arr[1]的全排列组合
    var len1 = arr[0].length;
    var len2 = arr[1].length;
    var items = new Array(len1 * len2); // 创建全排列组合有可能次数的数组
    var index = 0; // 记录每次全排列组合后的数组下标
    for (var i = 0; i < len1; i++) {
      for (var j = 0; j < len2; j++) {
        if (Array.isArray(arr[0])) {
          // 当第二次进来后, 数组第一个元素必定是数组包着数组
          items[index] = arr[0][i].concat(arr[1][j]); // 对于已经是第二次递归进来的全排列直接追加即可
        } else {
          items[index] = [arr[0][i]].concat(arr[1][j]); // 这里因为只需要去arr[0]和arr[1]的全排列, 所以这里就直接使用concat即可
        }
        index++; // 更新全排列组合的下标
      }
    }
    // 如果数组大于2, 这里新的newArr做一个递归操作
    var newArr = new Array(len - 1); // 递归的数组比传进来的数组长度少一, 因为上面已经将传进来的数组的arr[0]和arr[1]进行全排列组合, 所以这里的newArr[0]就是上面已经全排列好的数组item
    for (var i = 2; i < arr.length; i++) {
      // 这里的for循环是为了截取下标1项后的数组进行赋值给newArr
      newArr[i - 1] = arr[i];
    }
    newArr[0] = items; // 因为上面已经将传进来的数组的arr[0]和arr[1]进行全排列组合, 所以这里的newArr[0]就是上面已经全排列好的数组item
    // 重新组合后的数组进行递归操作
    return foo(newArr);
  } else {
    // 当递归操作后, 数组长度为1时, 直接返回arr[0],
    return arr[0];
  }
}
var arr = [
  ["A", "B"],
  ["a", "b"],
  [1, 2],
];
console.log(foo(arr));
```

### 实现方式二

```js
const getResult = (arr1, arr2) => {
  if (!Array.isArray(arr1) || !Array.isArray(arr2)) {
    return;
  }
  if (!arr1.length) {
    return arr2;
  }
  if (!arr2.length) {
    return arr1;
  }
  let result = [];
  for (let i = 0; i < arr1.length; i++) {
    for (let j = 0; j < arr2.length; j++) {
      result.push(String(arr1[i]) + String(arr2[j]));
    }
  }
  return result;
};

const findAll = (arr) =>
  arr.reduce((total, current) => {
    return getResult(total, current);
  }, []);
var arr = [
  ["A", "B"],
  ["a", "b"],
  [1, 2],
];
console.log(findAll(arr));
```

### 实现方式3：

```js
var arr = [
  ["A", "B"],
  ["a", "b"],
  [1, 2],
];
let res = [],
  lengthArr = arr.map((d) => d.length);
let indexArr = new Array(arr.length).fill(0);
function addIndexArr() {
  indexArr[0] = indexArr[0] + 1;
  let i = 0;
  let overflow = false;
  while (i <= indexArr.length - 1) {
    if (indexArr[i] >= lengthArr[i]) {
      if (i < indexArr.length - 1) {
        indexArr[i] = 0;
        indexArr[i + 1] = indexArr[i + 1] + 1;
      } else {
        overflow = true;
      }
    }
    i++;
  }
  return overflow;
}
function getAll(arr, indexArr) {
  let str = "";
  arr.forEach((item, index) => {
    str += item[indexArr[index]];
  });
  res.push(str);
  let overflow = addIndexArr();
  if (overflow) {
    return;
  } else {
    return getAll(arr, indexArr);
  }
}
getAll(arr, indexArr);
console.log(res);
```

## 17. 一段代码带你论证JS基础

```js
var b = 10;
(function b(){
    b = 20;
    console.log(b); 
})();
```

- 这段代码的在严格模式下输出:TypeError: Assignment to constant variable

可以确定的是，如果b是指外部用var声明的，那么在此代码中它一定是可修改的。我们都知道用var声明的全局变量它在任何地方肯定是可以修改的，因为该变量处于作用域的最顶端。

所以在这里我想大胆做个假设：变量b是指立即执行的具名函数名称b~

做完这个假设，我想说：那意味着全局变量b在立即执行的具名函数b里访问不到吗？

其实不是的，全局变量b在立即执行的具名函数b是可被访问的，只不过因为具名函数b的内部作用域里也存在了一个用function声明的变量b，所以在代码执行的时候js引擎首先找到用function声明的变量b。正如书籍中讲到的作用域查询是通过从里到外向上查询。

当然在非严格模式下，大家可以试着动手在立即执行的具名函数内部函数打印一下window.b,也可论证全局变量b在立即执行的具名函数b里可被访问！ 代码如下：

```js
var b = 10;
(function b(){
    b = 20;
    console.log(window.b); 
})();
```

我所理解的是：当遇到具名的函数表达式的时，会创建一个辅助的特定对象，将函数表达式的名称作为唯一的key，用来存储函数表达式的名称，然后添加到函数的作用域链中，该值只读，并且不可以被删除，所以不能对该值进行操作。

- 而在非严格模式下输出:

```js
ƒ b(){
    b = 20;
    console.log(b); 
}
```

在非严格模式下会静默失败，所以不报错。针对如上的分析之后，会输出立即执行的具名函数b本身。


## 18. 请说出以下代码输出的内容，需要区分nodejs,chrome以及chrome去掉splice之后的输出内容

```js
var obj = {
    '2': 3,
    '3': 4,
    'length': 2,
    'splice': Array.prototype.splice,
    'push': Array.prototype.push
}
obj.push(1)
obj.push(2)
console.log(obj)
```

1. node下面输出

```js
{ '2': 1,
  '3': 2,
  length: 4,
  splice: [Function: splice],
  push: [Function: push] }
```

2. chrome下面输出

```js
[empty × 2, 1, 2, splice: ƒ, push: ƒ]
```

3. chrome去掉splice下面输出

```js
{2: 1, 3: 2, length: 4, push: ƒ}
```

### 题解

在解答题目之前，我们再看看这段代码

```js
const arr = new Array(2)
// 输出  2 [empty * 2]
console.log(arr.length, arr)
arr.push(1)
// 输出  3 [empty * 2, 1]
console.log(arr.length, arr)
```

可以看到push方法会将数组的length + 1, 然后将值放在索引为length - 1的位置,比如上面的代码，因为在初始化数组的时候，已经将数组长度指定为了2, 所以在push之后length就变成了3,然后arr[3 - 1] = 1

在MDN上面对push的方法的解释是:

> push 方法具有通用性。该方法和 call() 或 apply() 一起使用时，可应用在类似数组的对象上。push 方法根据 length 属性来决定从哪里开始插入给定的值。如果 length 不能被转成一个数值，则插入的元素索引为 0，包括 length 不存在时。当 length 不存在时，将会创建它。

根据MDN解释，push既可以使用到数组中，也可以使用到类数组中。而根据前文中对类数组的解释，可以看到题目中的obj就是一个标准的类数组，那就可以在obj上面使用数组的push方法。

再看obj.push(1), 因为obj.length = 2, 所以会将length + 1就变成了3, 这时候 索引值时obj[3 - 1] = 1 即obj[2] = 1, 同理 obj.push(2) 也一样的。因为在obj中已经有了属性(索引)2和3，所以在push的时候会覆盖掉2和3上面的默认值。

所以在nodejs中就会输出

```js
{ '2': 1,
  '3': 2,
  length: 4,
  splice: [Function: splice],
  push: [Function: push] }
```

但是在chrome控制台中输出

```js
[empty × 2, 1, 2, splice: ƒ, push: ƒ]
```

很奇怪，为什么会输出这样呢？这一块有一个很特殊的陷阱，就是chrome控制台是如何判断打印的内容是数组还是其他对象呢？对于这个，chrome就是通过判断对象上面是否有splice和length这两个属性来判断的，所以如果你将splice去掉之后，就会输出以下内容

```js
{2: 1, 3: 2, length: 4, push: ƒ}
```

## 19. 有16瓶水，其中只有一瓶水有毒，小白鼠喝一滴之后一小时会死，请问最少用多少只小白鼠，在1小时的时间一定可以找出有毒的水？

答案是至少需要4只小鼠，怎么理解呢？我们可以用二进制去推理一下：

假设有4只小鼠，分别是甲乙丙丁,使用二进制来表示小鼠喝药的顺序，1代表喝药,0代表不喝药

甲: 1111 1111 0000 0000
乙: 1111 0000 1111 0000
丙: 1100 1100 1100 1100
丁: 1010 1010 1010 1010

那么我们就可以这样去判断:

1. 甲乙丙丁都死了，说明第一瓶有毒

2. 甲乙丙死了，说明第二瓶有毒

3. 甲乙丁死了，说明第三瓶有毒

4. 甲乙死了，说明第四瓶有毒

5. 甲丙丁死了，说明第五瓶有毒

6. 。。。 依次类推

其实对于这道题，可以使用2的n次方来判断，比如有32瓶水，那么就是2的5次方，所以就需要5只小鼠。

## 20. this指向问题

this指向问题一直是比较混乱的，在箭头函数出现之前，this的指向与代码在哪里定义并没有关系，而是取决于是被谁执行的，正因为此，所以许多开发人员经常会搞不清楚this到底是谁。下面的两道题都是和this指向相关的问题。

### 题目一（青铜）

```js
var num = 1;
let obj = {
    num: 2,
    add: function() {
        this.num = 3;
        (function() {
            console.log(this.num);
            this.num = 4;
        })();
        console.log(this.num);
    },
    sub: function() {
        console.log(this.num)
    }
}
obj.add();
console.log(obj.num);
console.log(num);
const sub = obj.sub;
sub();
```

输出结果: 1,3,3,4,4， 你答对了吗？下面我们来看看代码解析

```js
var num = 1;
let obj = {
    num: 2,
    add: function() {
        this.num = 3;
      	// 这里的立即指向函数，因为我们没有手动去指定它的this指向，所以都会指向window
        (function() {
            // 所有这个 this.num 就等于 window.num
            console.log(this.num);
            this.num = 4;
        })();
        console.log(this.num);
    },
    sub: function() {
        console.log(this.num)
    }
}
// 下面逐行说明打印的内容

/**
 * 在通过obj.add 调用add 函数时，函数的this指向的是obj,这时候第一个this.num=3
 * 相当于 obj.num = 3 但是里面的立即指向函数this依然是window,
 * 所以 立即执行函数里面console.log(this.num)输出1，同时 window.num = 4
 *立即执行函数之后，再输出`this.num`,这时候`this`是`obj`,所以输出3
 */
obj.add() // 输出 1 3

// 通过上面`obj.add`的执行，obj.name 已经变成了3
console.log(obj.num) // 输出3
// 这个num是 window.num
console.log(num) // 输出4
// 如果将obj.sub 赋值给一个新的变量，那么这个函数的作用域将会变成新变量的作用域
const sub = obj.sub
// 作用域变成了window window.num 是 4
sub() // 输出4
```



### 题目二（黄金）

```js
var num = 10
const obj = {num: 20}
obj.fn = (function (num) {
  this.num = num * 3
  num++
  return function (n) {
    this.num += n
    num++
    console.log(num)
  }
})(obj.num)
var fn = obj.fn
fn(5)
obj.fn(10)
console.log(num, obj.num)
```

输出结果为: 22 23 65 30, 你答对了吗？ 下面我们解析一下

```js
var num = 10
const obj = {num: 20}
obj.fn = (function (num) {
  this.num = num * 3
  num++
  return function (n) {
    this.num += n
    num++
    console.log(num)
  }
})(obj.num)
var fn = obj.fn
fn(5)
obj.fn(10)
console.log(num, obj.num)
```

我们把上面的代码分为以下几步进行分析

1. 先看第三行代码，是一个赋值操作，我们知道赋值操作是从右向左的，而=号右边是一个立即执行函数，所以会优先执行立即执行函数，立即执行函数没有手动指定this,这时候this = window，而立即函数的参数num是传进来的obj.num,所以num参数默认值是 20

2. 第四行相当于window.num = 20 * 3

3. 第五行为传入的参数加一，所以 num = 20 + 1

4. 第六行return了一个函数，而这个函数就是obj.fn的值, 但是因为return的函数引用了立即执行函数里面的num，所以形成了闭包。

```js
obj.fn = function(n) {
  this.num += n
  // 这个num是立即执行函数里面的num
  num++
  console.log(num)
}
```

5. var fn = obj.fn, 将obj.fn赋值给新的变量，而这个变量的作用域是window

6. 在调用fn(5)的时候， 在第二步，window.num的值已经变成了60, 然后因为这时候fn的this是window, this.num += n相当于window.num += n, 即window.num = 65

7. num++, 因为闭包的原因，第三步num是21,所以这一步 num变成了22, 同时输出22

8. 然后obj.fn(10),这时候fn的this为obj,obj.num默认值是20, this.num += n相当于 obj.num += 10

9. 和第七步一样， num + 1 输出 23

10. console.log(num, obj.num)相当于 console.log(window.num, obj.num)，从上面几步可知， window.num = 65, obj.num = 30。

## 21. 数据类型转换问题

```js
console.log([] + [])
console.log({} + [])
console.log([] == ![])
console.log(true + false)
```

1. 第一行代码

```js
// 输出 "" 空字符串
console.log([] + [])
```

这行代码输出的是空字符串""， 包装类型在运算的时候，会先调用valueOf方法，如果valueOf返回的还是包装类型，那么再调用toString方法

```js
// 还是 数组
const val = [].valueOf()
// 数组 toString 默认会将数组各项使用逗号 "," 隔开, 比如 [1,2,3].toSting 变成了"1,2,3",空数组 toString 就是空字符串
const val1 = val.toString() // val1 是空字符串
```

2. 第二行代码

```js
// 输出 "[object Object]"
console.log({} + [])
```

和第一题道理一样，对象 {}隐氏转换成了[object Object],然后与""相加

3. 第三行代码

对于===, 会严格比较两者的值，但是对于==就不一样了

  1. 比如 null == undefined

  2. 如果非number与number比较，会将其转换为number

  3. 如果比较的双方中由一方是boolean,那么会先将boolean转换为number

```js
// 这个输出 false
console.log(![])
// 套用上面第三条 将 false 转换为 数值
// 这个输出 0
console.log(Number(false))
// 包装类型与 基本类型 == 先将包装类型通过 valueOf toString 转换为基本类型 
// 输出 ""
console.log([].toString())
// 套用第2条， 将空字符串转换为数值、
// 输出 0
console.log(Number(""))
// 所以
console.log(0 == 0)
```

4. 两个基本类型相加，如果其中一方是字符，则将其他的转换为字符相加，否则将类型转换为Number,然后相加, Number(true) 是1, Number(false)是0, 所以结果是 1


## 22. 有一个函数，参数是一个函数，返回值也是一个函数，返回的函数功能和入参的函数相似，但这个函数只能执行3次，再次执行无效，如何实现

这个题目是考察闭包的使用

```js
function sayHi() {
    console.log('hi')
}

function threeTimes(fn) {
    let times = 0
    return () => {
        if (times++ < 3) {
            fn()
        }
    }
}

const newFn = threeTimes(sayHi)
newFn()
newFn()
newFn()
newFn()
newFn() // 后面两次执行都无任何反应
```

## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 