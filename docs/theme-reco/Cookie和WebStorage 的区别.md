---
title: Cookie和WebStorage 的区别
date: 2019-05-10 11:05:12
tags: [cookie,WebStorage]
---



Web Storage是HTML5引入的一个非常重要的功能，在前端开发中经常用到，可以在客户端本地存储数据，类似HTML4的cookie，但可实现功能要比cookie强大的多，cookie大小被限制在4KB，Web Storage官方建议为每个网站5MB。78

Web Storage又分为两种：sessionStorage和localStorage

共同点：都是保存在浏览器端，且同源的，一样都是用来存储客户端临时信息的对象。他们均只能存储字符串类型的对象（虽然规范中可以存储其他原生类型的对象，但是目前为止没有浏览器对其进行实现）。 

##### sessionStorage 

* 以域名为单位进行数据划分
* 同域下的所有页面一起共享这些数据
* 对数据的改动会导致同时共享这些数据的其他页面触发storage事件

`sessionStorage`用于本地储存一个会话（session）中的资料，这些资料只有在同一个会话中的页面才能访问并且当会话结束后资料也随之销毁。因此`sessionStorage`不是一种持久化的本地储存，仅仅是会话级別的储存。

#####localStorage 

* 以每个“顶级页面（top-level browsing context）”为单位
* 在所有同域的“子页面”中共享这些数据（对数据的改动会触发storage事件）
* 为所有同域的以下“顶级页面”拷贝这些数据（通过超链接新打开的页面、通过脚本新打开的页面）

​    `localStorage`用于持久化的本地储存，除非主动删除资料，否则资料是永远不会过期的。

​       html5 web storage的浏览器支持情況: 浏览器的支援除了IE７及以下不支援外，其他标准浏览器都完全支持(ie及FF需在web服务器里执行)，值得一提的是IE总是办好事，例如IE7、IE6中的UserData其实就是javascript本地储存的解決方案。通过简单的程式码封裝可以統一到所有的浏览器都支援web storage。



####sessionStorage与localStorage的区别：

cookie数据始终在同源的http请求中携带（即使不需要），即cookie在浏览器和服务器间来回传递。而sessionStorage和localStorage不会自动把数据发给服务器，仅在本地保存。cookie数据还有路径（path）的概念，可以限制cookie只属于某个路径下。存储大小限制也不同，cookie数据不能超过4k，同时因为每次http请求都会携带cookie，所以cookie只适合保存很小的数据，如会话标识。sessionStorage和localStorage 虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大。数据有效期不同，sessionStorage：仅在当前浏览器窗口关闭前有效，自然也就不可能持久保持；localStorage：始终有效，窗口或浏览器关闭也一直保存，因此用作持久数据；cookie只在设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭。作用域不同，sessionStorage不在不同的浏览器窗口中共享，即使是同一个页面；localStorage 在所有同源窗口中都是共享的；cookie也是在所有同源窗口中都是共享的。Web Storage 支持事件通知机制，可以将数据更新的通知发送给监听者。Web Storage 的 api 接口使用更方便。

localStorage生命周期是永久，这意味着除非用户显示在浏览器提供的UI上清除localStorage信息，否则这些信息将永远存在。



####web storage和cookie共同点：

都是保存在浏览器端,且同源的



####web storage和cookie的区别：

#####webStorage:

1. 是为了更大的容量存储设计的。比cookie大得多，可以达到5M或更大。
2. Web Storage拥有setItem,getItem,removeItem,clear等方法，不像cookie需要前端开发者自己封装setCookie，getCookie。
3. webStorage仅仅是为了本地 "存储" 数据而生的。

#####Cookie:

1. 大小是受限的，并且你每请求一次新的页面的时候,cookie都会被发送过去,这个无形中浪费了带宽。

2. cookie是需要指定作用域的，是不可以跨域调用的。
3.  需要前端开发自己封装setCookie,getCookie。
4. cookie的作用是与服务器进行交互，作为HTTP规范的一部分而存在。
5. cookie数据还有路径（path）的概念，可以限制。cookie只属于某个路径下。

6. cookie存在安全性问题。如果cookie被人拦截了，那人就可以取得所有的session信息。