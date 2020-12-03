<!--
 * @Author: Li Zhiliang
 * @Date: 2020-12-02 16:00:52
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-12-02 16:13:28
 * @FilePath: /FE-Interview.git/project/adapter.md
-->

# 移动端适配方案具体实现以及对比

## 常见的移动端适配方案

- media queries

- flex 布局

- rem + viewport

- vh vw

- 百分比

### 一、Meida Queries(响应式)

meida queries 的方式可以说是我早期采用的布局方式，它主要是通过查询设备的宽度来执行不同的 css 代码，最终达到界面的配置。

核心语法：

```css
@media only screen and (max-width: 374px) {
  /* iphone5 或者更小的尺寸，以 iphone5 的宽度（320px）比例设置样式*/
}
@media only screen and (min-width: 375px) and (max-width: 413px) {
  /* iphone6/7/8 和 iphone x */
}
@media only screen and (min-width: 414px) {
  /* iphone6p 或者更大的尺寸，以 iphone6p 的宽度（414px）比例设置样式 */
}
```

- 优点

    - media query 可以做到设备像素比的判断，方法简单，成本低，特别是针对移动端和 PC 端维护同一套代码的时候。目前像 Bootstrap 等框架使用这种方式布局

    - 图片便于修改，只需修改 css 文件

    - 调整屏幕宽度的时候不用刷新页面即可响应式展示

- 缺点
    
    - 代码量比较大，维护不方便
    
    - 为了兼顾大屏幕或高清设备，会造成其他设备资源浪费，特别是加载图片资源
    
    - 为了兼顾移动端和 PC 端各自响应式的展示效果，难免会损失各自特有的交互方式

### 二、Flex 弹性布局

viewport 是固定的：

```html
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
```

高度定死，宽度自适应，元素都采用 px 做单位。

随着屏幕宽度变化，页面也会跟着变化，效果就和 PC 页面的流体布局差不多，在哪个宽度需要调整的时候使用响应式布局调调就行（比如网易新闻），这样就实现了『适配』。

### 三、rem+viewport 缩放

**实现原理：**

根据 rem 将页面放大 dpr 倍, 然后 viewport 设置为 1/dpr.

- 如 iphone6 plus 的 dpr 为 3, 则页面整体放大 3 倍, 1px(css 单位)在 plus 下默认为 3px(物理像素)

- 然后 viewport 设置为 1/3, 这样页面整体缩回原始大小. 从而实现高清。

这样整个网页在设备内显示时的页面宽度就会等于设备逻辑像素大小，也就是 device-width。这个 device-width 的计算公式为：

设备的物理分辨率/(devicePixelRatio * scale)，在 scale 为 1 的情况下，device-width = 设备的物理分辨率/devicePixelRatio。

### 四、rem 实现

rem是相对长度单位，rem方案中的样式设计为相对于根元素font-size计算值的倍数。根据屏幕宽度设置html标签的font-size，在布局时使用 rem 单位布局，达到自适应的目的。

viewport 是固定的：

```html
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">。
```

通过以下代码来控制 rem 基准值(设计稿以 720px 宽度量取实际尺寸)

```js
!(function (d) {
  var c = d.document;
  var a = c.documentElement;
  var b = d.devicePixelRatio;
  var f;
  function e() {
    var h = a.getBoundingClientRect().width,
      g;
    if (b === 1) {
      h = 720;
    }
    if (h > 720) h = 720; //设置基准值的极限值
    g = h / 7.2;
    a.style.fontSize = g + "px";
  }
  if (b > 2) {
    b = 3;
  } else {
    if (b > 1) {
      b = 2;
    } else {
      b = 1;
    }
  }
  a.setAttribute("data-dpr", b);
  d.addEventListener(
    "resize",
    function () {
      clearTimeout(f);
      f = setTimeout(e, 200);
    },
    false
  );
  e();
})(window);
```

- 优点：

    - 兼容性好，页面不会因为伸缩发生变形，自适应效果更佳。

- 缺点：

    - 不是纯 css 移动适配方案，需要在头部内嵌一段 js脚本监听分辨率的变化来动态改变根元素的字体大小，css样式和 js 代码有一定耦合性，并且必须将改变font-size的代码放在 css 样式之前。

    - 小数像素问题，浏览器渲染最小的单位是像素，元素根据屏幕宽度自适应，通过 rem 计算后可能会出现小数像素，浏览器会对这部分小数四舍五入，按照整数渲染，有可能没那么准确。


### 五、纯 vw 方案

视口是浏览器中用于呈现网页的区域。

- vw : 1vw 等于 视口宽度 的 1%

- vh : 1vh 等于 视口高度 的 **1% **

- vmin : 选取 vw 和 vh 中 最小 的那个

- vmax : 选取 vw 和 vh 中 最大 的那个

虽然 vw 能更优雅的适配，但是还是有点小问题，就是宽，高没法限制。

```js
$base_vw = 375;
@function vw ($px) {
    return ($px/$base_vw) * 100vw
};
```

- 优点
    
    - 纯 css 移动端适配方案，不存在脚本依赖问题。
    
    - 相对于 rem 以根元素字体大小的倍数定义元素大小，逻辑清晰简单。

- 缺点：

    - 存在一些兼容性问题，有些浏览器不支持

### 六、vw + rem 方案

```css
// scss 语法
// 设置html根元素的大小 750px->75 640px->64
// 将屏幕分成10份，每份作为根元素的大小。
$vw_fontsize: 75
@function rem($px) {
    // 例如：一个div的宽度为100px，那么它对应的rem单位就是（100/根元素的大小）* 1rem
    @return ($px / $vw_fontsize) * 1rem;
}
$base_design: 750
html {
    // rem与vw相关联
    font-size: ($vw_fontsize / ($base_design / 2)) * 100vw;
    // 同时，通过Media Queries 限制根元素最大最小值
    @media screen and (max-width: 320px) {
        font-size: 64px;
    }
    @media screen and (min-width: 540px) {
        font-size: 108px;
    }
}

// body 也增加最大最小宽度限制，避免默认100%宽度的 block 元素跟随 body 而过大过小
body {
    max-width: 540px;
    min-width: 320px;
}
```

### 七、百分比

使用百分比%定义宽度，高度用px固定，根据可视区域实时尺寸进行调整，尽可能适应各种分辨率，通常使用max-width/min-width控制尺寸范围过大或者过小。

- 优点：

    - 原理简单，不存在兼容性问题

- 缺点：

    - 如果屏幕尺度跨度太大，相对设计稿过大或者过小的屏幕不能正常显示，在大屏手机或横竖屏切换场景下可能会导致页面元素被拉伸变形，字体大小无法随屏幕大小发生变化。

    - 设置盒模型的不同属性时，其百分比设置的参考元素不唯一，容易使布局问题变得复杂。

###