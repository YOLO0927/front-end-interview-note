* 简短介绍 AMD、CMD、commonJs、ES6 模块系统，以及说明它们之间的区别
  1. AMD：异步模块加载机制，典型代表 require.js ，推崇依赖前置，先将所需依赖全部定义，然后在依赖加载完毕的回调中使用它，即还没开始使用就需要先定义好，浏览器会先加载完所有依赖再执行代码；
  2. CMD：与 AMD 一样都是异步模块加载机制，典型代表 sea.js ，推崇就近依赖，即依赖懒加载，在代码需要时在加载，与 AMD 的不同的是不论你代码如何写，AMD都是先加载依赖再执行编码，无论你位置放在哪，即使你是将依赖的加载位于代码之后，那么也是先预先加载依赖才执行代码，而 CMD 则是懒加载，你将依赖放在哪加载就是哪加载，是按照代码顺序依次执行代码或加载依赖的

      若仍不清楚两者区别请去查看相应详细的例子解析 https://www.douban.com/note/283566440/
  3. commonJs：NodeJs 的模块机制，使用 require 加载依赖，由于在 node 中都是从硬盘中直接加载依赖的，所以硬盘的读写速度就是依赖的加载速度，所以你无论怎么写都不会阻塞太久，因为服务端硬盘的读写是很快的，而且每加载一次后就会自动将模块进行缓存到内存中，而浏览器端依赖的加载是吃网速的，如果网速不好就会使页面长时间出现白屏或业务假死的情况，所以浏览器端不能像 commonjs 一样同步加载依赖。
  4. ES6的模块机制：与 AMD CMD 一样是作用与浏览器的 异步模块加载机制 ，只是与 AMD CMD 的写法不一样了，使用 import exports module.exports 来进行引入与输出，需要注意的是 exports 出来的是一个默认对象，会被 module.exports 覆盖，所以大部分框架中一般都要将 module.exports 指向回 exports 以免使用者错误引用或方便其使用对象扩展符局部引用。

    异同：commonJs 是 node 的规范，是同步加载依赖的，AMD、CMD、ES模块都是浏览器侧异步加载模块的，而 ES6 的写法与 AMD、CMD 写法相差很多但简化了引入的代码逻辑，AMD是依赖前置加载，CMD是依赖就近顺序加载。

* 网易业务 API 设计题：加入A，B，C模块需要用户信息，让你设计一个API去实现这个用户信息获取的公共方法，不能出现重复请求，（比如A来获取时，本地没有，需要向服务器发请求，这个请求发送过程中B又来请求）考察任务队列

  本题考察创建任务队列与闭包 => 我的答案：
  ```js
  var getHttpUser = function (url) {
    var resStacks = [], userInfo, httpLock = false

    function outputStacks () {
      resStacks.forEach(function (cb) {
        cb(Object.assign({}, userInfo))
      })
      resStacks = []
    }

    return function (cb) {
      resStacks.push(cb)
      if (userInfo) {
        outputStacks()
      } else {
        if (!httpLock) {
          httpLock = true
          http.get(url).then(function (data) {
            userInfo = data
            outputStacks()
          })
        }
      }
    }
  }

  var getUser = getHttpUser('http://127.0.0.1/getUserInfo')

  module.exports = getUser

  // 调用此模块后 getUser(function (data) { console.log(data) })
  ```

* 单页路由应用 Router 的实现机制？
  1. 基于 hashchange 事件的 url 监听，根据路由 change 时提取 url 后通过正则过滤路由深度跳转指定模版，简单实现可参考我博客这篇demo [教你50行代码实现前端路由小轮子](https://mp.csdn.net/mdeditor/78076473#)；
  2. 基于 H5 History 对象的实现操作，通过 `pushState(state, name, url)`，`replaceState``，popState` 实现指定跳转、替换与出栈跳回 3 种操作，监听 popstate 事件的前进后退等事件，由函数中的 `event.state` 可获取事件传递的状态参数来对应修改指定状态，由于 `pushState`、`popState`、`replaceState`没有对应事件监听，所以我们可以通过简单劫持来实现监听与模版的替换，例如
  ```js
    // html
    <a>测试</a>
    // js
    (function(history) {
      history.push = function (state, name, url) {
        console.log(`history pushState 了，替换成 ${name} 模版`)
        return history.pushState.apply(history, arguments)
      }
    })(window.history)

    document.querySelector('a').addEventListener('click', function () {
      history.push({color: '#bfc'}, 'test', '/test/123')
    })
    window.onpopstate = function (e) {
      console.log(e)
    }
  ```
  结果大家可自行测试查看

* 说一下原型链，对象，构造函数之间的一些联系(接下来是我自己的理解，并从这几个点扩展根据个人理解说即可)
  1. 构造函数：首先因为 js 没有类的概念，所以在 js 内我们可以将构造函数当作创建类的定义函数，并在构造函数中使用 this 指向后续实例化对象来以此提供后续类的操作，在 ES6 中将构造函数规划为了类的概念，由此 js 也由类了，用 class 来创建类，在 ES5 类函数本身就是 constructor，即构造实体、构造函数，在 ES6 将此属性单独分离，并引入 super 作为超集引入 this 作为指针引用，简单来说构造函数就是用于创建实体的类定义函数；
  2. 原型链：构造函数中我们可在其中将所需的函数操作写入原型，以此利用实例化来继承原型链上的方法，而一旦继承后就会产生链式关系，由于 ES5 中没有正统继承的概念，所以我们一般以原型实例化来实现继承操作，比如 ES5 原型继承即使赋值实例化对象入原型 `Class1.prototype = new Class2()`，由此 Class1 继承了 Class2 ，Class2 的实例化函数全部被注入 Class1 的原型中，由此便产生了一条简单的原型链，我们通过
  ```js
    var obj = new Class1()
    console.log(obj.__proto__)
  ```
  可查看到 Class1 的实体的父链为 Class2，这就是最简单的由继承所产生的原型链，而原型链的顶端即为一个具有 constructor 的对象，再向上即为 null，简而言之即是由继承产生的链式原型。


* DOM事件中 target 和 currentTarget 的区别

  答：event.target 是指触发事件的元素目标，event.currentTarget 是指当前正在处理事件的元素（你绑定的元素），简单来说就是当嵌套 div 时，点击事件同时注册多个 div，外部 div 会接收到内部 div 通过事件冒泡上来的事件，此时内部触发这次事件的 div 就是 target，而你使用外部 div 接收事件做处理时这个外部 div 就是 currentTarget，记住触发元素是 target，监听元素是 currentTarget 即可;

* 说一下深拷贝的实现原理

  答：就是简单的递归函数，函数内循环遍历拷贝对象的类型，如果为数组或对象这2个地址引用的类型就递归调用创建新的对象或数组遍历赋值，由此递归循环，下面是我的实现 demo
  ```js
  var deepClone = function (target) {
      var judgeType = Object.prototype.toString
      var newObj = new Object(), newArr = new Array(), newTarget

      if (judgeType.call(target) === '[object Object]') {
        newTarget = newObj
        for (var key in target) {
          if (judgeType.call(target[key]) === '[object Object]' || '[object Array]') {
            newTarget[key] = deepClone(target[key])
          } else {
            newTarget[key] = target[key]
          }
        }
      } else if (judgeType.call(target) === '[object Array]') {
        newTarget = newArr
        for (var i = 0; i < target.length; i++) {
          if (judgeType.call(target[i]) === '[object Object]' || '[object Array]') {
            newTarget[i] = deepClone(target[i])
          } else {
            newTarget[i] = target[i]
          }
        }
      } else {
        return newTarget = target
      }
      return newTarget
    }
  ```

* 白板写代码，用最简洁的代码实现数组去重
  1. ES6: `[...new Set([1, 2, 3, 4, 1, 2, 11, 11, 42, 4])]`
  2. ES5:
  ```js
    var arr = [1, 2, 3, 4, 1, 2, 11, 11, 42, 4], obj = {}
    for (var i = 0; i < arr.length; i++) {
      if (obj[arr[i]]) continue
      obj[arr[i]] = arr[i]
    }
    Object.values(obj)
  ```

* Promise 原理及规范
  1. Promise/A+ 规范
    * 一个 Promise 必须处于 3 个状态中的其中一个

      pending（可转换 fulfilled 或 rejected）

      fulfilled（不能转换为其他状态）

      rejected（不能转换为其他状态）
    * thenable then 是可以串行的链式调用，且具有 onFulfilled 和 onRejected 为可选参数
    * 错误时必须有抛出异常的原因（reason）

  2. 源码实现
  ```js
  var Promise = function (fn) {
    var deffereds = [],
        value = null,
        state = 'pending'

    var resolve = function (newValue) {
      // 2. 引入状态判断与变换
      if (state === 'rejected') return ;
      if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
        var then = newValue.then
        if (typeof then === 'function') {
          then.call(newValue, resolve, reject)
          return
        }
      }
      value = newValue
      state = 'fulfilled'
      finale()
    }

    function reject (reason) {
      // 2. 引入状态判断与变换
      if (state === 'fulfilled') return ;
      state = 'rejected'
      value = reason
      finale()
    }

    // 3. 串行链式调用
    this.then = function (onFulfilled, onRejected) {
      return new Promise((resolve, reject) => {
        handle({
          onFulfilled: onFulfilled || null,
          onRejected: onRejected || null,
          resolve: resolve,
          reject: reject
        })
      })
    }

    // 4. 异常错误处理
    this.catch = function (errFn) {
      if (state === 'rejected') errFn(value)
    }

    // 1. 延时异步实现
    function finale () {
      setTimeout(() => {
        deffereds.forEach(deffered => {
          handle(deffered)
        })
      }, 0)
    }

    function handle (deffered) {
      // 2. 引入状态判断与变换
      if (state === 'pending') {
        deffereds.push(deffered)
        return ;
      }
      var cb = state === 'fulfilled' ? deffered.onFulfilled : deffered.onRejected
      // 当出错并且没有传入 onRejected 时，直接执行 reject 方法将 state 变为 rejected 状态
      // 此时如果是 resolve 时代码出错时我们便需要监测到并且手动执行 reject 犯法，所以下面我们要使用 try catch 抓取此情况
      if (cb === null) {
        cb = state === 'fulfilled' ? deffered.resolve : deffered.reject
        cb(value)
        return ;
      }
      // 4. 异常错误处理
      try {
        // 如若执行 promise.resolve 时出错这里会抓取异常并执行 reject 方法将异常发出去
        var ret = cb(value)
        deffered.resolve(ret)
      } catch (err) {
        deffered.reject(err)
      }
    }

    fn(resolve, reject)
  }
  ```
  详细过程可参考美团技术博客 https://tech.meituan.com/promise_insight.html

* 简单说明事件委托

  事件委托指利用 dom 事件冒泡的原理，使子元素的事件冒泡到父级元素时，由父级监听并处理的过程，只需在父级监听对应子级事件并判断当前触发的节点是否为子级并执行对应业务操作即可，由此可以达到只监听一个元素与节点便可处理每个指定节点的对应的操作。 下面是简单实现
  ```html
    <style>
    .box{
        width: 100px;
        height: 100px;
        background-color: #bfc;
        display: inline-block;
      }
    </style>
    <button id="add" type="button" name="button">添加</button>
    <button id="remove" type="button" name="button">删除</button>
    <div class="box-parent"></div>
    <script type="text/javascript">
      var parent = document.getElementsByClassName('box-parent')[0]
      document.getElementById('add').addEventListener('click', function (e) {
        var node = document.createElement('span')
        node.className = 'box'
        node.setAttribute('data-index', parent.childElementCount + 1)
        parent.appendChild(node)
      })
      document.getElementById('remove').addEventListener('click', function (e) {
        parent.removeChild(parent.lastElementChild)
      })
      parent.addEventListener('click', function (e) {
        if (e.target.nodeName === 'SPAN') {
          console.log(e.target.getAttribute('data-index'))
        }
      })
    </script>
  ```

* 说一下箭头函数This指向问题

  箭头函数实际上就是一个普通函数 bind(this) 之后的语法糖，使函数中的 this 被绑定在当前地址域中而不会被干扰改变指向（如定时器内），例如
  ```js
    var obj = {
      name: 'yolo',
      interval: function () {
        setTimeout(function () {
          console.log(this)
        })
      }
    }
    obj.interval()  // window

    // 改造
    obj.interval = function () {
      setTimeout((function () {
        console.log(this)
      }).bind(this))
    }
    obj.interval()  // obj
  ```

* for in 与 for of 的区别
  1. **for of** 可以遍历一切具有迭代器的对象，如 数组、字符串，都具有，在其原型链中查看是否具有 `Symbol.iterator` 即可，而对象是没有遍历接口的，即没有迭代器，如果需要可以利用 generator 写一个携带遍历器的对象即可，此外当使用 for of 时不需要再依次调用 next 方法获取，而是直接返回生成器分段返回的结果，且 for of 不会遍历到目标的原型，下面可利用 generator 将对象变为存在遍历器使其满足 for of
  ```js
    var obj = {
      name: 'yolo',
      age: '25',
      gender: 'male'
    }

    obj[Symbol.iterator] = function* () {
      for (let key in obj) {
        yield [obj[key], key]
      }
    }

    for (let item of obj) {
      // [ 'yolo', 'name' ]
      // [ '25', 'age' ]
      // [ 'male', 'gender' ]
      console.log(item)
    }
  ```

  2. **for in** 没什么好说的，遍历过程中不保证顺序的遍历方式，且遍历出的 key 会隐式转换为字符串并会遍历到对象原型中的可修改属性，所以我们一般只用其遍历对象，因为如果遍历普通数组，不但无法保证顺序，而且会使下标变为字符串而无法进行计算，除非你自己处理
  ```js
    Object.prototype.c = 1
    var obj = {a: 1, b: 2}
    for (var key in obj) {
      // a
      // b
      // c
      console.log(key)
    }
  ```
* 说一下你对generator的了解
  generator 实际上是一个具有分段返回能力的函数，可用于处理多个异步操作来实现分段返回，执行 generator 函数会返回一个遍历器对象，我们可以依次调用这个对象的 next 方法去使其内部协程依次执行，但是一定要注意的是同一事件循环线程下不嫩头同时跑 2 个 生成器的 next，否则会报错已经在 running
  ```js
    function* Test () {
      yield 'hello'
      yield 'world'
      return 'ending'
    }
    var test = Test()
    let test1 = test.next() // {value: "hello", done: false}
    test.next() // {value: "world", done: false}
    test.next() // {value: "ending", done: true}
  ```
  更具体的用法请查看 http://es6.ruanyifeng.com/#docs/generator

* event loop（js 的事件循环机制）
  搞清楚 2 个就行，哪些是属于 macro task 与 micro task 的任务，明白一个队列中是 micro task 先进行，然后才到 macro task 再 micro task 如此循环

  事件循环的顺序：决定了JavaScript代码的执行顺序。它从script(整体代码)开始第一次循环。之后全局上下文进入函数调用栈。直到调用栈清空(只剩全局)，然后执行所有的micro-task。当所有可执行的micro-task执行完毕之后。循环再次从macro-task开始，找到其中一个任务队列执行完毕，然后再执行所有的micro-task，这样一直循环下去

  **micro task**：process.nextTick, Promise.then, Object.observe(已废弃), MutationObserver(H5)

  **macro task**：script(整体代码), setTimeout, setInterval, setImmediate, I/O, UI rendering

  正确理解后的测试题
  ```js
    console.log('a');
    setTimeout(() => {
      console.log('b');
    }, 0);
    console.log('c');
    Promise.resolve().then(() => {
      console.log('d');
    })
    .then(() => {
      console.log('e');
    });

    console.log('f');
  ```
  正确的输出顺序为 acfdeb

  更加详细的解析 https://yangbo5207.github.io/wutongluo/ji-chu-jin-jie-xi-lie/shi-er-3001-shi-jian-xun-huan-ji-zhi.html

* 介绍自己写过的中间件

  由于 路由、静态文件挂载、session 等都使用开源插件作为中间件处理了，所以自己大部分写的是业务中间件，简单说 2 个
  1. 用户是否站内（即是否登录）
    * 首先获取请求携带的用户信息态 cookie，过滤出用户id，去与存储在 session 中的用户id做比较，查看是否相同，若相同则代表已登录过并仍处于缓存保留期间，若 session 中不存在或 cookie 过期则直接返回未登录状态码且前端调起登录页面。
  2. 校验活动是否过期
    * 直接获取中间件传入活动名的参数，找到 config 中对应活动的配置，查看当前日期是否符合活动配置内所定日期

* 关于 Cluster 集群的多线程服务

  首先必须了解如何利用 tcp 与进程间通信完成集群部署服务（master-worker）
    1. 从主进程创建 TCP 服务，使用 `child_process` 模块 fork 复制多个子进程，一般按照 cpu 数量 fork（原因是 js 是单进程运算，所以最多只能使用一个核）；
    2. 向子进程发送tcp句柄，`work.send(message, handle)`，handle 为第一步通过 net 模块创建 tcp 服务并监听端口后返回的 net.server；
    3. 子进程由 http 或 https 模块创建对应服务，但不需监听端口，并且子进程监听 message 事件，由主进程发送的信息 message 判断当前是否为主进程发出 tcp 句柄时的信息，接受 `net.server` 后监听 connection 事件，当主进程 master 监听到创建连接时响应到子进程 tcp 句柄的 connection 事件中，最后在此调用之前创建的 http 或 https 服务 emit('connection', socket) 触发服务的 request 事件，每次 curl 当前 tcp 服务的端口及 host 时都会触发 connection 事件的主从传递，由此抢占式分发到不同的子进程去处理请求。

  而 Cluster 模块就式简洁的做了这件事，我们直接使用 `cluster.isMaster` 判断当前进程的 cluster 是否为主进程，如果是则去 `cluster.fork` 多个子进程，若不是则直接创建 http 或 https 服务并监听所需端口，此时 cluster 模块创建的子进程会使用上述的集群部署的原理去自动做这件事，此时的 http 服务也是通过 tcp 句柄传递过去接收触发的，由于是同一句柄，所以是同一描述符允许共享端口

  详细的 demo 可查看我的博客 https://blog.csdn.net/yolo0927/article/details/81224942

* 数组的扁平化、去重、排序

  排序你可以自己写，也可以直接用 `Array.prototype.sort(cb)` 去自定义，大家自己决定，排序我就写个简单的冒泡吧，突然之间你让我写完所有也不可能啦，所以我就写个最简单的
   ```js
   var arr = [[1, 2, 2], [3, 4, 5, 5], [6, 7, 8, 9, [11, 12, [12, 13, [14]]]], 10]
   var flatten = function (arr) {
     var newArr = []
     for (var i = 0; i < arr.length; i++) {
       if (Array.isArray(arr[i])){
         newArr = newArr.concat(flatten(arr[i]))
       } else {
         newArr.push(arr[i])
       }
     }
     return newArr
   }

   var up = function (arr) {
     for (var i = 0; i < arr.length; i++) {
       for (var j = i + 1; j < arr.length; j++) {
         if (arr[i] > arr[j]) {
           var num = arr[i]
           arr[i] = arr[j]
           arr[j] = num
         }
       }
     }
     return arr
   }
   console.log(up([...new Set(flatten(arr))]))
   ```
