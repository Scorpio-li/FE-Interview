<!--
 * @Author: Li Zhiliang
 * @Date: 2020-11-18 11:14:03
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-12-15 22:57:00
 * @FilePath: /FE-Interview.git/vue/knowledge.md
-->
# Knowledge

## 1. Vue的优点及缺点

首先Vue最核心的两个特点，**响应式**和**组件化**。

- **响应式**：这也就是vue.js最大的优点，通过MVVM思想实现数据的双向绑定，通过虚拟DOM让我们可以用数据来操作DOM，而不必去操作真实的DOM，提升了性能。且让开发者有更多的时间去思考业务逻辑。

- **组件化**：把一个单页应用中的各个模块拆分到一个个组件当中，或者把一些公共的部分抽离出来做成一个可复用的组件。所以组件化带来的好处就是，提高了开发效率，方便重复使用，使项目的可维护性更强。

**虚拟DOM**，当然，这个不是vue中独有的。

**缺点**：

- 基于对象配置文件的写法，也就是options写法，开发时不利于对一个属性的查找。

- 另外一些缺点，在小项目中感觉不太出什么，vuex的魔法字符串，对ts的支持。

- 兼容性上存在一些问题。不利于seo。

- 导航不可用，如果一定要导航需要自行实现前进、后退。（由于是单页面不能用浏览器的前进后退功能，所以需要自己建立堆栈管理）。初次加载时耗时多。

## 2. Vue中hash模式和history模式的区别

- 最明显的是在显示上，hash模式的URL中会夹杂着#号，而history没有。

- Vue底层对它们的实现方式不同。hash模式是依靠onhashchange事件(监听location.hash的改变)，而history模式是主要是依靠的HTML5 history中新增的两个方法，pushState()可以改变url地址且不会发送请求，replaceState()可以读取历史记录栈,还可以对浏览器记录进行修改。

- 当真正需要通过URL向后端发送HTTP请求的时候，比如常见的用户手动输入URL后回车，或者是刷新(重启)浏览器，这时候history模式需要后端的支持。因为history模式下，前端的URL必须和实际向后端发送请求的URL一致，例如有一个URL是带有路径path的(例如www.lindaidai.wang/blogs/id)，如果后端没有对这个路径做处理的话，就会返回404错误。所以需要后端增加一个覆盖所有情况的候选资源，一般会配合前端给出的一个404页面。

**hash**

```js
window.onhashchange = function(event){
  // location.hash获取到的是包括#号的，如"#heading-3"
  // 所以可以截取一下
  let hash = location.hash.slice(1);
}
```

## 3. Vue的响应式原理

## 4. Vue 中父组件可以监听到子组件的生命周期吗？

### 一、使用 on 和 emit

```js
// Parent.vue
<Child @mounted="doSomething"/>
// Child.vue
mounted() {
  this.$emit("mounted");
}
```

### 二、使用 hook 钩子函数

```js
//  Parent.vue
<Child @hook:mounted="doSomething" ></Child>

doSomething() {
   console.log('父组件监听到 mounted 钩子函数 ...');
},
//  Child.vue
mounted(){
   console.log('子组件触发 mounted 钩子函数 ...');
},
// 以上输出顺序为：
// 子组件触发 mounted 钩子函数 ...
// 父组件监听到 mounted 钩子函数 ...
```

## 5. 怎么给 Vue 定义全局方法

### 一、将方法挂载到 Vue.prototype 上面

> 缺点：调用这个方法的时候没有提示

```js
// global.js
const RandomString = (encode = 36, number = -8) => {
  return Math.random() // 生成随机数字, eg: 0.123456
    .toString(encode) // 转化成36进制 : "0.4fzyo82mvyr"
    .slice(number);
},
export default {
	RandomString,
  ...
}
```

```js
// 在项目入口的main.js里配置
import Vue from "vue";
import global from "@/global";
Object.keys(global).forEach((key) => {
  Vue.prototype["$global" + key] = global[key];
});
```

```js
// 挂载之后，在需要引用全局变量的模块处(App.vue)，不需再导入全局变量模块，而是直接用this就可以引用了，如下:
export default {
  mounted() {
    this.$globalRandomString();
  },
};
```

### 二、利用全局混入mixin

> 优点：因为mixin里面的methods会和创建的每个单文件组件合并。这样做的优点是调用这个方法的时候有提示

```js
// mixin.js
import moment from 'moment'
const mixin = {
  methods: {
    minRandomString(encode = 36, number = -8) {
      return Math.random() // 生成随机数字, eg: 0.123456
        .toString(encode) // 转化成36进制 : "0.4fzyo82mvyr"
        .slice(number);
    },
    ...
  }
}
export default mixin
```

```js
// 在项目入口的main.js里配置
import Vue from 'vue'
import mixin from '@/mixin'
Vue.mixin(mixin)
```

```js
export default {
 mounted() {
   this.minRandomString()
 }
}
```

### 三、使用Plugin方式

> Vue.use的实现没有挂载的功能，只是触发了插件的install方法，本质还是使用了Vue.prototype。

```js
// plugin.js
function randomString(encode = 36, number = -8) {
  return Math.random() // 生成随机数字, eg: 0.123456
    .toString(encode) // 转化成36进制 : "0.4fzyo82mvyr"
    .slice(number);
}
const plugin = {
  // install 是默认的方法。
  // 当外界在 use 这个组件或函数的时候，就会调用本身的 install 方法，同时传一个 Vue 这个类的参数。
  install: function(Vue){
    Vue.prototype.$pluginRandomString = randomString
    ...
  },
}
export default plugin
```

```js
// 在项目入口的main.js里配置
import Vue from 'vue'
import plugin from '@/plugin'
Vue.use(plugin)
```

```js
export default {
 mounted() {
   this.$pluginRandomString()
 }
}
```

### 四、任意 vue 文件中写全局函数

```js
// 创建全局方法
this.$root.$on("test", function () {
  console.log("test");
});
// 销毁全局方法
this.$root.$off("test");
// 调用全局方法
this.$root.$emit("test");
```

## 6.说一下 vm.$set 原理

### vm.$set()解决了什么问题

在 Vue.js 里面只有 data 中已经存在的属性才会被 Observe 为响应式数据，如果你是新增的属性是不会成为响应式数据，因此 Vue 提供了一个 api(vm.$set)来解决这个问题。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Vue Demo</title>
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
  <body>
    <div id="app">
      {{user.name}} {{user.age}}
      <button @click="addUserAgeField">增加一个年纪字段</button>
    </div>
    <script>
      const app = new Vue({
        el: "#app",
        data: {
          user: {
            name: "test",
          },
        },
        mounted() {},
        methods: {
          addUserAgeField() {
            // this.user.age = 20 这样是不起作用, 不会被Observer
            this.$set(this.user, "age", 20); // 应该使用
          },
        },
      });
    </script>
  </body>
</html>
```

### 原理

vm.$set()在 new Vue()时候就被注入到 Vue 的原型上。

### 源码位置: vue/src/core/instance/index.js

```js
import { initMixin } from "./init";
import { stateMixin } from "./state";
import { renderMixin } from "./render";
import { eventsMixin } from "./events";
import { lifecycleMixin } from "./lifecycle";
import { warn } from "../util/index";

function Vue(options) {
  if (process.env.NODE_ENV !== "production" && !(this instanceof Vue)) {
    warn("Vue is a constructor and should be called with the `new` keyword");
  }
  this._init(options);
}

initMixin(Vue);
// 给原型绑定代理属性$props, $data
// 给Vue原型绑定三个实例方法: vm.$watch，vm.$set，vm.$delete
stateMixin(Vue);
// 给Vue原型绑定事件相关的实例方法: vm.$on, vm.$once ,vm.$off , vm.$emit
eventsMixin(Vue);
// 给Vue原型绑定生命周期相关的实例方法: vm.$forceUpdate, vm.destroy, 以及私有方法_update
lifecycleMixin(Vue);
// 给Vue原型绑定生命周期相关的实例方法: vm.$nextTick, 以及私有方法_render, 以及一堆工具方法
renderMixin(Vue);

export default Vue;
```

- stateMixin()

```js
Vue.prototype.$set = set;
Vue.prototype.$delete = del;
```

- set(): 源码位置: vue/src/core/observer/index.js

```js
export function set(target: Array<any> | Object, key: any, val: any): any {
  // 1.类型判断
  // 如果 set 函数的第一个参数是 undefined 或 null 或者是原始类型值，那么在非生产环境下会打印警告信息
  // 这个api本来就是给对象与数组使用的
  if (
    process.env.NODE_ENV !== "production" &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(
      `Cannot set reactive property on undefined, null, or primitive value: ${(target: any)}`
    );
  }
  // 2.数组处理
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    // 类似$vm.set(vm.$data.arr, 0, 3)
    // 修改数组的长度, 避免索引>数组长度导致splcie()执行有误
    //如果不设置length，splice时，超过原本数量的index则不会添加空白项
    target.length = Math.max(target.length, key);
    // 利用数组的splice变异方法触发响应式, 这个前面讲过
    target.splice(key, 1, val);
    return val;
  }
  //3.对象，且key不是原型上的属性处理
  // target为对象, key在target或者target.prototype上。
  // 同时必须不能在 Object.prototype 上
  // 直接修改即可, 有兴趣可以看issue: https://github.com/vuejs/vue/issues/6845
  if (key in target && !(key in Object.prototype)) {
    target[key] = val;
    return val;
  }
  // 以上都不成立, 即开始给target创建一个全新的属性
  // 获取Observer实例
  const ob = (target: any).__ob__;
  // Vue 实例对象拥有 _isVue 属性, 即不允许给Vue 实例对象添加属性
  // 也不允许Vue.set/$set 函数为根数据对象(vm.$data)添加属性
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== "production" &&
      warn(
        "Avoid adding reactive properties to a Vue instance or its root $data " +
          "at runtime - declare it upfront in the data option."
      );
    return val;
  }
  //5.target是非响应式数据时
  // target本身就不是响应式数据, 直接赋值
  if (!ob) {
    target[key] = val;
    return val;
  }
  //6.target对象是响应式数据时
  //定义响应式对象
  defineReactive(ob.value, key, val);
  //watcher执行
  ob.dep.notify();
  return val;
}
```

- 工具函数

```js
// 判断给定变量是否是未定义，当变量值为 null时，也会认为其是未定义
export function isUndef(v: any): boolean %checks {
  return v === undefined || v === null;
}

// 判断给定变量是否是原始类型值
export function isPrimitive(value: any): boolean %checks {
  return (
    typeof value === "string" ||
    typeof value === "number" ||
    // $flow-disable-line
    typeof value === "symbol" ||
    typeof value === "boolean"
  );
}

// 判断给定变量的值是否是有效的数组索引
export function isValidArrayIndex(val: any): boolean {
  const n = parseFloat(String(val));
  return n >= 0 && Math.floor(n) === n && isFinite(val);
}
```

- 在初始化 Vue 的过程中有

```js
export function initState(vm: Component) {
  vm._watchers = [];
  const opts = vm.$options;
  if (opts.props) initProps(vm, opts.props);
  if (opts.methods) initMethods(vm, opts.methods);
  if (opts.data) {
    //opts.data为对象属性
    initData(vm);
  } else {
    observe((vm._data = {}), true /* asRootData */);
  }
  if (opts.computed) initComputed(vm, opts.computed);
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch);
  }
}
```

- initData(vm)

```js
function initData(vm: Component) {
  let data = vm.$options.data;
  data = vm._data = typeof data === "function" ? getData(data, vm) : data || {};

  // 省略...

  // observe data
  observe(data, true /* asRootData */);
}
```

从源码可以看出 set 主要逻辑如下：

1. 类型判断

2. target 为数组：调用 splice 方法

3. target 为对象，且 key 不是原型上的属性处理：直接修改

4. target 不能是 Vue 实例，或者 Vue 实例的根数据对象，否则报错

5. target 是非响应式数据时，我们就按照普通对象添加属性的方式来处理

6. target 为响应数据，且key 为新增属性，我们 key 设置为响应式，并手动触发其属性值的更新

### 总结

**vm.$set(target、key、value)**

- 当 target 为数组时，直接调用数组方法 splice 实现；

- 如果目标是对象，会先判读属性是否存在、对象是否是响应式

- 最终如果要对属性进行响应式处理，则是通过调用 defineReactive 方法进行响应式处理

  - defineReactive 方法就是 Vue 在初始化对象时，给对象属性采用 Object.defineProperty 动态添加 getter 和 setter 的功能所调用的方法


## 7. vue的data为什么是一个方法

因为组件是用来复用的，且 JS 里对象是引用关系，如果组件中 data 是一个对象，那么这样作用域没有隔离，子组件中的 data 属性值会相互影响，如果组件中 data 选项是一个函数，那么每个实例可以维护一份被返回对象的独立的拷贝，组件实例之间的 data 属性值不会互相影响；而 new Vue 的实例，是不会被复用的，因此不存在引用对象的问题

一个组件被复用多次的话，也就会创建多个实例。本质上，这些实例用的都是同一个构造函数。如果data是对象的话，对象属于引用类型，会影响到所有的实例。所以为了保证组件不同的实例之间data不冲突，data必须是一个函数

## 8. watch 监听实现

vm 调用 $watch 后，首先调用 observe 函数 创建 Observer 实例观察数据，Observer 又创建 Dep , Dep 用来维护订阅者。然后创建 Watcher 实例提供 update函数。一旦数据变动，就层层执行回调函数

## 9. vue-router的hash跟history模式

### 9.1 hash模式

hash模式， 原本用来结合锚点控制页面视窗的位置，具有以下特点：

- 在hash模式下，所有的页面跳转都是客户端进行操作，因此对于页面拦截更加灵活；但每次url的改变不属于一次http请求，所以不利于SEO优化

- hash 模式是一种把前端路由的路径用井号 # 拼接在真实 URL 后面的模式。当井号 # 后面的路径发生变化时，浏览器并不会重新发起请求，而是会触发 hashchange 事件。

- 可以改变URL，但不会触发页面重新加载（hash的改变会记录在window.hisotry中）因此并不算是一次http请求，所以这种模式不利于SEO优化

- 只能修改#后面的部分，因此只能跳转与当前URL同文档的URL

- 只能通过字符串改变URL

- 通过window.onhashchange监听hash的改变，借此实现无刷新跳转的功能

### 9.2 history模式

history模式， 根据 Mozilla Develop Network 的介绍，调用 history.pushState() 相比于直接修改 hash，存在以下优势

- history API 是 H5 提供的新特性，允许开发者直接更改前端路由，即更新浏览器 URL 地址而不重新发起请求

- 新的URL可以是与当前URL同源的任意 URL，也可以与当前URL一样，但是这样会把重复的一次操作记录到栈中

- 通过参数stateObject可以添加任意类型的数据到记录中

- 可额外设置title属性供后续使用

- 通过pushState、replaceState实现无刷新跳转的功能

- 兼容性不如 hash，且需要服务端支持，否则一刷新页面就404了

- 在history模式下，借助history.pushState实现页面的无刷新跳转；这种方式URL的改变属于http请求，因此会重新请求服务器，这也使得我们必须在服务端配置好地址，否则服务端会返回404，为确保不出问题，最好在项目中配置404页面

## 10. vue3 Proxy跟vue2 defineProperty区别

### defineProperty缺点

- 无法检测对象属性的添加或移除，为此我们需要使用 Vue.set 和 Vue.delete 来保证响应系统的运行符合预期

- 无法监控到数组下标及数组长度的变化，当直接通过数组的下标给数组设置值或者改变数组长度时，不能实时响应

- 性能问题，当data中数据比较多且层级很深的时候，因为要遍历data中所有的数据并给其设置成响应式的，会导致性能下降

### 对比区别

- Object.defineProperty只能劫持对象的属性，对新增属性需要手动进行 Observe，而 Proxy 是直接代理对象

- 为什么 Proxy 可以解决以上的痛点呢？ 本质的原因在于 Proxy 是一个内置了拦截器的对象，所有的外部访问都得先经过这一层拦截。不管是先前就定义好的，还是新添加属性，访问时都会被拦截（proxy具体学习请看阮一峰老师的ES6教程 Proxy）


## 11. computed和watch区别

- computed： 是计算属性，依赖其它属性值，并且 computed 的值有缓存，只有它依赖的属性值发生改变，下一次获取 computed 的值时才会重新计算 computed的值

- watch： 更多的是「观察」的作用，类似于某些数据的监听回调 ，每当监听的数据变化时都会执行回调进行后续操作

- 相同： computed和watch都起到监听/依赖一个数据，并进行处理的作用

- 不同：它们其实都是vue对监听器的实现，只不过computed主要用于对同步数据的处理，watch则主要用于观测某个值的变化去完成一段开销较大的复杂业务逻辑。能用computed的时候优先用computed，避免了多个数据影响其中某个数据时多次调用watch的尴尬情况

## 12. 在哪个生命周期内调用异步请求

可以在钩子函数 created、beforeMount、mounted 中进行调用，因为在这三个钩子函数中，data 已经创建，可以将服务端端返回的数据进行赋值。但是本人推荐在 created 钩子函数中调用异步请求，因为在 created 钩子函数中调用异步请求有以下优点

- 能更快获取到服务端数据，减少页面 loading 时间

- ssr 不支持 beforeMount 、mounted 钩子函数，所以放在 created 中有助于一致性

## 13. 虚拟DOM

### 虚拟DOM有什么好处？

假设一次操作中有10次更新DOM的动作，虚拟DOM不会立即操作DOM，而是将这10次更新的diff内容保存到本地一个JS对象中，最终将这个JS对象一次性attch到DOM树上，再进行后续操作，避免大量无谓的计算量。所以，用JS对象模拟DOM节点的好处是，页面的更新可以先全部反映在虚拟DOM上，操作内存中的JS对象的速度显然要更快，等更新完成后，再将最终的JS对象映射成真实的DOM，交由浏览器去绘制

### 通过DIFF算法对比操作JS对象实现差量更新

- 没有旧的节点，则创建新的节点，并插入父节点。

- 如果没有新的节点，则摧毁旧的节点。

- 如果节点发生了变化，则用replaceChild改变节点信息

- 如果节点没有变化，则对比该节点的子节点进行判断，使用递归调用

### 虚拟DOM实际渲染规则

Vue和React通用流程：vue template/react jsx -> render函数 -> 生成VNode -> 当有变化时，新老VNode diff -> diff算法对比，并真正去更新真实DOM

## 14. keep-alive

keep-alive是Vue.js的一个内置组件。<keep-alive> 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。 当组件在 <keep-alive> 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。它提供了include与exclude两个属性，允许组件有条件地进行缓存

## 15. vuex

### 15.1 vuex核心：

- state：存储store的各种状态

- mutation： 改变store的状态只能通过mutation方法

- action： 异步操作方法

- module： 模块化

- getter： 相当于计算属性，过滤出来一些值

### 15.2 vuex使用

每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的状态 (state)。Vuex 和单纯的全局对象有以下两点不同：


Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。


你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用


##
##
##
##
##