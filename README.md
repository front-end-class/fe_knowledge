# 前端基础知识(面试题)，不断更新并补上对应的文章(答案)

### JavaScript
-  let、const、var 的区别有哪些？- var可变量提升，可重复声明，无暂存死区，无块级作用域  
-  [this详解](http://www.inode.club/webframe/javascript/this.html)
   ```js
   自执行函数：
   
   var foo = 1
   (function foo() {
       foo = 10
       console.log(foo)
   }()) // -> ƒ foo() { foo = 10 ; console.log(foo) }
   
   因为当 JS 解释器在遇到非匿名的立即执行函数时，会创建一个辅助的特定对象，然后将函数名称作为这个对象的属性，因此函数内部才可以访问到 foo，但是这个值又是只读的，所以对它的赋值并不生效，所以打印的结果还是这个函数，并且外部的值也没有发生更改。
   ```
-  执行上下文
   + 浏览器执行js，首先进入的是一个全局的执行上下文，压入栈顶。
   + 当调用一个函数时，程序流就进入该被调用函数内，此时引擎就会为该函数创建一个新的执行上下文，并且将其压入到执行栈顶部（作用域链）。浏览器总是执行位于执行栈顶部的当前执行上下文，一旦执行完毕，该执行上下文就会从执行栈顶部弹出，并且控制权将进入其下的执行上下文。
   + 这样，执行栈中的执行上下文就会被依次执行并且弹出，直到回到全局的执行上下文。
   + 全局上下文只有唯一的一个，它在浏览器关闭时出栈。
-  作用域链
   + 无论是 LHS 还是 RHS 查询，都会在当前的作用域开始查找，如果没有找到，就会向上级作用域继续查找目标标识符，每次上升一个作用域，一直到全局作用域为止。
-  LHS和RHS查询
   + 当变量出现在赋值操作的左侧时进行LHS查询，出现在右侧时进行RHS查询。
   + RHS：查找某个变量的值（谁是赋值操作的源头），LHS：查询变量的容器本身（谁是赋值操作的目标）。
   ```js
   function foo (a) {
      var b = a;
      return a + b;
   }
   var c = foo(2);

   答案：
   LHS：c = , a = 2, b =
   RHS：foo(2), = a, return a… , … + b
   
   解析：
   首先 c = foo(2)中，“c = "进行了一次LHS查询，同时需要查询“foo(2)”的值，因此“foo(2)”便是一次RHS查询，“foo(2)”为隐式变量分配，相当于“a = 2”，“a = ”为LHS。随后进入foo方法中，“b = a”中，“b =”为LHS，此时查询a的值为2，所以“= a”为RHS。“return a + b”，均需查询a和b的值，为两次RHS。不成功的 RHS 引用会导致抛出 ReferenceError 异常。不成功的 LHS 引用会导致自动隐式地创建一个全局变量（非严格模式下），该变量使用 LHS 引用的目标作为标识符，或者抛出 ReferenceError 异常（严格模式下）。
   ```
-  "尾调用优化"（Tail call optimization），即只保留内层函数的调用帧，如果所有函数都是尾调用，那么完全可以做到每次执行时，调用帧只有一项，这将大大节省内存。
   ```js
   // TCO
   function f() {
        let m = 1;
        let n = 2;
        return g(m + n)
    }
    //等同于
    function f() {
        return g(3);
    }
    f()
    //等同于
    g(3)
    
    // no TCO
    // 内层函数inner用到了外层函数addOne的内部变量one
    function addOne(a) {
        let one = 1;

        function inner(b) {
            return b + one
        }
        return inner(a)
    }
   ```
-  函数调用自身，称为递归。如果尾调用自身就称为尾递归。尾递归由于只存在一个调用帧，所以永远不会发生“栈溢出”错误。
   ```js
   function factorial(n) {
        if (n === 1) return 1;
        return n * factorial(n - 1)
    }

    console.log(factorial(5)); // 120
    console.log(factorial(4)); // 24
    
   上面代码是一个阶乘函数，计算n的阶乘，最多需要保存n个调用记录。如果改写成尾递归，只保留一个调用记录。

   function factorial(n, total) {
        if (n === 1) return total;
        return factorial(n - 1, n * total);
   };
   console.log(factorial(5, 1)); // 120
   console.log(factorial(4, 1)); // 24
   
   计算Fibonacci数列（斐波那契数列），也能充分说明尾递归优化的重要性

    function Fibonacci(n) {
        if (n <= 1) { return 1 };
        return Fibonacci(n - 1) + Fibonacci(n - 2)
    }

    console.log(Fibonacci(10)); //89
    console.log(Fibonacci(100)); //堆栈溢出
   
   尾调用优化过后的Fibonacci数列实现如下

    function Fibonacci(n, ac1 = 1, ac2 = 1) {
        if (n <= 1) { return ac2 };
        return Fibonacci(n - 1, ac2, ac1 + ac2);
    }

    console.log(Fibonacci(10)); //89
    console.log(Fibonacci(30)); //1346269
    console.log(Fibonacci(100)); //10946
   ```
-  数据类型和引用类型(深浅拷贝，堆栈问题) 
   + string 、number 、boolean 和 null undefined 这五种类型统称为原始类型（Primitive），表示不能再细分下去的基本类型;
   + symbol 是ES6中新增的数据类型，symbol 表示独一无二的值，通过 Symbol 函数调用生成，由于生成的 symbol 值为原始类型，所以 Symbol 函数不能使用new 调用；
   + object 属于引用类型，array 和 function 是 object 的子类型。对象在逻辑上是属性的无序集合，是存放各种值的容器。对象值存储在堆的是引用地址，所以对象值是可变的。

-  深浅拷贝实则是一种递归算法，可以利用 weakMap 弱引用来存储防止爆栈  https://juejin.im/post/5d6aa4f96fb9a06b112ad5b1
-  [节流和防抖](https://segmentfault.com/a/1190000016261602)   
   + 防抖是将多次执行变为最后一次执行，节流是将多次执行变成每隔一段时间执行。
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
   ```js
   // 经典问题：
   for ( var i=1; i<=5; i++) {
      setTimeout( function timer() {
         console.log( i );
      }, i*1000 );
   }
   
   // 利用 setTimeout 的第三个参数，给第一个函数传递参数
   for ( var i=1; i<=5; i++) {
      setTimeout( function timer(j) {
         console.log( j );
      }, i*1000, i);
   }
   ```
-  数组的哪些API会改变原数组？
   ```html
   修改原数组的API有:
   splice/reverse/fill/copyWithin/sort/push/pop/unshift/shift

   不修改原数组的API有:
   slice/map/forEach/every/filter/reduce/entries/find
   ```
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
   + promise.all中的执行顺序是并行的，但是会等全部完成的结果传递给then
   + 执行顺序，promise是then方法调用之后才会执行吗？还是从创建那一刻就开始执行？ promise从创建那一刻就开始执行，只是把结果传递给了then，then与promise的执行无关。
-  async/await 
   ```js
   function sleep() {
     return new Promise(resolve => {
       setTimeout(() => {
         console.log('完成')
         resolve("sleep");
       }, 2000);
     });
   }
   async function test() {
     let value = await sleep();
     console.log("开始");
   }
   test()
   // 完成
   // 开始
   ```
   + async 和 await 相比直接使用 Promise 来说，优势在于处理 then 的调用链，能够更清晰准确的写出代码。缺点在于滥用 await 可能会导致性能问题，因为 await 会阻塞代码，也许之后的异步代码并不依赖于前者，但仍然需要等待前者完成，导致代码失去了并发性。
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
-  ES6中，子类中，super方法是必须调用的，因为子类本身没有自身的this对象，需要通过super方法拿到父类的this对象。在子类中，没有构造函数，那么在默认的构造方法内部自动调用super方法，继承父类的全部属性，子类的构造方法中，必须先调用super方法，然后才能调用this关键字声明其它属性。
   ```js
      class Student extends Person{
          constructor(name,sex){
              console.log(this);//Error
              super(name,sex);
              this.sex=sex;
          }
      }
   ```
-  [] == ![]
   + 首先，我们需要知道 ! 优先级是高于 == (更多运算符优先级可查看: 运算符优先级)
   + ![] 引用类型转换成布尔值都是true,因此![]的是false
   + 其中一方是 boolean，将 boolean 转为 number 再进行判断，false转换成 number，对应的值是 0.
   + 有一方是 number，那么将object也转换成Number,空数组转换成数字，对应的值是0.(空数组转换成数字，对应的值是0，如果数组中只有一个数字，那么转成number就是这个数字，其它情况，均为NaN)
   0 == 0; 为true
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

