* 两种以上方式实现已知或者未知宽度的垂直水平居中
  1. 利用父级相对定位，子级绝对定位后，水平垂直都移动 50% 后，再通过 transform 相对自身 translate 各 50%

    ```css
    .parent {
      position: relative;
      width: 5rem;
      height: 5rem;
    }

    .child {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
    ```
  2. 利用弹性布局，调整父级水平与垂直属性的居中

    ```css
    .parent {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 100%;
      height: 5rem;
    }

    .child {
      flex: 1;
      height: 2rem;
    }
    ```
  3. 在第 1 点的基础上，不使用 translate 来反移自身，而是使用 margin 负值来实现自身的回移一半

    ```css
    .parent {
      position: relative;
      width: 5rem;
      height: 5rem;
    }

    .child {
      position: absolute;
      top: 50%;
      left: 50%;
      width: 2rem;
      height: 2rem;
      margin: -1rem 0 0 -1rem;
    }
    ```

  极其不推荐第 3 种，因为需要定死自身元素的宽高来进行回移，一半情况下，我们会令元素内容自适应，而无法得知自身的动态宽高，这无疑极其的限制了布局上的适应性，而 transform 中的 translate 就是以自身为单位去计算百分比的，极大的增加了灵活性，所以我们一般不推崇第 3 种方式。

* 移动端适配1px的问题
  * 原因：移动端会产生 1px 细线增大的原因就是因为现在大部分手机已经是高清屏了，但是我们在开发时会将视口内容限定为设备宽，也就是
  ```html
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  ```
  在移动端我们可通过 BOM `window.devicePixelRatio` 获取设备的像素比，而 iphone6/7/8 的像素比为 2，即在此设备内 1px 反应出来实际是肉眼的 2 物理像素，即 1px => 2px，所以才出现了移动端需要适配 1px 的问题，由此产生的字体或元素宽高变化我们使用 rem 布局即可解决

  * 解决方法
    1. 使用 backgorund 渐变来解决，具体则是由透明渐变到指定颜色即可，0-0.5 时为透明，0.5-1 时为指定颜色到指定颜色即可

    ```css
    @media screen and (-webkit-min-device-pixel-ratio: 2){
      .ui-border {
          background-image: -webkit-linear-gradient(180deg, transparent 0%, transparent 50%, gray 100%;)
      }
    }
    ```
    2. 使用伪类 + transform scale，这种方法很简单，就是在所需元素上加 before 或 after 伪类，通过设置伪类元素的宽高或边界，然后使用`transform: scale(0.5)` 缩小一倍即可
    3. 用 border-image 使用图片来代替像素的变更，因为图片是不会缩放的= =。。。这个方法个人不推荐，因为不同情况，你得准备多少图片啊。。。

* 改变哪些 css 属性会导致重绘或回流
  * 添加、删除元素（回流 + 重绘）;
  * 隐藏元素，`display: none（回流 + 重绘）`、`visibility: hidden（重绘，不回流）`;
  * 移动元素，改变 top、left、移动元素到另外的父元素中（重绘 + 回流）;
  * 改变浏览器大小（回流 + 重绘）;
  * 改变浏览器字体大小（回流 + 重绘）;
  * 改变元素 padding、border、margin（回流 + 重绘）;
  * 改变浏览器的字体颜色（重绘，不回流）;
  * 改变元素的背景颜色（重绘，不回流）


* 为什么简单的动画需要移动 dom 元素时，我们优先使用 transform 而不推荐使用 left top 等属性

  直接改变 left、top、margin、padding 等属性时会导致页面的重绘与回流，频繁改变会极大的损耗性能，因为一直页面一直在重绘与回流，而使用 transform 去偏移元素位置不会导致频繁回流，侧面提高了性能，尤其在移动端较明显

* 在没有兼容性（IE9 以上）的条件与时间间隔的条件下时，使用 js 去控制动画为什么我们要使用 requestAnimationFrame 去代替 定时器
  1. 使用定时器（setTimeout 或 setInterval）
    * 使用定时器来改变元素样式来写动画时，一般我们会反复的进行页面的重绘与回流，因为大部分动画情况下，我们不会只改变一个会导致回流的 css 属性，这会导致定时器每一个周期（一次 Timeout 或 一次 interval）发生多次回流的现象
    * 在浏览器当前页面是未激活的状态下时，定时器仍会调用（当然如果你不是用定时器来控制动画而是计数型操作的话请无视这单），但其实当前我们已经早就切换到浏览器其他页面了，根本没人关注你这个页面的动画是否仍在进行，此时动画仍在进行的话，无形间就占用了 CPU。

  2. 使用 requestAnimationFrame
    * requestAnimationFrame 会把每一帧中的所有 DOM 操作集中起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率，这就杜绝了执行一次动画而多次重绘回流的现象；
    * 在隐藏或不可见的元素中，requestAnimationFrame 将不会进行重绘或回流，减小了 CPU 与 GPU 的使用情况；
    * requestAnimationFrame 是浏览器专门为动画而生的 API，所以浏览器会自行对其进行优化，最明显的就是体现在，页面是未激活的情况时，动画会自动暂停，不会无效占用 CPU 与 GPU；
