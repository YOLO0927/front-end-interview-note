1. 两种以上方式实现已知或者未知宽度的垂直水平居中
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

2. 移动端适配1px的问题
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

3. 改变哪些 css 属性会导致重绘或回流
  * 添加、删除元素（回流 + 重绘）;
  * 隐藏元素，`display: none（回流 + 重绘）`、`visibility: hidden（重绘，不回流）`;
  * 移动元素，改变 top、left、移动元素到另外的父元素中（重绘 + 回流）;
  * 改变浏览器大小（回流 + 重绘）;
  * 改变浏览器字体大小（回流 + 重绘）;
  * 改变元素 padding、border、margin（回流 + 重绘）;
  * 改变浏览器的字体颜色（重绘，不回流）;
  * 改变元素的背景颜色（重绘，不回流）
