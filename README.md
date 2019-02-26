# 我也来整理一份前端基础知识(面试题)，不断更新并补上对应的答案

### JavaScript
-  数据类型和引用类型(深浅拷贝，堆栈问题)  
-  节流和防抖  
-  解释prototype和__proto__的区别  
-  闭包原理和作用  
-  观察者模式和发布订阅模式区别  
-  事件模型(浏览器区别，事件代理，内存问题)  
-  解释下事件任务队列(event loop)  
-  js异步都有哪些，延伸Promise.all和Promise.race用法  
-  for in，for of，Object.keys和Object.getOwnPropertyNames的区别  
-  了解哪些数据结构和算法  
-  前端性能优化，长列表如何优化  
-  前端缓存  
-  前端安全相关问题  https://mp.weixin.qq.com/s/tgIWXcfzswcKcKcb-Dzt_g
-  MVC和MVVM的区别  
-  简单讲解下vue2原理，是否有了解vue3的变化 (最主要就是数据劫持，object.defineProperty需要遍历对象的属性作操作，proxy是拦截当前对象作操作。还有其他待补充) 
-  vuex原理和为什么要使用以及状态持久化(vuex在vue里当作插件引入，并且被数据劫持。持久化可通过store plugins做存储，[IndexedDB](http://www.ruanyifeng.com/blog/2018/07/indexeddb.html)之类)  
-  vue-router原理  (基于Hash和History API)
-  VNode的理解  
-  keep-alive的理解 (用来对组件进行缓存，从而节省性能，由于是一个抽象组件，所以在页面渲染完毕后不会被渲染成一个DOM元素,例如页面的切换不再刷新，钩子的触发顺序created-> mounted-> activated，退出时触发deactivated。当再次进入（前进或者后退）时，只触发activated。)  
-  函数式编程([高阶函数，高阶组件](https://blog.csdn.net/mapbar_front/article/details/79697863))  
-  webpack js分包的操作  
-  ssr，pwa的了解  
-  electron(electron-vue)的了解 (基于h5技术开发的混合桌面应用 http://electronjs.org/docs) 
-  koa2的原理，是否了解其他的框架？(不局限于nodejs)  
-  TCP,HTTP,WebSocket的关系于异同，HTTP协议有哪些  https://blog.csdn.net/sinat_31057219/article/details/72872359
-  GET和POST区别  https://segmentfault.com/a/1190000018129846
-  浏览器重绘和重排 https://mp.weixin.qq.com/s/doGi80RRf1LZ3IevHzM3xA    
-  谈谈函数式编程  
-  解释下call，apply和bind用法 (都可以改变上下文，前两者传参方式不同，而且调用立即执行，但bind并不会 http://web.jobbole.com/83642/)    
-  谈谈mpvue，taro，原生小程序等等经验 (分别基于vue和react语法，方便h5复用某些组件结构逻辑(按照我们的经验，大概有50%可以复用，调整的细节还是不少)，快速迁移到小程序) 
-  RN，flutter，ionic的了解和实践  
-  什么是跨域？前端跨域有哪些处理方法？(相同的域，相同的端口，相同的协议; CORS, new Image, document.domain, Jsonp, postMessage)  
-  GraphQL的了解  
-  Docker相关的了解  
-  Nginx相关的了解  
-  webGl相关的了解  


## CSS
-  flex布局  
-  动画  


## 其他
-  前端代码开发完成，咱们是先上线页面，还是先上线静态资源？（消息摘要算法，http://web.jobbole.com/93678/)   
-  谈谈带团队，跟项目的经验  
-  IQ题和EQ题  
-  有无开源作品  
-  有无做过分享
-  开发风格 (https://cn.vuejs.org/v2/style-guide/index.html)  
-  前端技术架构 (技术架构不是面向具体功能的，而是面向业务开发团队的需求，解决开发共性，简化开发流程。 https://mp.weixin.qq.com/s/QKWhFSqs8Qa8AxQsZZns9w https://mp.weixin.qq.com/s/v_tP-tiH327gwWliBHqT1A )  

