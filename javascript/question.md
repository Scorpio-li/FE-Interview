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
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 
## 16. 