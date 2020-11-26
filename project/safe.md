<!--
 * @Author: Li Zhiliang
 * @Date: 2020-11-24 13:54:50
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-11-25 18:08:52
 * @FilePath: /FE-Interview.git/project/safe.md
-->
# 项目中的安全问题

## 1、中间人劫持，怎么防止。x-frame-option?白屏的喔，怎么办？也不一定嵌入iframe啊，可以嵌入脚本、图片，怎么阻止【描述】

x-frame-option、重定向、https，请求前加密（https、加密代理）、请求中规避（请求拆包）、请求后弥补（前端做一些逻辑）。嵌入非iframe的，如果已经突破了前面两关，走前端逻辑：触发DOMNodeInserted、DOMContentLoaded、DOMAttrModified事件。或者是给能src的标签加上自己的data-xx属性标记区分。

## 2、csrf 和 xss

XSS：恶意攻击者往 Web 页面里插入恶意 Script 代码，当用户浏览该页之时，嵌入其中 Web 里面的 Script 代码会被执行，从而达到恶意攻击用户的目的。

CSRF：CSRF 攻击是攻击者借助受害者的 Cookie 骗取服务器的信任，可以在受害者毫不知情的情况下以受害者名义伪造请求发送给受攻击服务器，从而在并未授权的情况下执行在权限保护之下的操作。
