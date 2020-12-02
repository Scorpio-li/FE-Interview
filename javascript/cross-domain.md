<!--
 * @Author: Li Zhiliang
 * @Date: 2020-11-25 15:49:02
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-12-02 10:59:38
 * @FilePath: /FE-Interview.git/javascript/cross-domain.md
-->
## 跨域问题总结

### 1. JSONP的原理以及手写一个实现

**基本原理**：主要就是利用 script 标签的src属性没有跨域的限制，通过指向一个需要访问的地址，由服务端返回一个预先定义好的 Javascript 函数的调用，并且将服务器数据以该函数参数的形式传递过来，此方法需要前后端配合完成。JSONP 是一种非正式传输协议，允许用户传递一个callback给服务端，然后服务端返回数据时会将这个callback 参数作为函数名来包裹住 JSON 数据，这样客户端就可以随意定制自己的函数来自动处理返回数据了。当GET请求从后台页面返回时，可以返回一段JavaScript代码，这段代码会自动执行，可以用来负责调用后台页面中的一个callback函数。

执行过程：

- 前端定义一个解析函数(如: jsonpCallback = function (res) {})

- 通过params的形式包装script标签的请求参数，并且声明执行函数(如cb=jsonpCallback)

- 后端获取到前端声明的执行函数(jsonpCallback)，并以带上参数且调用执行函数的方式传递给前端

- 前端在script标签返回资源的时候就会去执行jsonpCallback并通过回调函数的方式拿到数据了。

缺点： 只能进行GET请求

优点： 兼容性好，在一些古老的浏览器中都可以运行

```js
<script>
    function JSONP({
        url,
        params = {},
        callbackKey = 'cb',
        callback
    }) {
        // 定义本地的唯一callbackId，若是没有的话则初始化为1
        JSONP.callbackId = JSONP.callbackId || 1;
        let callbackId = JSONP.callbackId;
        // 把要执行的回调加入到JSON对象中，避免污染window
        JSONP.callbacks = JSONP.callbacks || [];
        JSONP.callbacks[callbackId] = callback;
        // 把这个名称加入到参数中: 'cb=JSONP.callbacks[1]'
        params[callbackKey] = `JSONP.callbacks[${callbackId}]`;
        // 得到'id=1&cb=JSONP.callbacks[1]'
        const paramString = Object.keys(params).map(key => {
            return `${key}=${encodeURIComponent(params[key])}`
        }).join('&')
        // 创建 script 标签
        const script = document.createElement('script');
        script.setAttribute('src', `${url}?${paramString}`);
        document.body.appendChild(script);
        // id自增，保证唯一
        JSONP.callbackId++;

    }
    JSONP({
        url: 'http://localhost:8080/api/jsonps',
        params: {
            a: '2&b=3',
            b: '4'
        },
        callbackKey: 'cb',
        callback (res) {
            console.log(res)
        }
    })
    JSONP({
        url: 'http://localhost:8080/api/jsonp',
        params: {
            id: 1
        },
        callbackKey: 'cb',
        callback (res) {
            console.log(res)
        }
    })
</script>
```

### 2. 浏览器为什么要跨域？如果是因为安全的话那小程序或者其他的为什么没有跨域？

跨域的产生来源于现代浏览器所通用的同源策略，所谓同源策略，是指只有在地址的：

1. 协议名

2. 域名

3. 端口名

均一样的情况下，才允许访问相同的cookie、localStorage，以及访问页面的DOM或是发送Ajax请求。若在不同源的情况下访问，就称为跨域。

例如以下为同源：

```js
http://www.example.com:8080/index.html
http://www.example.com:8080/home.html
```

以下为跨域：

```js
http://www.example.com:8080/index.html
http://www3.example.com:8080/index.html
```

> 但是有两种情况：http默认的端口号为80，https默认端口号为443。

```js
http://www.example.com:80 === http://www.example.com
https://www.example.com:443 === https://www.example.com
```

#### 为什么浏览器会禁止跨域？

- 首先，跨域只存在于浏览器端，因为我们知道浏览器的形态是很开放的，所以我们需要对它有所限制。浏览器它为web提供了访问入口，并且访问的方式很简单，在地址栏输入要访问的地址或者点击某个链接就可以了，正是这种开放的形态，所以我们需要对它有所限制。

- 其次，同源策略主要是为了保证用户信息的安全，可分为两种：Ajax同源策略和DOM同源策略。

    - Ajax同源策略它主要做了这两种限制：
        
        1. 不同源页面不能获取cookie；
        
        2. 不同源页面不能发起Ajax请求。我认为它是防止CSRF攻击的一种方式吧。因为我们知道cookie这个东西它主要是为了解决浏览器与服务器会话状态的问题，它本质上是存储在浏览器或本地文件中一个小小的文本文件，那么它里面一般都会存储了用户的一些信息，包括隐私信息。如果没有Ajax同源策略，恶意网站只需要一段脚本就可以获取你的cookie，从而冒充你的身份去给其它网站发送恶意的请求。
    
    - DOM同源策略也一样，它限制了不同源页面不能获取DOM。例如一个假的网站利用iframe嵌套了一个银行网站mybank.com，并把宽高或者其它部分调整的和原银行网站一样，仅仅只是地址栏上的域名不同，若是用户没有注意的话就以为这个是个真的网站。如果这时候用户在里面输入了账号密码，如果没有同源策略，那么这个恶意网站就可以获取到银行网站中的DOM，也就能拿到用户的输入内容以此来达到窃取用户信息的攻击。

    同源策略它算是浏览器安全的第一层屏障吧，因为就像CSRF攻击，它只能限制不同源页面cookie的获取，但是攻击者还可能通过其它的方式来达到攻击效果。

    (注，上面提到的iframe限制DOM查询，案例如下）
    
    ```html
    // HTML
    <iframe name="yinhang" src="www.yinhang.com"></iframe>
    // JS
    // 由于没有同源策略的限制，钓鱼网站可以直接拿到别的网站的Dom
    const iframe = window.frames['yinhang']
    const node = iframe.document.getElementById('你输入账号密码的Input')
    console.log(`拿到了这个${node}，我还拿不到你刚刚输入的账号密码吗`)
    ```

- Ajax同源策略主要是使得不同源的页面不能获取cookie且不能发起Ajax请求，这样在一定层度上防止了CSRF攻击。

- DOM同源策略也一样，它限制了不同源页面不能获取DOM，这样可以防止一些恶意网站在自己的网站中利用iframe嵌入正gui的网站并迷惑用户，以此来达到窃取用户信息。

### 3. CORS跨域的原理

跨域资源共享(CORS)是一种机制，是W3C标准。它允许浏览器向跨源服务器，发出XMLHttpRequest或Fetch请求。并且整个CORS通信过程都是浏览器自动完成的，不需要用户参与。

而使用这种跨域资源共享的前提是，浏览器必须支持这个功能，并且服务器端也必须同意这种"跨域"请求。因此实现CORS的关键是服务器需要服务器。通常是有以下几个配置：

- Access-Control-Allow-Origin

- Access-Control-Allow-Methods

- Access-Control-Allow-Headers

- Access-Control-Allow-Credentials

- Access-Control-Max-Age

**Answer**

- 当我们发起跨域请求时，如果是非简单请求，浏览器会帮我们自动触发预检请求，也就是 OPTIONS 请求，用于确认目标资源是否支持跨域。如果是简单请求，则不会触发预检，直接发出正常请求。

- 浏览器会根据服务端响应的 header 自动处理剩余的请求，如果响应支持跨域，则继续发出正常请求，如果不支持，则在控制台显示错误。

- 浏览器先根据同源策略对前端页面和后台交互地址做匹配，若同源，则直接发送数据请求；若不同源，则发送跨域请求。

- 服务器收到浏览器跨域请求后，根据自身配置返回对应文件头。若未配置过任何允许跨域，则文件头里不包含 Access-Control-Allow-origin 字段，若配置过域名，则返回 Access-Control-Allow-origin + 对应配置规则里的域名的方式。

- 浏览器根据接受到的 响应头里的 Access-Control-Allow-origin 字段做匹配，若无该字段，说明不允许跨域，从而抛出一个错误；若有该字段，则对字段内容和当前域名做比对，如果同源，则说明可以跨域，浏览器接受该响应；若不同源，则说明该域名不可跨域，浏览器不接受该响应，并抛出一个错误。

### 4. 如何处理跨域

跨域处理的方案是在太多太多了，我们公司主要是通过nginx进行转发，前端发起请求的地址是和页面地址一致的所以不存在跨域，nginx将请求转发到正确的服务。页面地址：web.taobao.com，请求接口地址web.taobao.com/api.taobao.com/***

### 5. 表单可以跨域吗

表单提交是可以进行跨域的，不受浏览器的同源策略限制，估计是历史遗留原因，也有可能是表单提交的结果js是拿不到的，所以不限制问题也不大。但是存在一个问题，就是csrf攻击，具体不展开了，因为可以自动带上cookie造成攻击成功，而cookie的新属性SameSite就能用来限制这种情况

###
###
###
###
###
###
###
###
###
###
###
###
###
###
###
###
###
###