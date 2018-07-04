## 描述
通过vue-cli创建项目需要用到websocket，然而用socket.io创建的socket存在跨域问题，提示如下:
> Failed to load http://localhost:8888/socket.io/?EIO=3&transport=polling&t=MHa8ktG: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8081' is therefore not allowed access. The response had HTTP status code 426.

通过在chrome调试network窗口看到ResponseHeaders没有Access-Control-Allow-Origin信息，因此在node_modules/webpack-dev-server/lib/Server.js的app.all函数中添加跨域信息
```js
 // 87行
 app.all('*', (req, res, next) => { // eslint-disable-line
   res.header('Access-Control-Allow-Origin', '*');
   res.header('Access-Control-Allow-Headers', 'Content-Type, Content-Length, Authorization, Accept, X-Requested-With , yourHeaderFeild');
   res.header('Access-Control-Allow-Methods', 'PUT, POST, GET, DELETE, OPTIONS');
   if (this.checkHost(req.headers)) { return next(); }
   res.send('Invalid Host header');
 });

```
此后在chrome调试network窗口可以看到Access-Control-Allow-Origin信息，但是还是跨域失败，需要继续研究。
