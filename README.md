# 前端基础知识，不断更新并补上对应的文章(答案)

## JavaScript
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
-  尾调用 [相关文章](https://segmentfault.com/a/1190000020694801)
   + 某个函数的最后一步是调用另一个函数；
   + 由于是函数的最后一步操作，所以不需要保留外层函数的调用记录，只要直接用内层函数的调用记录取代外层函数的调用记录就可以了，调用栈中始终只保持了一条调用帧。
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
   
   // es6默认值
   function factorial(n, total = 1) {
        if (n === 1) return total;
        return factorial(n - 1, n * total);
   };
   console.log(factorial(5)); // 120
   console.log(factorial(4)); // 24
   
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
   + prototype是构造函数的属性。
   + __proto__ 是每个实例都有的属性，可以访问 [[prototype]] 属性。
   + 实例的__proto__ 与其构造函数的prototype指向的是同一个对象。
   ```js
   function Student(name) {
       this.name = name;
   }
   Student.prototype.setAge = function(){
       this.age = 30;
   }
   let mon = new Student('mon');
   console.log(mon.__proto__);
   console.log(Student.prototype);
   console.log(mon.__proto__ === Student.prototype);
   ```
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
-  js异步都有哪些，延伸 `Promise.all` 和 `Promise.race` 的[实现](https://github.com/YvetteLau/Blog/issues/2)和用法  
   + promise.all中的执行顺序是并行的，但是会等全部完成的结果传递给then
   + 执行顺序，promise是then方法调用之后才会执行吗？还是从创建那一刻就开始执行？ promise从创建那一刻就开始执行，只是把结果传递给了then，then与promise的执行无关。
-  [Promise的源码实现（完美符合Promise/A+规范）](https://juejin.im/post/5c88e427f265da2d8d6a1c84)
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
-  简单讲解下vue2原理，是否有了解vue3的变化 (最主要就是数据劫持，object.defineProperty需要遍历对象的属性作操作，proxy是拦截当前对象作操作。还有其他待补充) [Vue.js 技术揭秘](https://ustbhuangyi.github.io/vue-analysis/)
-  vuex原理和为什么要使用以及状态持久化(vuex在vue里当作插件引入，并且被数据劫持。持久化可通过store plugins做存储，[vuex-persistedstate](https://github.com/robinvdvleuten/vuex-persistedstate), [IndexedDB](http://www.ruanyifeng.com/blog/2018/07/indexeddb.html)之类)  
-  vue-router原理  (基于Hash和History API)
-  VNode的理解  
-  keep-alive的理解 (用来对组件进行缓存，从而节省性能，由于是一个抽象组件，所以在页面渲染完毕后不会被渲染成一个DOM元素,例如页面的切换不再刷新，钩子的触发顺序created-> mounted-> activated，退出时触发deactivated。当再次进入（前进或者后退）时，只触发activated。)  
-  函数式编程([高阶函数，高阶组件](https://blog.csdn.net/mapbar_front/article/details/79697863))  
-  装饰器模式 https://segmentfault.com/a/1190000018277217  
-  webpack js分包的操作，它是如何压缩前端代码  
-  [从零实现webpack热更新 HMR](https://juejin.im/post/5df36ffd518825124d6c1765)
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
-  解释下call，apply和bind用法 (都可以改变上下文，前两者传参方式不同，而且调用立即执行，但bind并不会)  
   ```js
   // 模拟 call 原理，apply同理类似
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
   
   
   // 模拟 bind
   Function.prototype.bind = function () {
        const self = this;                          // 保存原函数
        let context = [].shift.call(arguments);     // 取出第一个参数context上下文
        let args = [...arguments].slice(1);         // 取出剩余的参数转为数组
        return function () {                        // 返回一个新函数
            let res = [...args, ...arguments];
            self.apply(context,res);
        }
   }
   ```
-  ```js
    let obj = {
     a: 1,
     f () {
       return this.a
     }
   }
   const r = obj.f.bind({ a: 2 }).bind({ a: 3 }).call({ a: 4 })
   console.log(r)
   // 返回多少？为什么？
   ```
   当我们用一个函数调用bind的时候，返回的函数中会保存这当前this（obj.f）和a参数。所以最后调用call的时候执行的函数是目标函数，也就是调用了bind的函数，这些都是无法被修改的了，但是参数是调用bind和call时可以叠加的，这是唯一可以修改的地方。执行两次bind的原理可以参考bind的源码，和call的差不多，也是目标函数和this是被固定的了，只有参数列表会叠加。
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
   [1, [2, [3, [4]], 5]].flat(Number.MAX_SAFE_INTEGER)
   // Number.MAX_SAFE_INTEGER == Math.pow(2, 53) - 1
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
-  [Async/Await真的只是简单的语法糖吗？](https://kiwenlau.com/2018/07/18/javascript-engine-await-promise/)
-  为什么 0.1 + 0.2 != 0.3 ?
   + js没有整数概念，其实是双精度浮点数。
   + 0.1 + 0.2 != 0.3 是因为在进制转换和进阶运算的过程中出现精度损失。

   下面是详细解释:
   JavaScript使用 Number 类型表示数字(整数和浮点数)，使用64位表示一个数字。
   
   ![](https://camo.githubusercontent.com/f1313e3e42dbc4804a37483fdf7b4ddf6f8d264c/68747470733a2f2f6e6f74652e796f7564616f2e636f6d2f7977732f7075626c69632f7265736f757263652f66343537303163633431303530346537316462626362643838363165386430632f786d6c6e6f74652f5745425245534f5552434566316132626430346538646561383732656264373832383966646438623138622f3239343938)

   图片说明:

   第0位：符号位，0表示正数，1表示负数(s)
   第1位到第11位：储存指数部分（e）
   第12位到第63位：储存小数部分（即有效数字）f
   计算机无法直接对十进制的数字进行运算, 需要先对照 IEEE 754 规范转换成二进制，然后对阶运算。

   1.进制转换

   0.1和0.2转换成二进制后会无限循环

   0.1 -> 0.0001100110011001...(无限循环)
   0.2 -> 0.0011001100110011...(无限循环)
   但是由于IEEE 754尾数位数限制，需要将后面多余的位截掉，这样在进制之间的转换中精度已经损失。

   2.对阶运算

   由于指数位数不相同，运算时需要对阶运算 这部分也可能产生精度损失。

   按照上面两步运算（包括两步的精度损失），最后的结果是

   0.0100110011001100110011001100110011001100110011001100

   结果转换成十进制之后就是 0.30000000000000004。  
   
-  [javascript内存管理](https://www.cxymsg.com/guide/memory.html)
-  [null==0 为 false 而 null>=0 为 true](https://github.com/dt-fe/weekly/blob/v2/025.%E7%B2%BE%E8%AF%BB%E3%80%8Anull%20%3E%3D%200%3F%E3%80%8B.md)
   + >，< 为关系运算符，在设计上需要尝试转为一个number，而相等运算符在设计上，则没有这方面的考虑。
   + null > 0  // null 尝试转类型为number , Number(null) > 0，为 false。
   + null >= 0 // null 尝试转类型为number , Number(null) >= 0，为 true。
   + null == 0 // 在设计上，会 return false，所以为false。
-  ES6 模块和 CommonJS 模块的差异（ import 和 require 的区别）？
   + ES6模块在编译时，就能确定模块的依赖关系，以及输入和输出的变量；
   + CommonJS 模块，运行时加载；
   + ES6 模块自动采用严格模式，无论模块头部是否写了 "use strict"；
   + require 可以做动态加载，import 语句做不到，import 语句必须位于顶层作用域中；
   + ES6 模块中顶层的 this 指向 undefined，ommonJS 模块的顶层 this 指向当前模块；
   + CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。如：
   ```js
   //name.js
   var name = 'William';
   setTimeout(() => name = 'Github', 200);
   module.exports = {
       name
   };
   //index.js
   const name = require('./name');
   console.log(name); //William
   setTimeout(() => console.log(name), 300); //William
   ```
   + 对比 ES6 模块看一下，ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令 import ，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。
   ```js
   //name.js
   var name = 'William';
   setTimeout(() => name = 'Github', 200);
   export { name };
   //index.js
   import { name } from './name';
   console.log(name); //William
   setTimeout(() => console.log(name), 300); //Github  
   ```
   

## 算法与数据结构
-  [前端该如何准备数据结构和算法？](https://juejin.im/post/5d5b307b5188253da24d3cd1)
-  [数据结构和算法专题](http://www.conardli.top/docs/dataStructure/)
-  [如何分析时间复杂度?](https://www.cxymsg.com/guide/algorithm.html#%E5%A6%82%E4%BD%95%E5%88%86%E6%9E%90%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6)

## 小程序
-  [小程序状态响应式：你可以零侵入式实现小程序的全局状态管理吗](https://juejin.im/post/5e89768a51882573b2195205)
-  [Uniapp 从入门到进阶](https://juejin.im/book/5da9d16c5188254796427201)

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


## 浏览器
-  [图解浏览器的基本工作原理](https://zhuanlan.zhihu.com/p/47407398)

## 微前端
-  [统一运营工作台的解决方案](https://mp.weixin.qq.com/s/xmcXz5GWSEYFy18APPHwlg)

## HTTP和HTTPS
-  请求头Connection: keep-alive的优点
   + 较少的CPU和内存的使用（由于同时打开的连接的减少了）
   + 允许请求和应答的HTTP管线化
   + 降低拥塞控制 （TCP连接减少了）
   + 减少了后续请求的延迟（无需再进行握手）
   + 报告错误无需关闭TCP连接
-  浏览器强缓存和协商缓存
   ![来源于互联网](https://user-images.githubusercontent.com/25027560/38223505-d8ab53da-371d-11e8-9263-79814b6971a5.png) 
   
   
## 实用代码
-  [JavaScript实现base64编码解码](https://www.cnblogs.com/mofish/archive/2012/02/25/2367858.html)


## 自动化探索
-  [Puppeteer 爬虫实践](https://mp.weixin.qq.com/s?__biz=MzI0NTE5NzYyMw==&mid=2247483769&idx=1&sn=f4d65fe8b60d8870b0a243ad5da48048&scene=21)


## 在线电子书
-  [深入理解 TypeScript](https://jkchao.github.io/typescript-book-chinese/)
-  [Flutter完整开发实战详解](https://guoshuyu.cn/home/wx/)


## 工具/网站
-  [Storybook](https://storybook.js.org/) - 一个方便管理和测试组件的开源工具，适用于 React, Vue, 和 Angular。
-  [cypress](https://www.cypress.io/) - 更适合于前端工程师使用的自动化测试工具
-  [Vue Datav](http://datav.jiaminghi.com/) - Vue 大屏数据展示组件库

## 其他
-  前端代码开发完成，咱们是先上线页面，还是先上线静态资源？（消息摘要算法，http://web.jobbole.com/93678/)   
-  [如何推动前端团队的基础设施建设](https://mp.weixin.qq.com/s/2VSa3xBpy5St8G1v0RjW9g)
-  谈谈带团队，跟项目的经验  
   >> 团建(运动，聚会)，团队开源项目和blog，每周分享，每天站会，单独探讨
-  IQ题和EQ题  
-  有无开源作品  
-  有无做过分享
-  开发风格 (https://cn.vuejs.org/v2/style-guide/index.html)  
-  前端技术架构 (技术架构不是面向具体功能的，而是面向业务开发团队的需求，解决开发共性，简化开发流程。 https://mp.weixin.qq.com/s/QKWhFSqs8Qa8AxQsZZns9w https://mp.weixin.qq.com/s/v_tP-tiH327gwWliBHqT1A )  
-  moment.js 打包时的问题和处理
-  线上前端项目首屏静态资源 gzip 后的体积是多少
-  登录系统设计（单点登录）
   + 当公司内部业务线比较复杂但相互之间的耦合度比较高时，我们应该考虑设计添加单点登录系统。
   + 具体来说，用户在一处登录，即可以在任何页面访问，登出时，也同样在任何页面都失去登录状态。
   
   SSO的好处很多：
   + 增强用户体验；
   + 打通了不同业务系统之间的用户数据；
   + 方便统一管理用户；
   + 有利于引流；
   + 降低开发系统的成本（不需要每个业务都开发一次登录系统和用户状态控制）；

   总的来说，大中型web应用，SSO可以带来很多好处，缺点却很少。
