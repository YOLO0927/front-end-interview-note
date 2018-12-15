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
