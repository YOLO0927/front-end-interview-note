* 说说你了解的盒模型

  组成：由 width、height、padding、border、margin 几个属性组成盒模型，存在两种盒子模型；
  1. `box-sizing: content-box;` 以内容为盒子的限定，padding、border、margin 都不会被计入盒子元素尺寸内；

  2. `box-sizing: border-box;` 以边界作为盒子的限定，padding、border 的尺寸会被计入元素尺寸中，而 margin 不会被算入元素宽高中


* 未知宽高的盒子中如何水平竖直居中？
  1. 利用弹性布局 flex ，然后使用 `align-items: center`（元素垂直居中）、`align-content: center`（内容垂直居中）、`justify-content: 'center'`（元素水平居中）、`justify-items: 'center'`（内容水平居中）

  2. 特殊情况下可利用父级相对定位，子级绝对定位后设置 `top: 50%; left: 50; transform: translate(-50%, -50%)`来实现水平垂直居中
