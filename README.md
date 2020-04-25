# 前端基础知识(面试题)，不断更新并补上对应的文章(答案)

### JavaScript
-  let、const、var 的区别有哪些？- var可变量提升，可重复声明，无暂存死区，无块级作用域  
-  [this详解](http://www.inode.club/webframe/javascript/this.html)
-  数据类型和引用类型(深浅拷贝，堆栈问题) 
   + string 、number 、boolean 和 null undefined 这五种类型统称为原始类型（Primitive），表示不能再细分下去的基本类型;
   + symbol 是ES6中新增的数据类型，symbol 表示独一无二的值，通过 Symbol 函数调用生成，由于生成的 symbol 值为原始类型，所以 Symbol 函数不能使用new 调用；
   + object 属于引用类型，array 和 function 是 object 的子类型。对象在逻辑上是属性的无序集合，是存放各种值的容器。对象值存储在堆的是引用地址，所以对象值是可变的。

-  深浅拷贝实则是一种递归算法，可以利用 weakMap 弱引用来存储防止爆栈  https://juejin.im/post/5d6aa4f96fb9a06b112ad5b1
-  [节流和防抖](https://segmentfault.com/a/1190000016261602)   
-  解释prototype和__proto__的区别 https://www.cnblogs.com/myfirstboke/p/10449272.html  https://www.cnblogs.com/shamoyuu/p/prototype.html 
-  [new()到底做了些什么](https://www.jianshu.com/p/e7015984f608)  
   ```js
   function newObject() {

       let obj = {}; //创建 一个新对象

       Con = [].shift.call(arguments); //shift方法去除数组第一个元素，并返回第一个元素

       obj.__proto__ = Con.prototype;//实例的隐式原型指向构造函数原型，

       let result = Con.apply(obj, arguments);
       //使用 apply，改变构造函数 this 的指向到新建的对象，这样 obj 就可以访问到构造函数中的属性

       return typeof result === 'object' ? result : obj; //返回该对象
   };
   
   newObject(Person, 'name', 'age')
   ```
-  闭包原理和作用  
   + 函数里的子函数，该函数可以是匿名函数，该子函数能够读写父函数的局部变量。
-  观察者模式和发布订阅模式区别 [详细讲解发布订阅模式](https://juejin.im/post/5d69eef7f265da03f12e70a5) 
-  事件模型(浏览器区别，事件代理，内存问题)  
-  垃圾回收和内存泄露   https://juejin.im/post/5d0706a6f265da1bc23f77a9  https://blog.csdn.net/weixin_42674359/article/details/88798648  https://blog.csdn.net/weixin_38098195/article/details/81135137 （实操）https://cloud.tencent.com/developer/article/1356698
-  setInterval、setTimeout和requestAnimationFrame https://gyufei.github.io/2018/09/15/requestAnimationFrame/
-  解释下事件任务队列(event loop) 
>>script(宏任务) - 清空微任务队列 - 执行一个宏任务  - 清空微任务队列 - 执行一个宏任务， 如此往复。  
   1.先执行script里的同步代码（此时是宏任务）。碰到异步任务，放到任务队列。  
   2.查找任务队列有没有微任务，有就把此时的微任务全部按顺序执行 （这就是为什么promise会比setTimeout先执行，因为先执行的宏任务是同步代码，setTimeout被放进任务队列了，setTimeout又是宏任务，在它之前先得执行微任务(就比如promise)）。  
   3.执行一个宏任务（先进到队列中的那个宏任务），再把这次宏任务里的宏任务和微任务放到任务队列。  
   4.一直重复2、3步骤  
-  js异步都有哪些，延伸Promise.all和Promise.race用法  
-  for in，for of，Object.keys和Object.getOwnPropertyNames的区别  
-  了解哪些数据结构和算法  
-  [前端性能优化，长列表如何优化](https://juejin.im/post/5b960fcae51d450e9d645c5f)  
-  [前端缓存，浏览器缓存](https://mp.weixin.qq.com/s/Q2h1EEKubAXkaM4g85Mkrw)    
-  前端安全相关问题  https://mp.weixin.qq.com/s/tgIWXcfzswcKcKcb-Dzt_g https://juejin.im/post/5bad9140e51d450e935c6d64
-  MVC和MVVM的区别  
-  简单讲解下vue2原理，是否有了解vue3的变化 (最主要就是数据劫持，object.defineProperty需要遍历对象的属性作操作，proxy是拦截当前对象作操作。还有其他待补充) 
-  vuex原理和为什么要使用以及状态持久化(vuex在vue里当作插件引入，并且被数据劫持。持久化可通过store plugins做存储，[vuex-persistedstate](https://github.com/robinvdvleuten/vuex-persistedstate), [IndexedDB](http://www.ruanyifeng.com/blog/2018/07/indexeddb.html)之类)  
-  vue-router原理  (基于Hash和History API)
-  VNode的理解  
-  keep-alive的理解 (用来对组件进行缓存，从而节省性能，由于是一个抽象组件，所以在页面渲染完毕后不会被渲染成一个DOM元素,例如页面的切换不再刷新，钩子的触发顺序created-> mounted-> activated，退出时触发deactivated。当再次进入（前进或者后退）时，只触发activated。)  
-  函数式编程([高阶函数，高阶组件](https://blog.csdn.net/mapbar_front/article/details/79697863))  
-  装饰器模式 https://segmentfault.com/a/1190000018277217  
-  webpack js分包的操作，它是如何压缩前端代码  
-  ssr，pwa的了解  
-  electron(electron-vue)的了解 (基于h5技术开发的混合桌面应用 http://electronjs.org/docs) 
-  koa2的原理，是否了解其他的框架？(不局限于nodejs)  
-  TCP,HTTP,WebSocket的关系于异同，HTTP协议有哪些  https://blog.csdn.net/sinat_31057219/article/details/72872359
-  GET和POST区别  https://segmentfault.com/a/1190000018129846
-  浏览器重绘和重排 https://mp.weixin.qq.com/s/doGi80RRf1LZ3IevHzM3xA    
-  柯里化 https://cloud.tencent.com/developer/article/1356699  
   ```js
   const curry = (fn, ...args) => {
      return args.length < fn.length ? (...ar) => curry(fn,...args,...ar): fn(...args)
   }

   function add(a,d,c){
      return a+d+c 
   }
   console.log(curry(add)(1,2,3))
   ```
-  解释下call，apply和bind用法 (都可以改变上下文，前两者传参方式不同，而且调用立即执行，但bind并不会 http://web.jobbole.com/83642/)  
   ```js
   // 模拟 call 原理
   Function.prototype.callMock = function(){
       let [self, ...args] = [...arguments]
      
       //如果没有this，在浏览器中代表window，node.js代表global
       if(!self) self = typeof window === 'undefined' ? global : window

       self.fn = this //把当前方法（this）挂载在 self 作用域某个属性上

       let rest = self.fn(...args)
       delete self.fn // 移除属性
       return rest
   }

   const fullName = function() {
     		 console.log(this.firstName+this.lastName)
   }
   var person = {
       firstName:"mon",
       lastName: "w3c",
   }

   fullName.callMock(person); 

   ```
-  谈谈mpvue，taro，uniapp，原生小程序等等经验 (分别基于vue和react语法，方便h5复用某些组件结构逻辑(按照我们的经验，大概有50%可以复用，调整的细节还是不少)，快速迁移到小程序) ，踩坑及解决方法  
-  RN，ionic，flutter的了解和实践  
-  什么是跨域？前端跨域有哪些处理方法？(相同的域，相同的端口，相同的协议; CORS, new Image, document.domain, Jsonp, postMessage)  
-  GraphQL的了解  
-  Docker相关的了解  
-  Nginx相关的了解  
-  webGl相关的了解  
-  typescript相关的了解
-  rxjs相关的了解
-  [isNaN与Number.isNaN的区别](https://blog.csdn.net/zforler/article/details/88074075)
-  [exports和module.exports的区别](https://juejin.im/post/5d5639c7e51d453b5c1218b4)
-  [浏览器系列之 Cookie 和 SameSite 属性](https://github.com/mqyqingfeng/Blog/issues/157)  
-  [关于 Uniapp](https://juejin.im/book/5da9d16c5188254796427201)
-  [数组去重](https://juejin.im/post/5ceebfe4f265da1bb96fc09c#heading-0)
-  flat扁平化 
   ```js
   [1, [2, [3, [4]], 5]].flat(Math.pow(2, 53) - 1)  
   -------------
   String([1,[2,3,[4,5,6,[7,8]]]]).split(',')
   String(["🐷", ["🐶", "🐂"], ["🐎", ["🐑", ["🐲"]], "🐛"]]).split(',')
   ```
-  a==1&&a==2&&a==3
   ```js
   let i = 1
   let a = new Proxy({},{

     get: function(){

       return () => i++
     }
   })

   console.log(a==1&&a==2&&a==3&&a==4)


   let b = [1,2,3]

   b.join = b.shift

   console.log(b==1&&b==2&&b==3)

   
   ```
-  ["1", "2", "3"].map(parseInt)  
   拆解：
   + parseInt("1",0);//第二个参数为进制，此时0会忽略，根据 string 以 1 ~ 9 的数字开头，parseInt() 将把它解析为十进制的整数1。
   + parseInt("2",1);//此时将2转为1进制数，由于超过进制数1，所以返回NaN。
   + parseInt("3",2);//此时将3转为2进制数，由于超过进制数1，所以返回NaN。

## CSS
-  BFC  
   ```html
   创建BFC
   + 根元素
   + 浮动元素（float 属性不为 none）
   + position 为 absolute 或 fixed
   + overflow 不为 visible 的块元素
   + display 为 inline-block, table-cell, table-caption
   
   作用
   + 防止边距重叠
   + 清除内部浮动
   ```
   
-  flex布局  
-  动画  
-  BEM  
-  盒子模型

## 实用代码
-  [JavaScript实现base64编码解码](https://www.cnblogs.com/mofish/archive/2012/02/25/2367858.html)

## 其他
-  前端代码开发完成，咱们是先上线页面，还是先上线静态资源？（消息摘要算法，http://web.jobbole.com/93678/)   
-  谈谈带团队，跟项目的经验  
>>团建(运动，聚会)，团队开源项目和blog，每周分享，每天站会，单独探讨
-  IQ题和EQ题  
-  有无开源作品  
-  有无做过分享
-  开发风格 (https://cn.vuejs.org/v2/style-guide/index.html)  
-  前端技术架构 (技术架构不是面向具体功能的，而是面向业务开发团队的需求，解决开发共性，简化开发流程。 https://mp.weixin.qq.com/s/QKWhFSqs8Qa8AxQsZZns9w https://mp.weixin.qq.com/s/v_tP-tiH327gwWliBHqT1A )  
-  moment.js 打包时的问题和处理
-  线上前端项目首屏静态资源 gzip 后的体积是多少

