<!--
 * @Author: Li Zhiliang
 * @Date: 2020-11-25 18:03:35
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-11-25 18:03:37
 * @FilePath: /FE-Interview.git/browser/cache.md
-->
# 浏览器缓存

- 浏览器缓存分为强缓存和协商缓存，强缓存会直接从浏览器里面拿数据，协商缓存会先访问服务器看缓存是否过期，再决定是否从浏览器里面拿数据。

- 控制强缓存的字段有：Expires和Cache-Control，Expires 和 Cache-Control。

- 控制协商缓存的字段是：Last-Modified / If-Modified-Since 和 Etag / If-None-Match，其中 Etag / If-None-Match的优先级比Last-Modified / If-Modified-Since高。

## 1. cookie 与 session 的区别

Session 是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中； Cookie 是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现 Session 的一种方式。

## 