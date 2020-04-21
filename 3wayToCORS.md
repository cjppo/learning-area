# 3种解决CORS错误的方式与Access-Control-Allow-Origin的作用原理

</br>

<font size=1>*&ensp;本文翻译自：
[3 Ways to Fix the CORS Error — and How the Access-Control-Allow-Origin Header Works](https://medium.com/@dtkatz/3-ways-to-fix-the-cors-error-and-how-access-control-allow-origin-works-d97d55946d9)
<font size=1>


## **CORS错误是什么**

![img](https://miro.medium.com/max/3200/0*bI2yxKryqJzyUkud)

是否曾经见过这个错误，或刚刚碰见？



必须说，我们在页面程序代码中用到各种API时，这个bug出现的太多了。每次碰到这个bug，内心都是如图所示：

![img](https://miro.medium.com/max/1000/1*vP1drWY1myDhV99P9YHhGg.png)

</br>
</br>

## **第一种方法：安装一个Allow-Control-Allow-Origin插件**
最简单粗暴的解决方法就是安装一个插件：[moesif CORS extension](https://chrome.google.com/webstore/detail/moesif-orign-cors-changer/digfbfaphojjndkpccljibejjbppifbc?hl=en-US)。打开链接，安装好插件，在你的浏览器中启用插件（插件图标变为了"on"）。
</br>
</br>
刷新你的页面，就可以正常访问API了。🎉
</br>
</br>
### **不过这种方法只是自我欺骗罢了**
插件是解决了问题，但这只在你自己的机器上起作用。本地开发调试时，插件确实能帮你直接跳过这个问题。
</br>
</br>
但是你的页面应用总要发布，你的用户并不一定会安装这个插件。因此这个方法并不太好。。。
</br>
</br>
当然，我们有更好的办法。不过在看这些方法之前，我们先来看以下几个问题：
</br>
</br>
### **首先，CORS错误为什么会发生**
这个错误起源于一个浏览器中的安全机制：[同源策略(same-origin policy)](https://en.wikipedia.org/wiki/Same-origin_policy)。
</br>
</br>
这个安全机制能够防止一种非常常见的网络攻击：[跨站请求伪造(Cross-site request forgery)](https://en.wikipedia.org/wiki/Cross-site_request_forgery)，通常都是利用浏览器中存储的cookie来伪造。
</br>
</br>
每当浏览器对一个域发出HTTP请求，都会带上这个域所属的cookie。这对认证或维持session等功能非常有用。比如，当你访问facebook-clone.com的时候，你的浏览器就会存储一个于这个域相关会话的cookie：
![img](https://miro.medium.com/max/1400/0*3lrH9-T9suD93ZzE)
当cookie保存在你的浏览器时，每次你重新访问facebook-clone.com或其域内的网址时，就不用再重新登陆了，API会认得你已经存储的会话cookie。
</br>
</br>
但这个机制有个麻烦，每当访问某个域的时候，浏览器都会自动地带上域相关的cookie。因此，假设这样一个场景，你点开了一个恶意弹出的页面：evil-site.com，
</br>
![img](https://miro.medium.com/max/1400/0*Buuk1Xt78FVpnf3W)
evil-site中有访问facebook-clone.com/api的代码。由于你曾经访问过facebook-clone.com，本地保存了cookie。所以在evil-site访问相同域的资源时，浏览器会带着cookie，因此获得facebook-clone.com的认证，至此你的账户成功地被evil-site用伪造的跨站请求黑了。
</br>
</br>
不过，现在的浏览器都会介入这个访问的过程，防止这些恶意代码对API的伪造请求，实际上evil-site对API的请求只能得到一个失败的结果，失败原因：*同源策略不允许这样的操作*。
</br>
</br>
### **揭开同源策略背后的原理**
每当发起访问时，浏览器在背后都会校验web app和服务端的***源***是否匹配，上面的例子中，***源***被简化为了前端页面程序和后端服务程序，实际上，***源***是由一系列属性构成的，包括网络传输协议，主机名称，端口等等。比如，https://www,facebook-clone.com的传输协议是https，主机名称是www.facebook-clone.com，端口是443（这里被隐藏了，https协议的默认访问端口是443）。
为了验证同源，浏览器会在所有请求中附带一个特殊的请求一起发送给域信息接收服务器。比如有个应用运行在域localhost:3000上，这个特殊请求的格式将像下面这样：
</br>
```js
Origin: http://localhost:3000
```

作为回应，服务器返回的response会附带一个包含键为<td bgcolor=gray>Access-Control-Allow-Origin</td>的header，用来标示什么样的***源***可以访问服务器的资源。该键可以对应两种值：

第一种，严格模式，只有一个指定的***源***可以获得权限访问:


```js
Access-Control-Allow-Origin: http://localhost:3000
```
第二种，宽松模式，服务端允许任何***源***访问自己的资源：
```js
Access-Control-Allow-Origin: *
```

浏览器接收到该响应后，会把前端应用的域和服务器返回的Access-Control-Allow-Origin值比较，如果比较不匹配，那么此次访问就被红灯了，并且会返回一个CORS错误的提示。
</br>
</br>
### **那么Allow-Control-Allow-Origin插件修复这个问题了吗**

**当然是没有**，插件仅仅是关闭了浏览器的同源策略检查而已。插件会在每次请求的response中加入一个
*Access-Control-Allow-Origin:&ensp;**
的head罢了，以此来欺骗浏览器仿佛服务器真的允许所有***源***的访问。
</br>
</br>
搞清楚了原因，我们知道我们不能掩耳盗铃。也许你知道，服务端会对你的线上页面放行，所以你在本地开发时用插件跳过检查。但如果你要使用其他的API，插件就不一定能帮你“修好”这个问题了。就想前面说过的，你不可能指望所有你网站的用户，都会安装这个插件。
</br>
</br>
## **第二种方法，通过代理发送你的请求**
你无法确定你的用户是否会安装浏览器插件，但是你可以确定的是，你可以控制你的web app访问的API的地址。
</br>
</br>
[cors-anywhere](https://github.com/Rob--W/cors-anywhere/#documentation)是一个能帮你在请求header中添加CORS内容的代理服务。作为客户端和服务端之间的中间人，这个代理服务会帮助你的前端web app发送请求，并且接收服务端的返回数据再传送给前端web app。和Allow-control-allow-origin插件一样，代理服务会在收到服务端返回的时候，在返回结果中插入
*Access-Control-Allow-Origin:&ensp;**
header。

假设你有一个对下面地址的GET请求：

```js
https://joke-api-strict-cors.appspot.com/jokes/random
```

但是这个api的返回的Access-Control-Allow-Origin中并没有包含你所在域的***源***，结果当然是访问不了。于是你修改一下请求的地址：

```js
https://cors-anywhere.herokuapp.com/https://joke-api-strict-cors.appspot.com/jokes/random
```

代理服务获取到了你想要访问的真正地址：https://joke-api-strict-cors.appspot.com/jokes/random，然后帮你把请求发给它，并收到api的返回。最后代理服务在原始的返回中，附加上一个
*Access-Control-Allow-Origin:&ensp;**
header。
</br>
</br>
这个解决方法在开发和生产环境中都可以生效。而其为什么会生效，是因为这个方法利用了*同源策略*只作用于浏览器-服务直间的通信，而不会限制服务-服务之间的通信！

不过这个方法有一个缺点就是通过代理会使得响应通常会变得更慢，代理导致的延迟会让你的应用看起来有点迟缓。

因此我们还需要第三种方法。

## **第三种方法：建立你自己的代理**
我推荐你建立自己的代理去解决这个问题。解决问题的原理你已经知道：利用*同源策略*只会作用于浏览器-服务之间，而不限制服务-服务的通信。你只需要建立自己的代理，而且不用与别人共享，能很大程度上消除延迟的影响，并且能用在你所有的服务上。

这里有一份简便的Node.js代码，通过express web framework去创建一个代理访问跟上面同样的https://joke-api-strict-cors.appspot.com/：

```javascript
const express = require('express');
const request = require('request');

const app = express();

app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  next();
});

app.get('/jokes/random', (req, res) => {
  request(
    { url: 'https://joke-api-strict-cors.appspot.com/jokes/random' },
    (error, response, body) => {
      if (error || response.statusCode !== 200) {
        return res.status(500).json({ type: 'error', message: err.message });
      }

      res.json(JSON.parse(body));
    }
  )
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`listening on ${PORT}`));
```

<font size=1>If you want to see this in action, head to the source code for the above, along with relevant steps in the README: [https://github.com/15Dkatz/beat-cors-server](https://github.com/15Dkatz/beat-cors-server)</font>


这段代码的工作原理是：利用了express中间件把服务端返回的所有返回结果都添加了
*Access-Control-Allow-Origin:&ensp;**
header。当有GET请求到了代理的jokes/random，代理服务会去另外的服务端请求一个随机笑话。即使代理的域和服务端的域并不一致，同源策略也不会介入生效，毕竟这是一个server-server的请求。最后，代理服务会对最初请求方（一个浏览器上的应用）返回一个响应，响应包含了从真正服务端得到的数据和中间件添加的
*Access-Control-Allow-Origin:&ensp;**
header。
</br>
</br>
## **总结**
CORS错误经常烦着前端开发人员。不过知道了这背后的同源策略，以及他们是怎么帮你防止跨站请求伪造攻击，你应该觉得这还是可以接收的，毕竟安全第一。
</br>
</br>
最后，希望有了上面的解决方法，当你再看到浏览器日志中的CORS错误时，应该不会再觉得棘手了。
