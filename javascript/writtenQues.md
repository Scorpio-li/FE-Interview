<!--
 * @Author: Li Zhiliang
 * @Date: 2021-04-01 16:00:45
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2021-04-06 22:38:54
 * @FilePath: /FE-Interview.git/javascript/writtenQues.md
-->
# 笔试题汇总

## 第一题：实现无限滚动

```html
<!--
 * @Author: Li Zhiliang
 * @Date: 2021-04-01 16:04:57
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2021-04-01 16:11:44
 * @FilePath: /test-demo/infinite-scroll.html
-->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>实现无限滚动</title>
    <style>
        .container {
            width: 300px;
            height: 500px;
            margin: 0 auto;
            border: 1px solid #efeff4;
            background: #ffffff;
            overflow: auto;
        }
        
        .card {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 150px;
            background: #25bb9b;
            margin: 10px;
            color: #ffffff;
            border-radius: 8px;
        }
    </style>
</head>

<body>
    <p>要求补全代码，当div滑动条离最下面距离小于100px时，则插入一个元素进去，实现无限滚动。</p>
    <div class="container">
        <div class="card">1</div>
        <div class="card">2</div>
        <div class="card">3</div>
        <div class="card">4</div>
        <div class="card">5</div>
        <div class="card">6</div>
        <div class="card">7</div>
        <div class="card">8</div>
        <div class="card">9</div>
        <div class="card">10</div>
    </div>

</body>
<script>
    // const conrainer = document.getElementsByClassName('container')[0];
    // const scrollTop = container.scrollTop;
    // const contentHeight = container.scrollHeight;
    // const height = container.clientHeight;
    // const shouldTrigger = contentHeight - height - scrollTop <= distance

    new InfiniteScroll({
        el: document.querySelector('.container'),
        // 触发加载的距离底部的阈值，单位px
        distance: 100,
        call: (el, index) => {
            let dom = document.createElement('div')
            dom.className = 'card'
            dom.innerText = index
            el.appendChild(dom)
        }
    });

    function InfiniteScroll(param) {
        const that = this;
        const container = param.el;
        const distance = +param.distance || 0;
        if (!container) return;
        that.index = 0;
        container.onscroll = () => {
            _handleScroll(that, container, distance, param.call)
        }
        _handleScroll(that, container, distance, param.call)
        _handleScroll(that, container, distance, param.call)
    }


    function _handleScroll(that, container, distance, call) {
        const scrollTop = container.scrollTop;
        const contentHeight = container.scrollHeight;
        const height = container.clientHeight;
        // const shouldTrigger = (contentHeight - scrollTop) > (height - distance);
        console.log(scrollTop, contentHeight, height, distance)
            // const shouldTrigger = height - (scrollTop + contentHeight) <= distance
        const shouldTrigger = contentHeight - height - scrollTop <= distance
        if (!shouldTrigger) return;
        // TODO: 请调用call回调，注意提供正确的参数
        call(container, that.index++)
            // _handleScroll.call(this, container, distance);
    }
</script>

</html>
```

## 第二题：一个正整数n最少多少步才能变为0

```html
<!--
 * @Author: Li Zhiliang
 * @Date: 2021-04-01 16:17:57
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2021-04-01 16:42:43
 * @FilePath: /test-demo/calculate.html
-->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title></title>
</head>

<body>
    <p>一个正整数n最少多少步才能变为0</p>

    <ul>
        <li>n - 1
        </li>
        <li>如果n是2的倍数：n / 2
        </li>
        <li>如果n是3的倍数：n / 3</li>
    </ul>
    <input type="number" name="num" id="num">
    <p>总共使用：
        <span id="result">0</span> 步
    </p>
</body>
<script>
    const numDom = document.getElementById('num');
    const resDom = document.getElementById('result');
    numDom.onblur = () => {
        let num = numDom.value;

        let result = calculate(num, 0);
        console.log(result)
        resDom.innerHTML = result;
    };

    function calculate(num, index = 0) {
        let data = num;
        let d3 = data >= 3 ? data % 3 : false;
        let d2 = data >= 2 ? data % 2 : false;
        if (d3 === 0 || d3 === 1) {
            data = d3 === 0 ? data / 3 : (data - 1) / 3;
            index = d3 === 0 ? index + 1 : index + 2;
        } else if (d2 === 0) {
            data = data / 2;
            inde += 1;
        } else {
            data -= 1;
            index += 1;
        }
        console.log('data', data, index, d3, d2);
        if (data > 0) {
            return calculate(data, index);
        }
        return index;
    }
</script>

</html>
```

## 第五题：将中文数字字符串转换成数字

输入一：“一千三”

输出一：1300

输入二：“一千三百零一”

输出二：1301

输入三：“十二”

输出三：12

输入四：“一千三百二十一万一千三百二十一”

输出四：13211321