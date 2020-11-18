<!--
 * @Author: Li Zhiliang
 * @Date: 2020-11-18 11:03:22
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-11-18 13:58:14
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



