* webpack的原理, loader 和 plugin 是干什么的? 有自己手写过么 ?

  **loader**:
  1. 是 webpack 用于在编译过程中解析各类文件格式，并输出；
  2. 本质上就是一个 node 模块，通过写一个函数来完成自动化的过程；
  3. 由此我们就可以在开发模式下，通过解析各类前端无法解析的文件格式，然后将其解析后返回为对象或字符串供前端开发时使用，在 webpack 的编译过程中自动会将我们前端项目中引用的文件格式对应到指定 loader 解析后输出
  4. 接下来我将写一个非常简单的 loader 来解析 txt 文件，它将满足以下功能
    * 读取 txt 文件内容并输出为一个对象，对象内包括文件内容与文件名
    * 读取 webpack loader 选项，将内容中的 [name] 替换为我们选项 name 的值
      ```js
      // webpack.config.js
      module.exports = {
        //...
        module: {
          rules: [
            {
              test: /\.txt$/,
              use: {
                loader: path.resolve(__dirname, './txt-loader.js'),
                options: {
                  name: 'YOLO'
                }
              }
            }
          ]
        }
      }
      ```

      ```js
        // txt-loader.js
        var utils = require('loader-utils')

        module.exports = function (source) {
          const options = utils.getOptions(this)

          source = source.replace(/\[name\]/g, options.name)
          return `export default ${ JSON.stringify({
            content: source,
            filename: this.resourcePath
          }) }`
        }
      ```
      ```txt
        // 测试在项目中 import 的 txt 文件
        test loader output from [name]
      ```
      ```js
        // 前端引用
        import test from '@/public/test.txt'
        // 在浏览器中 log 出结果，因为 txt 中保存时自动换行了，所以大家可以看到存在一个换行符
        {
          content: "test loader output from YOLO↵"
          filename: "/Users/yanglu/Desktop/Programers/test-loader/src/public/test.txt"
        }
      ```
  更复杂的用法，请大家自行参照文档<br>
  https://webpack.docschina.org/contribute/writing-a-loader/

  **plugin**:
  1. 是 webpack 用于在编译过程中利用钩子进行各种自定义输出的函数；
  2. 本质上就是一个 node 模块，通过写一个类来使用编译暴漏出来的钩子实现编译过程的可控；
  3. 由此我们就可以在开发模式下，可以通过监听编译过程的各个钩子事件来完成如释出模版，对 js、css、html 进行压缩、去重等各类操作，结束后释出对应文件等等你可以想到的任何操作

    webpack 的插件简单来说就是在函数中通过调用 webpack 执行的钩子来完成自动化的过程，在函数中我们通过监听 compiler 钩子，并在回调中执行我们需要做的事情，最后调用回调中的第二个参数 callback 使 webpack 继续构建，否则将在此处停止编译，整个过程都是在 webpack 的整个编译过程中利用其暴漏出的钩子进行的，以下是我写的一个简短的例子，它完成了这样的功能：

    * 仿照 htmlWebpackPlugin 进行模版输出，实例化插件时传入模版地址，释出文件名，需解析参数变量即可；

    * 解析过程中会解析 {{ key }} 中的 param，将 {{ key }} => 实例化对象中 params[key]

    ```js
      // webpack 配置
      plugins: [
        new EmitTemplate({
          template: './src/template/template.html',
          filename: path.resolve(__dirname, './index-emit-template.html'),
          params: {
            title: '测试啊',
            name: 'YOLO'
          }
        })
      ]

      // emit-template-plugin.js
      const fs = require('fs')

      function EmitTemplate (options) {
        this.templateDir = options.template
        this.targetDir = options.filename
        this.params = options.params
      }

      EmitTemplate.prototype.apply = function (compiler) {
        var self = this
        compiler.plugin('emit', function (compilation, callback) {
          fs.readFile(self.templateDir, (err, data) => {
            if (err) throw err
            // 匹配参数替换 html 模版中变量
            var reg = new RegExp(`{{\\s*(${Object.keys(self.params).join('|')})\\s*}}`, 'g')
            var html = data.toString().replace(reg, function (str, key, index) {
              return self.params[key]
            })

            fs.writeFile(self.targetDir, html, function () {
              console.log('模版写入成功')
              callback()
            })
          })
        })
      }

      module.exports = EmitTemplate
    ```
    ```html
      // 模版 template.html
      <!DOCTYPE html>
      <html lang="en">
        <head>
          <meta charset="utf-8">
          <title>{{ title }}</title>
        </head>
        <body>
          <div class="">
            {{ name }}
          </div>
        </body>
      </html>

      // 解析后的释出文件 index-emit-template.html
      <!DOCTYPE html>
      <html lang="en">
        <head>
          <meta charset="utf-8">
          <title>测试啊</title>
        </head>
        <body>
          <div class="">
            YOLO
          </div>
        </body>
      </html>
    ```
    上面的写法是 webpack2 或 webpack3 调用钩子的方法，在 4 中已变为
    ```js
    compilation.hooks.compile.tap([compiler hooks event], (compilation, callback) => {
      ...
    })
    ```
    更多关于编写插件的详情请大家自行查阅文档<br>
    https://webpack.docschina.org/contribute/writing-a-plugin/

* webpack 中路由的动态加载模块

  可利用 import 语法实现动态导入模块，而在我们正常开发中的单页应用中，一个路由即是一个相对较大的组件，我们只需在路由文件中，动态的异步导入路由组件即可。

  在使用 babel 的情况下使用 [syntax-dynamic-import](https://babeljs.io/docs/en/babel-plugin-syntax-dynamic-import/) babel 插件，这样便可以令 import 返回 promise 来实现动态加载后的挂载处理，在 vue-router 中是直接支持返回的 promise 作为组件引入的，详见 [vue-router 路由懒加载](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html)，由于 promise 是 chunk 函数，若你使用 babel 并支持 async await 的话，你可以可以这样写
  ```js
    // promise
    import('[component-url]').then(component => {
      // 进行路由挂载
      ...
    })
    // async await
    var getComponent = async () {
      let component = await import('[component-url]')
      return ...
    }
    getComponent().then(component => {
      ...
    })
  ```

  现阶段推荐使用 import 的方法替换原本 webpack require.ensure 的方法，否则会比较麻烦，有需要大家可查原本当了解 [webpack-dynamic imports](https://webpack.docschina.org/guides/code-splitting/#%E5%8A%A8%E6%80%81%E5%AF%BC%E5%85%A5-dynamic-imports-)

* 服务端渲染SSR
  1. 为什么会重新使用服务端渲染

    由于现在的开发模式造成前后端分离后，前端为了提升用户体验而大量采用了异步请求的方式，导致很多页面在初始化时 html 很多 dom 节点中的内容是空的，在这之后由异步数据来填充，或如今采用三大框架后，vue 是采用 dom 劫持的方式重构 dom，而 react 是采用 virtual-dom，这会导致一开始页面中的内容全是变量名，结果就是爬虫无法爬取到网页内容而影响了 SEO，导致网页排名下降。
  2. ssr 优势
    * 提升首屏渲染的速度，尤其是单页应用，由于用户侧的客户端网速与性能是无法预测的，如果前端计算量过大，在移动端部分老旧的手机可能无法撑起繁重的计算过程，这个时候我们将逻辑放在服务端计算好在渲染出来，而客户端只负责解析 html，这就避免了这些繁琐的过程，从而加快首屏渲染的速度；
    * 首屏渲染时html中的内容已经被加载完毕后才渲染的，有利于 SEO；
    * 服务端解析后很多动态获取的数据可转化为静态资源，有利于更多的缓存；
  3. ssr 劣势
    * 维护较为困难，即使使用 node 做服务端渲染，也需要开发者对 node 比较熟悉；
    * 部分情况下，用户体验可能较差，比如前端懒加载。

* webpack 的 dev-server 是怎么跑起来的
  1. 利用 express 创建一个 http 服务，所以你看到 devServer 的配置中是可以配置 host 和 port 的，并使用 websocket 长链接实现实时监听；
  2. 初始化一个 hash 指，每次触发 HRM（热替换 hot module replace）时会生成一个新的 hash 指，并跟上次的 hash 值做比较，若不同则会触发模块的热替换，若删除了某些模块，或添加了模块，则会触发替换的同时触发浏览器的 reload；
  3. 如何触发的更新的，实际上是通过 chokidar 模块去监听整个项目下的文件变化来实现的，该模块本质上就是 fs.watch 或 fs.watchFile 去监听指定项目目录下的文件变化，通过文件变化来触发 HMR runtime 异步下载更新，然后通知引用代码完成更新，具体编译原理较复杂可以查阅 webpack 的 [热替换文档](https://webpack.docschina.org/concepts/hot-module-replacement/)
