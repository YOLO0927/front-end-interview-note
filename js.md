* 简短介绍 AMD、CMD、commonJs、ES6 模块系统，以及说明它们之间的区别
  1. AMD：异步模块加载机制，典型代表 require.js ，推崇依赖前置，先将所需依赖全部定义，然后在依赖加载完毕的回调中使用它，即还没开始使用就需要先定义好，浏览器会先加载完所有依赖再执行代码；
  2. CMD：与 AMD 一样都是异步模块加载机制，典型代表 sea.js ，推崇就近依赖，即依赖懒加载，在代码需要时在加载，与 AMD 的不同的是不论你代码如何写，AMD都是先加载依赖再执行编码，无论你位置放在哪，即使你是将依赖的加载位于代码之后，那么也是先预先加载依赖才执行代码，而 CMD 则是懒加载，你将依赖放在哪加载就是哪加载，是按照代码顺序依次执行代码或加载依赖的

    若仍不清楚两者区别请去查看相应详细的例子解析 https://www.douban.com/note/283566440/
  3. commonJs：NodeJs 的模块机制，使用 require 加载依赖，由于在 node 中都是从硬盘中直接加载依赖的，所以硬盘的读写速度就是依赖的加载速度，所以你无论怎么写都不会阻塞太久，因为服务端硬盘的读写是很快的，而且每加载一次后就会自动将模块进行缓存到内存中，而浏览器端依赖的加载是吃网速的，如果网速不好就会使页面长时间出现白屏或业务假死的情况，所以浏览器端不能像 commonjs 一样同步加载依赖。
  4. ES6的模块机制：与 AMD CMD 一样是作用与浏览器的 异步模块加载机制 ，只是与 AMD CMD 的写法不一样了，使用 import exports module.exports 来进行引入与输出，需要注意的是 exports 出来的是一个默认对象，会被 module.exports 覆盖，所以大部分框架中一般都要将 module.exports 指向回 exports 以免使用者错误引用或方便其使用对象扩展符局部引用。

    异同：commonJs 是 node 的规范，是同步加载依赖的，AMD、CMD、ES模块都是浏览器侧异步加载模块的，而 ES6 的写法与 AMD、CMD 写法相差很多但简化了引入的代码逻辑，AMD是依赖前置加载，CMD是依赖就近顺序加载。

* 网易业务 API 设计题：加入A，B，C模块需要用户信息，让你设计一个API去实现这个用户信息获取的公共方法，不能出现重复请求，（比如A来获取时，本地没有，需要向服务器发请求，这个请求发送过程中B又来请求）考察任务队列

  本题考察创建任务队列与闭包 => 我的答案：
  ```
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
  ```
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
  ```
    var obj = new Class1()
    console.log(obj.__proto__)
  ```
  可查看到 Class1 的实体的父链为 Class2，这就是最简单的由继承所产生的原型链，而原型链的顶端即为一个具有 constructor 的对象，再向上即为 null，简而言之即是由继承产生的链式原型。


* DOM事件中 target 和 currentTarget 的区别

  答：event.target 是指触发事件的元素目标，event.currentTarget 是指当前正在处理事件的元素（你绑定的元素），简单来说就是当嵌套 div 时，点击事件同时注册多个 div，外部 div 会接收到内部 div 通过事件冒泡上来的事件，此时内部触发这次事件的 div 就是 target，而你使用外部 div 接收事件做处理时这个外部 div 就是 currentTarget，记住触发元素是 target，监听元素是 currentTarget 即可;

* 说一下深拷贝的实现原理

  答：就是简单的递归函数，函数内循环遍历拷贝对象的类型，如果为数组或对象这2个地址引用的类型就递归调用创建新的对象或数组遍历赋值，由此递归循环，下面是我的实现 demo
  ```
  var deepClone = function (target) {
    var judgeType = Object.prototype.toString
    var newObj = new Object(), newArr = new Array(), newTarget

    if (judgeType.call(target) === '[object Object]') {
      newTarget = newObj
    } else if (judgeType.call(target) === '[object Array]') {
      newTarget = newArr
    } else {
      return newTarget = target
    }

    for (key in target) {
      if (judgeType.call(target[key]) === '[object Object]' || '[object Array]') {
        newTarget[key] = deepClone(target[key])
      } else {
        newTarget[key] = target[key]
      }
    }
    return newTarget
  }
  ```

* 白板写代码，用最简洁的代码实现数组去重
  1. ES6: `[...new Set([1, 2, 3, 4, 1, 2, 11, 11, 42, 4])]`
  2. ES5:
  ```
    var arr = [1, 2, 3, 4, 1, 2, 11, 11, 42, 4], obj = {}
    for (var i = 0; i < arr.length; i++) {
      if (obj[arr[i]]) continue
      obj[arr[i]] = arr[i]
    }
    Object.values(obj)
  ```
