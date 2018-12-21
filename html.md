* 说说你了解的盒模型

  组成：由 width、height、padding、border、margin 几个属性组成盒模型，存在两种盒子模型；
  1. `box-sizing: content-box;` 以内容为盒子的限定，padding、border、margin 都不会被计入盒子元素尺寸内；

  2. `box-sizing: border-box;` 以边界作为盒子的限定，padding、border 的尺寸会被计入元素尺寸中，而 margin 不会被算入元素宽高中


* 未知宽高的盒子中如何水平竖直居中？
  1. 利用弹性布局 flex ，然后使用 `align-items: center`（元素垂直居中）、`align-content: center`（内容垂直居中）、`justify-content: 'center'`（元素水平居中）、`justify-items: 'center'`（内容水平居中）

  2. 特殊情况下可利用父级相对定位，子级绝对定位后设置 `top: 50%; left: 50; transform: translate(-50%, -50%)`来实现水平垂直居中

* 如何实现 rem 布局
  1. 设置文档节点根元素的字体大小为标准单位
  ```js
    document.documentElement.style.fontSize = document.documentElement.clientWidth / 7.5 + 'px'
  ```

  2. 所有盒子模型的单位都替换为 rem，一般来说移动端宽 375 的情况下，1 rem == 50px 为最佳
  3. 监听屏幕 resize 事件，动态赋值第一步字体单位

* 圣杯或双飞翼（两边固定宽，中间自适应）
```html
  <style>
    .left1{
      float: left;
      width: 200px;
      background-color: red;
      height: 200px;
    }
    .right1{
      float: right;
      width: 220px;
      height: 200px;
      background-color: green;
    }
    .middle1{
      margin-left: 200px;
      margin-right: 220px;
      height: 200px;
      background-color: blue;
    }
  </style>
  <body>
    <div class="left1"></div>
    <div class="right1"></div>
    <div class="middle1"></div>
  </body>
```
