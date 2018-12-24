* Vue是如何实现双向绑定的
 1. 通过 Object.defineProperty() 循环绑定定义 data 对象；
 2. 通过定义 getter 访问器将 data 中的属性 push 入 watch 队列中监听在实例化 Vue 对象时，拿到 el 绑定的目标元素，将目标 DOM 树劫持下来并通过遍历 DOM 树与正则取出与 data 对象总相应的 key 值，将 data 中 key 对应的 value 赋值上去再重构 dom 树，并在赋值取值时触发 getter 从而将每个在 dom 中出现的属性压栈进 watch 队列，实现 model => view 单向绑定的过程；
 3. 通过 Object.defineProperty() 中定义 data 中属性的 setter，每次监听到 view 中对应属性改变时触发 setter ，通知 watch 队列更新 model 并重新更新 view。

  整体来说，通过定义 getter 访问器实现劫持 dom 树并赋值 dom 中对应双向绑定的变量重构 dom 树，在重构赋值时触发 getter 将 node push 到 watch 队列，后续在改变 dom 中的值时触发 setter 从而触发 watch 队列的遍历更新 view，如此反复进行。

  最简单的例子
  ```html
  <body>
    <input type="text" id="input" value="">
    <div id="input-display"></div>
  </body>
  <script type="text/javascript">
    var input = document.getElementById('input'),
        display = document.getElementById('input-display')
    var data = {
      test: 'hello world'
    }
    var obj = {}
    Object.defineProperty(obj, 'data', {
      get () {
        return data.test
      },
      set (val) {
        input.value = val
        display.innerHTML = input.value
        data.test = val
      }
    })
    obj.data = 123
    input.oninput = function (e) {
      obj.data = e.target.value
    }
  </script>
  ```


  详细过程可查看我博客早期的一篇文章 https://blog.csdn.net/yolo0927/article/details/53789075

  而 React 是单向绑定的，如若想实现双向绑定，则基于事件更新 state 触发更新 view 即可，所以说 React 是基于事件的双向绑定，而 vue 是通过劫持 dom 定义属性访问器而实现的双向绑定。

* 简单比较 Vue 与 React 的差异，说说对应的优点与缺点
  1. react 是完全基于视图的单向绑定，通过 AST 抽象生成树来实现 Virtual DOM 到真实 DOM 的更新，通过 diff 虚拟 dom 树，使其能够局部更新对应的真实 DOM，是基于事件的双向绑定，通过元素如点击事件等响应改变状态从而触发 diff；
  2. Vue 是基于劫持 dom 并重构以及 defineProperty 定义访问器所实现的双向绑定，通过触发属性 setter 访问器来实现 watch 队列的遍历更新视图；
  3. Vue 推崇模版写法，所以许多操作 Vue 已经代替开发者处理了，导致整体的 api 较多，而自由度相对没有这么高，比如有 计算属性 computed、通过 setter 监听的 watch、当没有在 data 中定义属性而后续加入双向绑定时使用 set 方法以及为了实现数组双向绑定而重写了 data 中数组的 push、pop、slice 等方法；
  4. 相比之下 React 则较自由，更考量开发者的 js 功底，基于事件的更新 state 触发 diff 算法而更新视图、由于对象或数组地址未变化而导致视图无法 diff 时通过合并为新对象或数组触发等，整体的开发思路与 Vue 有些许差异，但是纯 js 的写法会导致如果组件颗粒度不够细或抽象的不够好时，整个代码看起来会比较臃肿（因为可能会看起来一大块一大块的，这个因人而异），以及 css in js 方案对我来说有点难看= =。。。
  5. 但两者都做到了前端的组件化，所以个人认为没有优劣，还是要看项目的大小，如果是中大型项目且开发者水平都中上，对 js 比较熟悉，那么后续将组件抽象化程度以及颗粒度变细后，会极大发挥 react 自由度的优势，相比 vue 更好维护点

    而在中小型项目时，组件复用性并没有这么强的情况下，vue 快速开发的优势就会体现出来，并且是模版开发的模式，对开发者的水平要求没有这么高，因为有大量的 api 给予开发者使用，省去了开发者的学习成本，所以需要快速开发的中小型项目个人认为 vue 更有优势，还有最重要的一点是要看团队整体水平及方向来决定。
