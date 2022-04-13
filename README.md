# 前端知识，不断更新

## JavaScript
-  let、const、var 的区别有哪些？- var可变量提升，可重复声明，无暂存死区，无块级作用域  
-  [原型（prototype）、原型链和原型继承](https://zhuanlan.zhihu.com/p/35790971)
-  [this详解](http://www.inode.club/webframe/javascript/this.html)
-  [非匿名自执行函数](https://www.jianshu.com/p/561de763348e)
   + 因为当 JS 解释器在遇到非匿名的立即执行函数时，会创建一个辅助的特定对象，然后将函数名称作为这个对象的属性，因此函数内部才可以访问到 foo，但是这个值又是只读的，所以对它的赋值并不生效，所以打印的结果还是这个函数，并且外部的值也没有发生更改。
   ```js
   var foo = 1;
   (function foo() {
      // 'use strict'
      foo = 10
      console.log(foo)
   }()) // 返回function ƒ foo() { foo = 10 ; console.log(foo) }

   —————————转—————————

   specialObject = {};
   Scope = specialObject + Scope;
   foo = new FunctionExpression;
   foo.[[Scope]] = Scope;
   specialObject.foo = foo; // {DontDelete}, {ReadOnly}

   delete Scope[0]; // remove specialObject from the front of scope chain
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
-  关于堆栈解释
   ![](https://user-gold-cdn.xitu.io/2019/8/30/16ce31834037d16e)
   + 堆是动态分配内存，内存大小不一，也不会自动释放。
   + 栈是自动分配相对固定大小的内存空间，并由系统自动释放。栈的大小不是无限的。例如Chrome就会限定栈的最大为16,000帧。所以无限递归会导致Chrome抛出Maximum Call Stack size exceeded:
   ![](https://user-gold-cdn.xitu.io/2019/8/30/16ce31848d545bb6)
-  数据类型和引用类型(深浅拷贝，堆栈问题) 
   + string 、number 、boolean 和 null undefined 这五种类型统称为原始类型（Primitive），表示不能再细分下去的基本类型;
   + symbol 是ES6中新增的数据类型，symbol 表示独一无二的值，通过 Symbol 函数调用生成，由于生成的 symbol 值为原始类型，所以 Symbol 函数不能使用new 调用；
   + object 属于引用类型，array 和 function 是 object 的子类型。对象在逻辑上是属性的无序集合，是存放各种值的容器。对象值存储在堆的是引用地址，所以对象值是可变的。
   + 基本数据类型
   Undefined、Null、Boolean、String、Number、Symbol都是直接按值直接存在栈中，每种类型的数据占用的内存空间大小都是固定的，并且由系统自动分配自动释放。
   + 引用数据类型
   Object，Array，Function这样的数据存在堆内存中，但是数据指针是存放在栈内存中的，当我们访问引用数据时，先从栈内存中获取指针，通过指针在堆内存中找到数据。
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

       //这里的图片更容易理解   
       // https://github.com/webfansplz/article/issues/6
   };
   
   newObject(Person, 'name', 'age')
   ```
-  闭包原理和作用  
   + 函数里的子函数，该函数可以是匿名函数，该子函数能够读写父函数的局部变量，并且被全局作用域引用。
   ```js
   // 经典问题：
   for ( var i=1; i<=5; i++) {
      // 先执行主线程结束才走异步队列，i已经执行到5，所以都返回6
      setTimeout( function timer() {
         console.log( i );
      }, i*1000 );
   }

   // 自执行函数
   for ( var i=1; i<=5; i++) {
      (function(j){
         setTimeout( function timer() {
         console.log( j );
      }, j*1000 )
      })(i)
   }

   // 定义一个函数
   function st(j){
      setTimeout( function() {
         console.log( j );
      }, j*1000 )
   }
   for ( var i=1; i<=5; i++) {
      // 让st产生一个AO对象，并且形参引用至外层for循环的作用域，产生一个闭包
      st(i)
   }
   
   // 利用let
   for ( let i=1; i<=5; i++) {
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
    [最详细清晰的一篇文章](https://juejin.cn/post/6844903919789801486)
>>script(宏任务) - 清空微任务队列 - 执行一个宏任务  - 清空微任务队列 - 执行一个宏任务， 如此往复。  
   1.先执行script里的同步代码（此时是宏任务）。碰到异步任务，放到任务队列。  
   2.查找任务队列有没有微任务，有就把此时的微任务全部按顺序执行 （这就是为什么promise会比setTimeout先执行，因为先执行的宏任务是同步代码，setTimeout被放进任务队列了，setTimeout又是宏任务，在它之前先得执行微任务(就比如promise)）。  
   3.执行一个宏任务（先进到队列中的那个宏任务），再把这次宏任务里的宏任务和微任务放到任务队列。  
   4.一直重复2、3步骤  
-  setTimeout，setImmediate谁先执行？
   setImmediate和process.nextTick为Node环境下常用的方法（IE11支持setImmediate），所以，后续的分析都基于Node宿主。

   Node.js是运行在服务端的js，虽然用到也是V8引擎，但由于服务目的和环境不同，导致了它的API与原生JS有些区别，其Event Loop还要处理一些I/O，比如新的网络连接等，所以与浏览器Event Loop不太一样。

   执行顺序如下：

   1. timers: 执行setTimeout和setInterval的回调
   2. pending callbacks: 执行延迟到下一个循环迭代的 I/O 回调
   3. idle, prepare: 仅系统内部使用
   4. poll: 检索新的 I/O 事件;执行与 I/O 相关的回调。事实上除了其他几个阶段处理的事情，其他几乎所有的异步都在这个阶段处理。
   5. check: setImmediate在这里执行
   6. close callbacks: 一些关闭的回调函数，如：socket.on('close', ...)  

   一般来说，setImmediate会在setTimeout之前执行，如下：
   ```js
   console.log('outer');
   setTimeout(() => {
   setTimeout(() => {
      console.log('setTimeout');
   }, 0);
   setImmediate(() => {
      console.log('setImmediate');
   });
   }, 0);
   ```
   其执行顺序为：

   1. 外层是一个setTimeout，所以执行它的回调的时候已经在timers阶段了
   2. 处理里面的setTimeout，因为本次循环的timers正在执行，所以其回调其实加到了下个timers阶段
   3. 处理里面的setImmediate，将它的回调加入check阶段的队列
   4. 外层timers阶段执行完，进入pending callbacks，idle, prepare，poll，这几个队列都是空的，所以继续往下
   5. 到了check阶段，发现了setImmediate的回调，拿出来执行
   6. 然后是close callbacks，队列是空的，跳过
   7. 又是timers阶段，执行console.log('setTimeout')
   8. 但是，如果当前执行环境不是timers阶段，就不一定了。  
   顺便科普一下Node里面对setTimeout的特殊处理：setTimeout(fn, 0)会被强制改为setTimeout(fn, 1)。

   看看下面的例子：
   ```js
   setTimeout(() => {
   console.log('setTimeout');
   }, 0);

   setImmediate(() => {
   console.log('setImmediate');
   });
   ```

   其执行顺序为：

   1. 遇到setTimeout，虽然设置的是0毫秒触发，但是被node.js强制改为1毫秒，塞入times阶段
   2. 遇到setImmediate塞入check阶段
   3. 同步代码执行完毕，进入Event Loop
   4. 结论：**先进入times阶段，检查当前时间过去了1毫秒没有，如果过了1毫秒，满足setTimeout条件，执行回调，如果没过1毫秒，跳过
   跳过空的阶段，进入check阶段，执行setImmediate回调**  
   可见，1毫秒是个关键点，所以在上面的例子中，setImmediate不一定在setTimeout之前执行了。

-  js异步都有哪些，延伸 `Promise.all`, `Promise.any` 和 `Promise.race` 的[实现](https://github.com/YvetteLau/Blog/issues/2)和用法  
   + promise.all中的执行顺序是并行的，但是会等全部完成的结果传递给then
   + 执行顺序，promise是then方法调用之后才会执行吗？还是从创建那一刻就开始执行？ promise从创建那一刻就开始执行，只是把结果传递给了then，then与promise的执行无关。
   ```js
   // Promise.all
   Promise.all = function (promiseArr) { // 类 koa compose() 函数
     let resArr = []; //用于存放每次执行后返回结果
     return new Promise(function (resolve, reject) {
       let i = 0;
       next(); // 递归
       function next() {
         promiseArr[i].then(function (res) {
           resArr.push(res); // 存储每次得到的结果
           i++;
           if (i == promiseArr.length) {
             // 如果函数数组中的函数都执行完，便resolve
             resolve(resArr);
           } else {
             next();
           }
         });
       }
     });
   }
   
   //Promise.race
   Promise.race = function (promiseArr) {
    promiseArr = Array.from(promiseArr);//将可迭代对象转换为数组
    return new Promise((resolve, reject) => {
        if (promises.length === 0) {
            return;
        } else {
            for (let i = 0; i < promiseArr.length; i++) {
                Promise.resolve(promiseArr[i]).then((data) => {
                    resolve(data);
                    return;
                }, (err) => {
                    reject(err);
                    return;
                });
            }
        }
    });
   }
   ```
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
-  MVC和MVVM的区别  
-  函数式编程([高阶函数，高阶组件](https://blog.csdn.net/mapbar_front/article/details/79697863))  
-  装饰器模式 https://segmentfault.com/a/1190000018277217  
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
-  flattenDeep多维数组降维
   ```js
   const flattenDeep = (arr) => Array.isArray(arr)
   ? arr.reduce( (a, b) => [...a, ...flattenDeep(b)] , [])
   : [arr]

   flattenDeep([1, [[2], [3, [4]], 5]])
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

-  转换规则：
   + 对象 == 字符串  对象.toString()为字符串
   + null == undefined 为true，和其他值比较不相等
   + NaN == NaN false
   + 剩下的都最终转为数字比较

-  [null==0 为 false 而 null>=0 为 true](https://github.com/dt-fe/weekly/blob/v2/025.%E7%B2%BE%E8%AF%BB%E3%80%8Anull%20%3E%3D%200%3F%E3%80%8B.md)
   + >，< 为关系运算符，在设计上需要尝试转为一个number，而相等运算符在设计上，则没有这方面的考虑。
   + null > 0  // null 尝试转类型为number , Number(null) > 0，为 false。
   + null >= 0 // null 尝试转类型为number , Number(null) >= 0，为 true。
   + null == 0 // 在设计上，会 return false，所以为false。
   
-  [javascript内存管理](https://www.cxymsg.com/guide/memory.html)

-  ES6 模块和 CommonJS 模块的差异（ [import 和 require 的区别](https://zhuanlan.zhihu.com/p/161015809)）[深入 CommonJs 与 ES6 Module](https://www.cnblogs.com/qixidi/p/10287679.html)？
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
-  [Tree-Shaking原理实践](https://cloud.tencent.com/developer/article/1688857)  
-  [关于proxy和symbol特点](https://mp.weixin.qq.com/s/4wt-ulMx6EKcV75W5ioPrw)  
-  [npm 包管理机制](https://cloud.tencent.com/developer/article/1556014)  
-  [peerDependencies](https://segmentfault.com/a/1190000022435060) 
-  [package.json 知多少？](https://mp.weixin.qq.com/s?__biz=Mzg2NDAzMjE5NQ==&mid=2247484994&idx=1&sn=7da1108c3858ad71c42bcb36f8ade9de&chksm=ce6ec2eef9194bf863bb5ddeb9cb5fa7507e3a5f27b6dde0f61946c2c378eb89d8f562424ca7&scene=178&cur_album_id=1403155327595495424#rd) 
-  箭头函数
   + 没有自己的this，取决于它外面的“第一个不是箭头函数的函数”的 this，指向调用函数的上一层运行时
   ```js
   let obj = {
   a: 'mon',
   foo: () => {
      console.log(this)
   },
   }

   obj.foo() // 输出结果: "Window {...}"。obj.foo在调用的时候如果是不使用箭头函数 this 应该指向的是 obj, 输出"{a: "mon", foo: ƒ}"，但是使用了箭头函数，取决于它外面的“第一个不是箭头函数的函数”的 this, 指向的就是全局了，所以输出结果是 "Window {...}"。
   ```
   + 没有 arguments，拿的是箭头函数外层函数的 arguments 属性
   ```js
   function fun() {
      return () => [...arguments].length
   }

   let result = fun(1,2)
   console.log(result()) // 2
   ```
   + 没有构造函数
   ```js
   let fun = () => {}
   let funNew = new fun()
   // 报错内容 TypeError: fun is not a constructor
   ```
   + 没有原型
   ```js
   let fun = () => {}
   console.loh(fun.prototype) // undefined
   ```
   + 没有 super 函数，没有原型，自然没有 super
-  [script defer和async的区别](https://segmentfault.com/q/1010000000640869)  

   默认情况下，脚本的下载和执行将会按照文档的先后顺序同步进行。当脚本下载和执行的时候，文档解析就会被阻塞，在脚本下载和执行完成之后文档才能往下继续进行解析。

   下面是async和defer两者区别：

   - 当script中有defer属性时，脚本的加载过程和文档加载是异步发生的，等到文档解析完(DOMContentLoaded事件发生)脚本才开始执行。

   - 当script有async属性时，脚本的加载过程和文档加载也是异步发生的。但脚本下载完成后会停止HTML解析，执行脚本，脚本解析完继续HTML解析。

   - 当script同时有async和defer属性时，执行效果和async一致。

## AST
-  [AST详解与运用](https://zhuanlan.zhihu.com/p/266697614)
-  [像玩 jQuery 一样玩 AST](https://juejin.cn/post/6923936548027105293)
-  [the-super-tiny-compiler](https://github.com/jamiebuilds/the-super-tiny-compiler)

## 算法与数据结构
-  [前端该如何准备数据结构和算法？](https://juejin.im/post/5d5b307b5188253da24d3cd1)
-  [数据结构和算法专题](http://www.conardli.top/docs/dataStructure/)
-  [如何分析时间复杂度?](https://www.cxymsg.com/guide/algorithm.html#%E5%A6%82%E4%BD%95%E5%88%86%E6%9E%90%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6)



## 小程序，Uni-app
-  [小程序状态响应式：你可以零侵入式实现小程序的全局状态管理吗](https://juejin.im/post/5e89768a51882573b2195205)
-  [Uniapp 从入门到进阶](https://juejin.im/book/5da9d16c5188254796427201)
-  [mp-html 小程序富文本组件](https://github.com/jin-yufeng/mp-html)
-  [wxparse](https://github.com/icindy/wxparse)


## Vue、React相关
-  [Vue3.0 有哪些特点](https://juejin.cn/post/6896438269291347976)
-  简单讲解下vue2原理，是否有了解vue3的变化 (最主要就是数据劫持，object.defineProperty需要遍历对象的属性作操作，proxy是拦截当前对象作操作。还有其他待补充) [Vue.js 技术揭秘](https://ustbhuangyi.github.io/vue-analysis/)
-  vuex原理和为什么要使用以及状态持久化(vuex在vue里当作插件引入，并且被数据劫持。持久化可通过store plugins做存储，[vuex-persistedstate](https://github.com/robinvdvleuten/vuex-persistedstate), [IndexedDB](http://www.ruanyifeng.com/blog/2018/07/indexeddb.html)之类)  
-  vue-router原理  (基于Hash和History API)
-  [VNode的详解](https://segmentfault.com/a/1190000016129036)   
-  [b站-快速掌握虚拟DOM和diff算法](https://www.bilibili.com/video/BV1dV411a7mT) 
-  [React性能优化：Virtual Dom原理浅析](https://zhuanlan.zhihu.com/p/36798520)
-  [深入分析虚拟DOM的渲染过程和特性](https://mp.weixin.qq.com/s?__biz=Mzg2NDAzMjE5NQ==&mid=2247484212&idx=1&sn=e4cf00c0c087c34ae2f181e7b2f2b257&chksm=ce6ec798f9194e8e0ef0327e9a4cbb8d1454218609a20663a5ea5fed9e31f4a39b7fb114e117&scene=21#wechat_redirect)
-  keep-alive的理解 (用来对组件进行缓存，从而节省性能，由于是一个抽象组件，所以在页面渲染完毕后不会被渲染成一个DOM元素,例如页面的切换不再刷新，钩子的触发顺序created-> mounted-> activated，退出时触发deactivated。当再次进入（前进或者后退）时，只触发activated。)  
-  [为什么vue的data必须是一个函数](https://www.jianshu.com/p/dc95ce948a07)
   + data return 出一个唯一的独立的对象，不能是一个共享的内存地址的对象
-  [Vue源码解析之nextTick](https://juejin.cn/post/6844903728995106823)
-  [手写Vue2.0]()
-  [手写React]()
-  [React 中的 ErrorBoundary 实用性](https://github.com/shfshanyue/Daily-Question/issues/11)
-  [React hooks，它带来了那些便利](https://segmentfault.com/a/1190000022163955)
-  [一文归纳 React Hooks 常用场景](https://juejin.cn/post/6918896729366462471)
-  [10分钟教你手写8个常用的自定义hooks](https://juejin.cn/post/6844904074433789959)
-  [用好这9个hooks，所向披靡](https://juejin.cn/post/6895966927500345351)
-  [React 项目性能分析及优化](https://github.com/brickspert/blog/issues/36)
-  [网页性能提升5倍 — 构建优化篇](https://mp.weixin.qq.com/s?__biz=Mzg2NDAzMjE5NQ==&mid=2247487514&idx=1&sn=357eed76ec82c1beb0a30a5e8e2008fb&chksm=ce6ed4b6f9195da0870d343b7efd3d13fd2e8e25ef9ddc5a400cd9a002312c07c7d095964613&scene=178&cur_album_id=1403155327595495424#rd)
-  [从Mixin到HOC再到Hook](https://mp.weixin.qq.com/s?__biz=Mzg2NDAzMjE5NQ==&mid=2247484193&idx=1&sn=0c152afb3566f2b600b0e218d5d24017&chksm=ce6ec78df9194e9b3e0e4f18706fd84be22f0f84d1e7554892e169a30265c6fa154731c1c7a3&scene=21#wechat_redirect)
-  [轻烤 React 核心机制：React Fiber 与 Reconciliation](https://juejin.cn/post/6891242214324699143)
-  [深入理解Redux Middleware](https://mp.weixin.qq.com/s/3yoHo6UXI2VOPO9zWI2aCQ)
-  [理解redux-thunk](https://zhuanlan.zhihu.com/p/85403048)
-  [redux-saga文档](https://redux-saga-in-chinese.js.org/)
-  [redux-saga入门](https://zhuanlan.zhihu.com/p/85518538)
-  [Dvajs](https://dvajs.com/guide/)




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
   
-  [Flexbox布局](https://segmentfault.com/a/1190000017115802) [10分钟理解FlexBox](https://segmentfault.com/a/1190000017347626) 
-  动画  
-  BEM  
-  盒子模型
-  [关于flex-shrink知识](https://segmentfault.com/a/1190000016746670)



## 工程化
-  [Monoreopo 前端多项目管理方式（yarn workspaces, lerna）](https://segmentfault.com/a/1190000019309820)
-  [Monoreopo工具介绍](https://juejin.cn/post/6913497232687759367)
-  升级npm7.0自带workspace 
-  [一种自动生成网页骨架屏的方式](https://github.com/famanoder/dps)
-  [vite工程化](https://juejin.cn/post/6910014283707318279)
-  [打造小团队前端工程化服务,基本该有都有](https://juejin.cn/post/6867861517603438605)



## Webpack，Rollup，Parcel，Snowpack，Vite
-  Webpack 对比 Rollup
   + Webpack的优势在于它更全面，基于“一切皆模块”的思想而衍生出丰富的loader和plugin可以满足各种使用场景；
   + 而Rollup则更像一把 手术刀，它更专注于JavaScript的打包。当然Rollup也支持许多其他类型的模块，但是总体而言在通用性上还是不如Webpack。
   + 如果当前的项目需求仅仅是打包JavaScript，比如一个JavaScript库，那么Rollup很多时候会是我们的第一选择。
-  [Tree Shaking 原理实践](https://cloud.tencent.com/developer/article/1688857)
-  [webpack中loader和plugin的区别](https://www.cnblogs.com/cxyqts/p/13722569.html)
   + loader运行在打包文件之前，文件转换；plugins扩展wp功能，为处理loader无法处理的事物，在整个编译周期都起作用
-  [webpack 打包原理 ? 看完这篇你就懂了 !](https://github.com/webfansplz/article/issues/38)
-  [hard-source-webpack-plugin提升构建速度](https://github.com/mzgoddard/hard-source-webpack-plugin)
-  [带你深度解锁Webpack系列](https://segmentfault.com/a/1190000022205477)
-  [Rollup](https://rollupjs.org/)
-  [Snowpack](https://www.snowpack.dev/)
-  [实现一个 webpack loader 和 webpack plugin](https://juejin.cn/post/6871239792558866440)
-  [实现一个真正的babel插件（不仅仅是替换字符）及 ast操作原理](https://zhuanlan.zhihu.com/p/91948992)
-  [webpack js分包的操作，它是如何压缩前端代码](https://www.timsrc.com/article/21/bundle-spliting)  
-  [从零实现webpack热更新 HMR](https://juejin.im/post/5df36ffd518825124d6c1765)
-  [深入浅出 Babel 上篇：架构和原理 + 实战](https://juejin.im/post/5d94bfbf5188256db95589be)  
   ![](https://user-gold-cdn.xitu.io/2019/10/2/16d8d0cd5a3f3a0c)
-  [深入浅出 Babel 下篇：既生 Plugin 何生 Macros](https://juejin.im/post/5da12397e51d4578364f6ffa)



## 性能优化
-  [前端性能优化](https://zhuanlan.zhihu.com/p/113864878)
-  图片懒加载关键 getBoundingClientRect


## 低代码
-  [何谓低代码开发](https://juejin.cn/post/6918355418326499341)
-  [amis](https://baidu.gitee.io/amis/zh-CN/docs/index)
-  [page-schema-player](https://github.com/ufologist/page-schema-player)
-  [json-schema](https://json-schema.org/understanding-json-schema/index.html)



## 浏览器
-  [图解浏览器的基本工作原理](https://zhuanlan.zhihu.com/p/47407398)
-  [ESM模块化](https://juejin.cn/post/6854573213985275911)
-  localhost:3000 与 localhost:5000 的 cookie 信息是否共享？  
   对于浏览器实现来说，“cookie区分域，而不区分端口，同一个ip下的多个端口下的cookie是共享的！


## Nginx
-  [五分钟看懂 Nginx 负载均衡](https://mp.weixin.qq.com/s/u-XbBwGxHrhJGiMiiqz26w)
-  [理解X-Forwarded-For与防范](https://juejin.cn/post/6844904174132396045)
-  [egg.js前置代理模式](https://eggjs.org/zh-cn/tutorials/proxy.html)

## 微前端
-  [统一运营工作台的解决方案](https://mp.weixin.qq.com/s/xmcXz5GWSEYFy18APPHwlg)
-  [万字解析微前端、微前端框架qiankun以及源码](https://mp.weixin.qq.com/s/_A5OfB4Xb3H_97r-bKlz0g) 
-  [深度：从零编写一个微前端框架](https://mp.weixin.qq.com/s/A0FR3h5Cvd0MWD5Gw_L19A) 
-  [几种微前端方案探究](https://vleedesigntheory.github.io/tech/front/micro20210125.htm)


## HTTP和HTTPS
-  请求头Connection: keep-alive的优点
   + 较少的CPU和内存的使用（由于同时打开的连接的减少了）
   + 允许请求和应答的HTTP管线化
   + 降低拥塞控制 （TCP连接减少了）
   + 减少了后续请求的延迟（无需再进行握手）
   + 报告错误无需关闭TCP连接
    
   ```js
   // 响应头开启 keep-alive
   Connection: Keep-Alive
   Keep-Alive: timeout=5, max=1000
   ```
-  浏览器强缓存和协商缓存
   ![来源于互联网](https://user-images.githubusercontent.com/25027560/38223505-d8ab53da-371d-11e8-9263-79814b6971a5.png)   
   HTTP缓存机制要点如下：
   + HTTP缓存机制分为强制缓存和协商缓存两类。
   + 强制缓存的意思就是不要问了(不发起请求)，直接用缓存吧。
   + 强制缓存常见技术有Expires和Cache-Control。
   + Expires的值是一个时间，表示这个时间前缓存都有效，都不需要发起请求。
   + Cache-Control有很多属性值，常用属性max-age设置了缓存有效的时间长度，单位为秒，这个时间没到，都不用发起请求。
   + immutable也是Cache-Control的一个属性，表示这个资源这辈子都不用再请求了，但是他兼容性不好，Cache-Control其他属性可以参考MDN的文档。
   + Cache-Control的max-age优先级比Expires高。
   + 协商缓存常见技术有ETag和Last-Modified。
   + ETag其实就是给资源算一个hash值或者版本号，对应的常用request header为If-None-Match。
   + Last-Modified其实就是加上资源修改的时间，对应的常用request header为If-Modified-Since，精度为秒。
   + ETag每次修改都会改变，而Last-Modified的精度只到秒，所以ETag更准确，优先级更高，但是需要计算，所以服务端开销更大。
   + 强制缓存和协商缓存都存在的情况下，先判断强制缓存是否生效，如果生效，不用发起请求，直接用缓存。如果强制缓存不生效再发起请求判断协商缓存。
-  content-type 为 `application/octet-stream`，代表二进制流，一般用以下载文件
-  XSS和CSRF理解
   + XSS：用户过分信任网站，放任来自浏览器地址栏代表的那个网站代码在自己本地任意执行。如果没有浏览器的安全机制限制，XSS代码可以在用户浏览器为所欲为；
   + CSRF：网站过分信任用户，放任来自所谓通过访问控制机制的代表合法用户的请求执行网站的某个特定功能。
-  XSS安全问题防范
   1. HttpOnly，https，Same-Site，HostOnly
   2. 输入特殊字符过滤 `<`,`>`等 和 检查 `HtmlEncode`、`JavaScriptEncode` 
-  CSRF安全问题防范
   1. Referer 源的检测
   2. 验证码
   3. token
   4. 设置SamesiteCookie为Strict,但是子域名也将无法共享你的cookie
-  [前端安全相关问题](https://mp.weixin.qq.com/s/tgIWXcfzswcKcKcb-Dzt_g)
-  Content Security Policy(CSP),只允许加载指定的脚本及样式，最大限度地防止 XSS 攻击，是解决 XSS 的最优解。
   ```js
   // 外部脚本可以通过指定域名来限制：Content-Security-Policy: script-src 'self'，self 代表只加载当前域名
   //Content-Security-Policy: script-src 'nonce-xxxxxxxxxxxxxxxxxx'
   ```
-  什么情况下会发送 OPTIONS 请求？  
   跨域且不是简单请求时就会发送 OPTIONS 请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json，CORS请求会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求
-  [网页性能提升5倍 — 网络优化篇](https://mp.weixin.qq.com/s?__biz=Mzg2NDAzMjE5NQ==&mid=2247487568&idx=1&sn=cb9334f34fed6da07c751bcc85a0059a&chksm=ce6ed4fcf9195dea0c835f22503a62997857d6d87ba429f25315314cf222d1a39908c0376d24&scene=178&cur_album_id=1403155327595495424#rd)

   
## Node.js和Deno.js
-  [《Deno 1.0 你需要了解的》](https://juejin.im/post/5eb8acec6fb9a043383d7610)
-  [了不起的 Deno 入门与实战](https://juejin.im/post/5ec24dea6fb9a04388076fba) 
-  [Umajs framework](https://github.com/wuba/Umajs)
-  [Eggjs](https://github.com/eggjs/egg)
-  [Nestjs](https://github.com/nestjs/nest)
-  [Darukjs](https://github.com/darukjs/daruk)



## 实用代码
-  [JavaScript实现base64编码解码](https://www.cnblogs.com/mofish/archive/2012/02/25/2367858.html)
-  电话号码隔位显示（344）
   ```js
    let str = '13800138000';
    let newStr = '';
    newStr = str.replace(/\s/g, '').replace(/(^\d{3})(?=\d)/g, "$1 ").replace(/(\d{4})(?=\d)/g, "$1 ");
    console.log(newStr);
   ```
-  Axios最佳实践封装
   ```js
   import axios from 'axios';
   import get from 'lodash/get';
   import storage from 'store';
   // 创建 axios 实例
   const request = axios.create({
   // API 请求的默认前缀
   baseURL: process.env.VUE_APP_BASE_URL,
   timeout: 10000, // 请求超时时间
   });

   // 异常拦截处理器
   const errorHandler = (error) => {
   const status = get(error, 'response.status');
   switch (status) {
      /* eslint-disable no-param-reassign */
      case 400: error.message = '请求错误'; break;
      case 401: error.message = '未授权，请登录'; break;
      case 403: error.message = '拒绝访问'; break;
      case 404: error.message = `请求地址出错: ${error.response.config.url}`; break;
      case 408: error.message = '请求超时'; break;
      case 500: error.message = '服务器内部错误'; break;
      case 501: error.message = '服务未实现'; break;
      case 502: error.message = '网关错误'; break;
      case 503: error.message = '服务不可用'; break;
      case 504: error.message = '网关超时'; break;
      case 505: error.message = 'HTTP版本不受支持'; break;
      default: break;
      /* eslint-disabled */
   }
   return Promise.reject(error);
   };

   // request interceptor
   request.interceptors.request.use((config) => {
   // 如果 token 存在
   // 让每个请求携带自定义 token 请根据实际情况自行修改
   // eslint-disable-next-line no-param-reassign
   config.headers.Authorization = `auth ${storage.get('ACCESS_TOKEN')}`;
   return config;
   }, errorHandler);

   // response interceptor
   request.interceptors.response.use((response) => {
   const dataAxios = response.data;
   // 这个状态码是和后端约定的
   const { code } = dataAxios;
   // 根据 code 进行判断
   if (code === undefined) {
      // 如果没有 code 代表这不是项目后端开发的接口
      return dataAxios;
   // eslint-disable-next-line no-else-return
   } else {
      // 有 code 代表这是一个后端接口 可以进行进一步的判断
      switch (code) {
      case 200:
         // [ 示例 ] code === 200 代表没有错误
         return dataAxios.data;
      case '3401':
         // [ 示例 ] 其它和后台约定的 code
         return '未登录';
      default:
         // 不是正确的 code
         return '错误的code';
      }
   }
   }, errorHandler);

   export default request;
   ```
-  常规校验
   ```js
   是否为手机号码： /^1[3|4|5|6|7|8][0-9]{9}$/
   是否全为数字： /^[0-9]+$/
   是否为邮箱：/^([A-Za-z0-9_\-\.])+\@([A-Za-z0-9_\-\.])+\.([A-Za-z]{2,4})$/
   是否为身份证：/^(^[1-9]\d{7}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}$)|(^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])((\d{4})|\d{3}[Xx])$)$/
   是否为Url：/[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)/
   是否为IP： /((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})(\.((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})){3}/
   ```
-  特殊字符转义之HTMLEncode
   ```js
   const HtmlEncode = (str) => {
      // 设置 16 进制编码，方便拼接
      const hex = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'];
      // 赋值需要转换的HTML
      const preescape = str;
      let escaped = "";
      for (let i = 0; i < preescape.length; i++) {
         // 获取每个位置上的字符
         let p = preescape.charAt(i);
         // 重新编码组装
         escaped = escaped + escapeCharx(p);
      }

      return escaped;
      // HTMLEncode 主要函数
      // original 为每次循环出来的字符
      function escapeCharx(original) {
         // 默认查到这个字符编码
         let found = true;
         // charCodeAt 获取 16 进制字符编码
         const thechar = original.charCodeAt(0);
         switch (thechar) {
               case 10: return "<br/>"; break; // 新的一行
               case 32: return "&nbsp;"; break; // space
               case 34: return "&quot;"; break; // "
               case 38: return "&amp;"; break; // &
               case 39: return "&#x27;"; break; // '
               case 47: return "&#x2F;"; break; // /
               case 60: return "&lt;"; break; // <
               case 62: return "&gt;"; break; // >
               case 198: return "&AElig;"; break; // Æ
               case 193: return "&Aacute;"; break; // Á
               case 194: return "&Acirc;"; break; // Â
               case 192: return "&Agrave;"; break; // À
               case 197: return "&Aring;"; break; // Å
               case 195: return "&Atilde;"; break; // Ã
               case 196: return "&Auml;"; break; // Ä
               case 199: return "&Ccedil;"; break; // Ç
               case 208: return "&ETH;"; break; // Ð
               case 201: return "&Eacute;"; break; // É
               case 202: return "&Ecirc;"; break;
               case 200: return "&Egrave;"; break;
               case 203: return "&Euml;"; break;
               case 205: return "&Iacute;"; break;
               case 206: return "&Icirc;"; break;
               case 204: return "&Igrave;"; break;
               case 207: return "&Iuml;"; break;
               case 209: return "&Ntilde;"; break;
               case 211: return "&Oacute;"; break;
               case 212: return "&Ocirc;"; break;
               case 210: return "&Ograve;"; break;
               case 216: return "&Oslash;"; break;
               case 213: return "&Otilde;"; break;
               case 214: return "&Ouml;"; break;
               case 222: return "&THORN;"; break;
               case 218: return "&Uacute;"; break;
               case 219: return "&Ucirc;"; break;
               case 217: return "&Ugrave;"; break;
               case 220: return "&Uuml;"; break;
               case 221: return "&Yacute;"; break;
               case 225: return "&aacute;"; break;
               case 226: return "&acirc;"; break;
               case 230: return "&aelig;"; break;
               case 224: return "&agrave;"; break;
               case 229: return "&aring;"; break;
               case 227: return "&atilde;"; break;
               case 228: return "&auml;"; break;
               case 231: return "&ccedil;"; break;
               case 233: return "&eacute;"; break;
               case 234: return "&ecirc;"; break;
               case 232: return "&egrave;"; break;
               case 240: return "&eth;"; break;
               case 235: return "&euml;"; break;
               case 237: return "&iacute;"; break;
               case 238: return "&icirc;"; break;
               case 236: return "&igrave;"; break;
               case 239: return "&iuml;"; break;
               case 241: return "&ntilde;"; break;
               case 243: return "&oacute;"; break;
               case 244: return "&ocirc;"; break;
               case 242: return "&ograve;"; break;
               case 248: return "&oslash;"; break;
               case 245: return "&otilde;"; break;
               case 246: return "&ouml;"; break;
               case 223: return "&szlig;"; break;
               case 254: return "&thorn;"; break;
               case 250: return "&uacute;"; break;
               case 251: return "&ucirc;"; break;
               case 249: return "&ugrave;"; break;
               case 252: return "&uuml;"; break;
               case 253: return "&yacute;"; break;
               case 255: return "&yuml;"; break;
               case 162: return "&cent;"; break;
               case '\r': break;
               default: found = false; break;
         }
         if (!found) {
               // 如果和上面内容不匹配且字符编码大于127的话，用unicode(非常严格模式)
               if (thechar > 127) {
                  let c = thechar;
                  let a4 = c % 16;
                  c = Math.floor(c / 16);
                  let a3 = c % 16;
                  c = Math.floor(c / 16);
                  let a2 = c % 16;
                  c = Math.floor(c / 16);
                  let a1 = c % 16;
                  return "&#x" + hex[a1] + hex[a2] + hex[a3] + hex[a4] + ";";
               } else {
                  return original;
               }
         }
      }
   }
   ```
-  特殊字符转义之JavaScriptEncode 
   ```js
   //使用“\”对特殊字符进行转义，除数字字母之外，小于127使用16进制“\xHH”的方式进行编码，大于用unicode（非常严格模式）。
   // 大部分代码和上面一样，我就不写注释了
   const JavaScriptEncode = function (str) {
      const hex = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'];
      const preescape = str;
      let escaped = "";
      for (let i = 0; i < preescape.length; i++) {
         escaped = escaped + encodeCharx(preescape.charAt(i));
      }
      return escaped;
      // 小于127转换成十六进制
      function changeTo16Hex(charCode) {
         return "\\x" + charCode.charCodeAt(0).toString(16);
      }
      function encodeCharx(original) {
         let found = true;
         const thecharchar = original.charAt(0);
         const thechar = original.charCodeAt(0);
         switch (thecharchar) {
               case '\n': return "\\n"; break; //newline
               case '\r': return "\\r"; break; //Carriage return
               case '\'': return "\\'"; break;
               case '"': return "\\\""; break;
               case '\&': return "\\&"; break;
               case '\\': return "\\\\"; break;
               case '\t': return "\\t"; break;
               case '\b': return "\\b"; break;
               case '\f': return "\\f"; break;
               case '/': return "\\x2F"; break;
               case '<': return "\\x3C"; break;
               case '>': return "\\x3E"; break;
               default: found = false; break;
         }
         if (!found) {
               if (thechar > 47 && thechar < 58) { //数字
                  return original;
               }
               if (thechar > 64 && thechar < 91) { //大写字母
                  return original;
               }
               if (thechar > 96 && thechar < 123) { //小写字母
                  return original;
               }
               if (thechar > 127) { //大于127用unicode
                  let c = thechar;
                  let a4 = c % 16;
                  c = Math.floor(c / 16);
                  let a3 = c % 16;
                  c = Math.floor(c / 16);
                  let a2 = c % 16;
                  c = Math.floor(c / 16);
                  let a1 = c % 16;
                  return "\\u" + hex[a1] + hex[a2] + hex[a3] + hex[a4] + "";
               } else {
                  return changeTo16Hex(original);
               }
         }
      }
   }
   ```
-  通过子节点递归找父节点
   ```js
   const findParent = (childrenId, arr, path) => {
    if (path === undefined) {
      path = []
    }
    for (let i = 0; i < arr.length; i++) {
      let tmpPath = path.concat();
      tmpPath.push(arr[i]);
      if (childrenId == arr[i].id) {
        return tmpPath
      }
      if (arr[i].children) {
        let findResult = findParent(childrenId, arr[i].children, tmpPath);
        if (findResult) {
          return findResult
        }
      }
    }
   }
   ```
-  compose函数
   ```js
   export default function compose(...funcs) {
     if (funcs.length === 0) {
       returnarg => arg
     }

     if (funcs.length === 1) {
       return funcs[0]
     }

     return funcs.reduce(function reducer(a, b) {
       return function (...args) {
         return a(b(...args))
       }
     })
   }
   // 函数依次从右向左执行
   ```



## 自动化探索
-  [Puppeteer 爬虫实践](https://mp.weixin.qq.com/s?__biz=MzI0NTE5NzYyMw==&mid=2247483769&idx=1&sn=f4d65fe8b60d8870b0a243ad5da48048&scene=21)
-  [自动化构建部署工具](https://juejin.cn/post/6916302771129942030)
-  [前端轻量化部署脚手架](https://juejin.cn/post/6844904046986280967)
-  [小工具实现更新本地iconfont](https://github.com/xianzhiyun/iconfont-script)
-  [微信小程序工程化之持续集成方案](https://zhuanlan.zhihu.com/p/89055847)
-  [小程序多端编译原理](https://juejin.cn/post/6844904094981685262)
-  [Electron + Puppeteer + Robotjs 实现工作自动化](https://zhuanlan.zhihu.com/p/197737856)
-  [VuePress + Travis CI + Github Pages 全自动上线文档](https://juejin.cn/post/6844903869558816781)



## 在线电子书
-  [深入理解 TypeScript](https://jkchao.github.io/typescript-book-chinese/)
-  [Flutter完整开发实战详解](https://guoshuyu.cn/home/wx/)



## 工具/网站
-  [Storybook](https://storybook.js.org/) - 一个高效有序地构建炫酷用户界面，方便管理和测试组件的开源工具，适用于 React, Vue, 和 Angular。
-  [cypress](https://www.cypress.io/) - 更适合于前端工程师使用的自动化测试工具
-  [Vue Datav](http://datav.jiaminghi.com/) - Vue 大屏数据展示组件库
-  [可视化Javascript底层运行](http://latentflip.com/loupe/)
-  [React hooks library](https://ahooks.js.org/zh-CN)
-  [SWR - React Hooks library for data fetching](https://swr.vercel.app/)
-  [form generator表单设计器](https://github.com/JakHuang/form-generator-plugin)
-  [docz文档生成器](https://www.docz.site/)
-  [verdaccio私库](https://verdaccio.org/docs/zh-CN)
-  [Carbon美化代码片段](https://carbon.now.sh/)
-  [SwitchHosts host管理工具](https://github.com/oldj/SwitchHosts)


## 规范工具
-  [Standardjs](https://standardjs.com/readme-zhcn.html)
-  [Airbnb-中文版](https://github.com/lin-123/javascript)
-  [Stylelint](https://stylelint.io/)
-  [ESLint](https://eslint.org/)
-  [Commitlint](https://github.com/conventional-changelog/commitlint)
-  git commit之 husky，lint-staged
   ```js
   "gitHooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint -E GIT_PARAMS"
   },
   "lint-staged": {
      "*.js": [
         "vue-cli-service lint"
      ],
      "*.vue": [
         "vue-cli-service lint"
      ]
   }
   ```


## 前端监控
-  [Sentry 部署应用实践](https://segmentfault.com/a/1190000021602782)



## 脚手架
-  [如何搭建一个成熟的脚手架](https://github.com/yokiyokiyoki/ds-cli)
-  [走进Vue-cli源码](https://www.jianshu.com/p/749b22170b7b)
-  [Vue CLI 是如何实现的](https://juejin.cn/post/6916303253487484942)
-  [vue-webpack-boilerplate 通用项目工程](https://github.com/monw3c/vue-wp-cli)
-  开发脚手架需要的包：
   + 脚手架工具： [plop](https://plopjs.com/)、[Yeoman](https://yeoman.io/)
   + 交互输入：[inquirer](http://npm.im/inquirer)、 [enquirer](http://npm.im/enquirer) 、[prompts](https://npm.im/prompts)
   + 版本检测：[semver](https://github.com/npm/node-semver)
   + 输出美化：[chalk](http://npm.im/chalk)、 [ink](http://npm.im/ink)、[log-symbols着色的符号](https://www.npmjs.com/package/log-symbols)
   + 加载动画：[ora](http://npm.im/ora)
   + 基本解析：[meow](http://npm.im/meow)、 [arg](http://npm.im/arg)
   + 通知插件：[node-notifier](https://www.npmjs.com/package/node-notifier)
   + 参数解析：[commander](http://npm.im/commander)、[yargs](https://www.npmjs.com/package/yargs)
   + 操作命令行：[sindresorhus/ansi-escapes](https://github.com/sindresorhus/ansi-escapes)
   + 输出截断： [sindresorhus/cli-truncate](https://github.com/sindresorhus/cli-truncate)
   + 下载git模板： [download-git-repo](https://www.npmjs.com/package/download-git-repo)
   + Vue-cli使用，计算字符串编辑距离算法：[leven](https://github.com/sindresorhus/leven)
   + 开启子线程：node.js自带的child_process


## Shell命令
-  tree  
   列出目录结构,更多功能查看[文档](https://wangchujiang.com/linux-command/c/tree.html)
   ```bash
   $ tree -L 2 -I 'node_modules' # 目录结构层级为2，忽略node_modules
   ├── app.js
   ├── dist
   ├── f.yml
   ├── package.json
   └── src
      ├── detail
      ├── index
      └── layout
   ```

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

