
```js
app.disable('x-powered-by');
```
 Express 默认会设置 X-Powered-By: Express 响应头，向客户端暴露服务端使用的框架/技术栈。
- 攻击者可据此识别技术栈，针对性地利用已知漏洞或构造攻击（比如 Express 特定版本的已知问题）。

```js
app.use(express.json({ limit: '32kb' }));
```
app.use(express.json({ limit: '32kb' })) 是注册 Express 的 JSON body 解析中间件，作用：
1. 解析请求体：把 Content-Type: application/json 的请求体解析成 JavaScript 对象，挂到 req.body，后续路由/控制器可以直接用 req.body。
2. 大小限制：limit: '32kb' 限制请求体最大 32KB。
