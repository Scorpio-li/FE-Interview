# Knowledge

## 1、BFC相关

> BFC(Block formatting context)直译为"块级格式化上下文"。

在讲BFC之前得先说下display的属性值，只有它符合成为条件才资格触发BFC机制

> 不是所有的元素模式都能产生BFC，w3c 规范： display 属性为 block, list-item, table 的元素，会产生BFC.

### 什么情况下可以让元素产生BFC

要给这些元素添加如下属性就可以触发BFC。

- float属性不为none

- position为absolute或fixed

- display为inline-block, table-cell, table-caption, flex, inline-flex

- overflow不为visible。

### BFC元素所具有的特性

- 在BFC中，盒子从顶端开始垂直地一个接一个地排列

- 盒子垂直方向的距离由margin决定。属于同一个BFC的两个相邻盒子的margin会发生重叠

- 在BFC中，每一个盒子的左外边缘（margin-left）会触碰到容器的左边缘(border-left)（对于从右到左的格式来说，则触碰到右边缘）。
    
    - BFC的区域不会与浮动盒子产生交集，而是紧贴浮动边缘。
    
    - 计算BFC的高度时，自然也会检测浮动或者定位的盒子高度

### BFC的主要用途

**(1) 清除元素内部浮动**

只要把父元素设为BFC就可以清理子元素的浮动了，最常见的用法就是在父元素上设置overflow: hidden样式

**(2) 解决外边距合并(塌陷)问题**

盒子垂直方向的距离由margin决定。属于同一个BFC的两个相邻盒子的margin会发生重叠

属于同一个sBFC的两个相邻盒子的margin会发生重叠，那么我们创建不属于同一个BFC，就不会发生margin重叠了。

**(3) 制作右侧自适应的盒子问题**

普通流体元素BFC后，为了和浮动元素不产生任何交集，顺着浮动边缘形成自己的封闭上下文

> 总结：BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。包括浮动，和外边距合并等等，因此，有了这个特性，我们布局的时候就不会出现意外情况了。

## 2、IFC相关

> IFC（inline Formatting Context）叫做“行级格式化上下”相对BFC比较简单且问的也不是很多，这里大该做个了解

布局规则如下：

- 内部的盒子会在水平方向，一个个地放置(默认就是IFC)；

- IFC的高度，由里面最高盒子的高度决定(里面的内容会撑开父盒子）；

- 当一行不够放置的时候会自动切换到下一行；


## 3、哪些属性开启了性能加速

硬件加速：就是将浏览器的渲染过程交给GPU(Graphics Processing Unit)处理，而不是使用自带的比较慢的渲染器。这样就可以使得**animation**与**transition**更加顺畅

我们可以在浏览器中用CSS开启硬件加速，使GPU发挥功能，从而提升性能

所谓GPU，就是图形处理器的缩写，相当于PC中的显卡。手机中的GPU也是为了对图形、图像处理而存在的，所谓强制渲染，就是hwa（hardware acceleration硬件加载）的一种，其存在的意义就是为了分担cpu的负担，其原理是通过GPU对软件图形的处理来减轻CPU的负担。从而使应用软件能够以更快的速度被处理，以达到提速的目的。

### 硬件加速的原理

浏览器接收到页面文档后，会将文档中的标记语言解析为DOM树。DOM树和CSS结合后形成浏览器构建页面的渲染树。渲染树中包含大量的渲染元素，每个渲染元素会被分到一个图层中，每个图层又会被加载到GPU形成渲染纹理，而图层在GPU中transform是不会触发repaint的，最终这些使用transform的图层都会由独立的合成器进程进行处理, CSS transform会**创建了一个新的复合图层，可以被GPU直接用来执行transform操作**。

### 浏览器什么时候会创建一个独立的复合图层呢？事实上一般是在以下几种情况下：

- 3D或者CSS transform

- video 和 canvas 标签

- css filters(滤镜效果)

- 元素覆盖时，比如使用了z-index属性

### 为什么硬件加速会使页面流畅

因为transform属性不会触发浏览器的repaint（重绘），而绝对定位absolute中的left和top则会一直触发repaint（重绘）。

### 为什么transform没有触发repaint呢？

简而言之，transform动画由GPU控制，支持硬件加载，并不需要软件方面的渲染。并不是所有的CSS属性都能触发GPU的硬件加载，事实上只有少数的属性可以，比如transform、opacity、filter

### 如何用CSS开启硬件加速

当浏览器检测到页面中某个DOM元素应用了某些CSS规则时就会开启，最显著的特征的元素是3D变化。

```css
.cube{
    translate3d(250px,250px,250px);
    rotate3d(250px,250px,250px,-120deg)
    scale3d(0.5,0.5,0.5);
}
```

> 可能在一些情况下，我们并不需要对元素应用3D变幻的效果，那怎么办呢?这时候我们可以使用“欺骗”浏览器来开启硬件加速。虽然我们可能不想对元素应用3D变幻，可我们一样可以开启3D引擎。例如我们可以用**transform:translateZ(0)**;来开启硬件加速





##
##
##
##
##
##
##
##
##
##