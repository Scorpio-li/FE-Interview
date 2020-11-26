<!--
 * @Author: Li Zhiliang
 * @Date: 2020-11-18 11:03:22
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-11-25 17:00:41
 * @FilePath: /FE-Interview.git/javascript/knowledge.md
-->
# Knowledge

## 1. 事件传播的三个阶段是什么？

**捕获 > 目标 > 冒泡**

在捕获阶段，事件通过父元素向下传递到目标元素。 然后它到达目标元素，冒泡开始。

![事件传播过程](../static/img/event-propagation.jpg)

## 2. 所有对象都有原型吗？

**错误**

基础对象指原型链终点的对象。基础对象的原型是null。

> 除基础对象外，所有对象都有原型。 基础对象可以访问某些方法和属性，例如.toString。 这就是您可以使用内置JavaScript方法的原因！ 所有这些方法都可以在原型上找到。 虽然JavaScript无法直接在您的对象上找到它，但它会沿着原型链向下寻找并在那里找到它，这使您可以访问它。

## 3. JavaScript全局执行上下文为你创建了两个东西:全局对象和this关键字.

**对**

> 基本执行上下文是全局执行上下文:它是代码中随处可访问的内容。

## 4. Javascript中的假值

- undefined

- null

- NaN

- 0

- '' (empty string)

- false

## 5. `setInterval`方法的返回值什么?

- 它返回一个唯一的id。 此id可用于使用clearInterval()函数清除该定时器。

## 6. 如何解决a标点击后hover事件失效的问题?

改变a标签css属性的排列顺序

只需要记住LoVe HAte原则就可以了：

```
link→visited→hover→active
```

比如下面错误的代码顺序：

```css
a:hover{
  color: green;
  text-decoration: none;
}
a:visited{ /* visited在hover后面，这样的话hover事件就失效了 */
  color: red;
  text-decoration: none;
}
```
正确的做法是将两个事件的位置调整一下。

注意⚠️各个阶段的含义：

> a:link：未访问时的样式，一般省略成a a:visited：已经访问后的样式 a:hover：鼠标移上去时的样式 a:active：鼠标按下时的样式

## 7. 点击一个input依次触发的事件

```js
const text = document.getElementById('text');
text.onclick = function (e) {
  console.log('onclick')
}
text.onfocus = function (e) {
  console.log('onfocus')
}
text.onmousedown = function (e) {
  console.log('onmousedown')
}
text.onmouseenter = function (e) {
  console.log('onmouseenter')
}
```

Answer

```
'onmouseenter'
'onmousedown'
'onfocus'
'onclick'
```

## 8. null和undefined的区别

- null表示一个"无"的对象，也就是该处不应该有值；而undefined表示未定义。

- 在转换为数字时结果不同，Number(null)为0，而undefined为NaN。

**null：**

- 作为函数的参数，表示该函数的参数不是对象

- 作为对象原型链的终点

**undefined:**

- 变量被声明了，但没有赋值时，就等于undefined

- 调用函数时，应该提供的参数没有提供，该参数等于undefined

- 对象没有赋值属性，该属性的值为undefined

- 函数没有返回值时，默认返回undefined

## 9. docoment,window,html,body的层级关系

```js
window > document > html > body
```

**层级关系**

- window是BOM的核心对象，它一方面用来获取或设置浏览器的属性和行为，另一方面作为一个全局对象。

- document对象是一个跟文档相关的对象，拥有一些操作文档内容的功能。但是地位没有window高。

- html元素对象和document元素对象是属于html文档的DOM对象，可以认为就是html源代码中那些标签所化成的对象。他们跟div、select什么对象没有根本区别。

## 10. addEventListener函数的第三个参数

第三个参数涉及到冒泡和捕获，是true时为捕获，是false则为冒泡。

或者是一个对象{passive: true}，针对的是Safari浏览器，禁止/开启使用滚动的时候要用到。

## 11. 有写过原生的自定义事件吗

- 使用Event

- 使用customEvent （可以传参数）

- 使用document.createEvent('CustomEvent')和initEvent()

**创建自定义事件**

1. 使用Event

```js
let myEvent = new Event('event_name');
```

2. 使用customEvent （可以传参数）

```js
let myEvent = new CustomEvent('event_name', {
	detail: {
		// 将需要传递的参数放到这里
		// 可以在监听的回调函数中获取到：event.detail
	}
})
```

3. 使用document.createEvent('CustomEvent')和initEvent()

```js
let myEvent = document.createEvent('CustomEvent');// 注意这里是为'CustomEvent'
myEvent.initEvent(
	// 1. event_name: 事件名称
	// 2. canBubble: 是否冒泡
	// 3. cancelable: 是否可以取消默认行为
)
```

- createEvent：创建一个事件

- initEvent：初始化一个事件

**事件的监听**

自定义事件的监听其实和普通事件的一样，使用addEventListener来监听：

```js
button.addEventListener('event_name', function (e) {})
```

**事件的触发**

触发自定义事件使用dispatchEvent(myEvent)。

注意⚠️，这里的参数是要自定义事件的对象(也就是myEvent)，而不是自定义事件的名称('myEvent')

```js
// 1.
// let myEvent = new Event('myEvent');
// 2.
// let myEvent = new CustomEvent('myEvent', {
//   detail: {
//     name: 'lindaidai'
//   }
// })
// 3.
let myEvent = document.createEvent('CustomEvent');
myEvent.initEvent('myEvent', true, true)

let btn = document.getElementsByTagName('button')[0]
btn.addEventListener('myEvent', function (e) {
  console.log(e)
  console.log(e.detail)
})
setTimeout(() => {
  btn.dispatchEvent(myEvent)
}, 2000)
```

## 12. 冒泡和捕获的具体过程

冒泡指的是：当给某个目标元素绑定了事件之后，这个事件会依次在它的父级元素中被触发(当然前提是这个父级元素也有这个同名称的事件，比如子元素和父元素都绑定了click事件就触发父元素的click)。

捕获则是从上层向下层传递，与冒泡相反。

（非常好记，你就想想水底有一个泡泡从下面往上传的，所以是冒泡）

例子：

```html
<!-- 会依次执行 button li ul -->
<ul onclick="alert('ul')">
  <li onclick="alert('li')">
    <button onclick="alert('button')">点击</button>
  </li>
</ul>
<script>
  window.addEventListener('click', function (e) {
    alert('window')
  })
  document.addEventListener('click', function (e) {
    alert('document')
  })
</script>
```

冒泡结果：button > li > ul > document > window

捕获结果：window > document > ul > li > button


## 13. 所有的事件都有冒泡吗？

并不是所有的事件都有冒泡的，例如以下事件就没有：

- onblur

- onfocus

- onmouseenter

- onmouseleave

## 14. 描述下原型链

## 15. typeof和instanceof的区别

- typeof表示是对某个变量类型的检测，基本数据类型除了null都能正常的显示为对应的类型，引用类型除了函数会显示为'function'，其它都显示为object。

- 而instanceof它主要是用于检测**某个构造函数的原型对象在不在某个对象的原型链上**。

## 16. typeof为什么对null错误的显示

这只是 JS 存在的一个悠久 Bug。

在 JS 的最初版本中使用的是 32 位系统，为了性能考虑使用低位存储变量的类型信息，000 开头代表是对象然而 null 表示为全零，所以将它错误的判断为 object 。

## 17. 一句话描述一下this

对于函数而言指向最后调用函数的那个对象，是函数运行时内部自动生成的一个内部对象，只能在函数内部使用；对于全局来说，this指向window。

## 18.  apply/call/bind的相同和不同

## 19. 介绍一下虚拟DOM

虚拟DOM本质就是用一个原生的JavaScript对象去描述一个DOM节点。是对真实DOM的一层抽象。

由于在浏览器中操作DOM是很昂贵的。频繁的操作DOM，会产生一定的性能问题，因此我们需要这一层抽象，在patch过程中尽可能地一次性将差异更新到DOM中，这样保证了DOM不会出现性能很差的情况。

另外还有很重要的一点，也是它的设计初衷，为了更好的跨平tai，比如Node.js就没有DOM,如果想实现SSR(服务端渲染),那么一个方式就是借助Virtual DOM,因为Virtual DOM本身是JavaScript对象。

Vue2.x中的虚拟DOM主要是借鉴了snabbdom.js，Vue3中借鉴inferno.js算法进行优化。

## 20. CommonJS和ES6模块的区别

- CommonJS模块是运行时加载，ES6 Modules是编译时输出接口

- CommonJS输出是值的拷贝；ES6 Modules输出的是值的引用，被输出模块的内部的改变会影响引用的改变

- CommonJs导入的模块路径可以是一个表达式，因为它使用的是require()方法；而ES6 Modules只能是字符串

- CommonJS this指向当前模块，ES6 Modules this指向undefined

- 且ES6 Modules中没有这些顶层变量：arguments、require、module、exports、__filename、__dirname

关于第一个差异，是因为CommonJS 加载的是一个对象（即module.exports属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。

## 21.