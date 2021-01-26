<!--
 * @Author: Li Zhiliang
 * @Date: 2021-01-25 16:33:58
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2021-01-25 16:45:24
 * @FilePath: /FE-Interview.git/javascript/virtualList.md
-->
# 虚拟列表优化

优化数据量大的业务场景长列表

## 1. 分片加载

```js
let index = 0;
let id = 0;

function load() {
    index += 20;
    if(index > total) {
        requestAnimationFram(() => {
            for(let i = 0; i < 20; i++) {
                // let fram
            }
        })
    }
}
```

## 2. 虚拟列表

- 可视区

- 真实列表

- startIndex

- endIndex

- 实现虚拟列表需要正确计算出可视区域的列表项，以及顶部和底部的留白高度，同时需要避免滚动时留白部分出现在可视区域内以及页面抖动，而通过给上下预留位置可以解决
