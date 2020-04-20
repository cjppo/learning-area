# 3种解决CORS错误的方式与Access-Control-Allow-Origin的作用原理

</br>

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
</br>
</br>

