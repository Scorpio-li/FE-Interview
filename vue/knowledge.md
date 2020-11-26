<!--
 * @Author: Li Zhiliang
 * @Date: 2020-11-18 11:14:03
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-11-25 16:06:01
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