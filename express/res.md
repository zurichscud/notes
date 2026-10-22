---
title: Express res 对象使用指南
lede: Express Response —— 设置状态码、响应头，并把数据返回给客户端
tags: [Node.js, Express, HTTP, 响应]

---

## 1. 概述

`res` 是 Express 在每个请求处理函数里传进来的**响应对象**，代表服务器要发回给客户端的 HTTP 响应。通过它可以设置状态码、响应头、Cookie，并发送文本、JSON、文件或重定向。

```js
app.get('/hello', (req, res) => {
  res.status(200).send('Hello Express');
});
```

- 它是 Node.js 原生 `http.ServerResponse` 的**增强版**，原生方法（`res.writeHead()`、`res.write()`、`res.end()`）依然可用
- 多数设置类方法返回 `res` 本身，可以**链式调用**
- 每个请求**只能发送一次响应**，发完再发会报错

## 2. 对象继承结构

```
stream.Writable
  └─ http.OutgoingMessage
      └─ http.ServerResponse        （Node.js 原生）
          └─ Express Response        （res，Express 在原型上扩展）
```

所以 `res` 也是一个可写流，可以直接 `stream.pipe(res)`。

## 3. 常用属性

| 属性              | 类型      | 说明                                                     |
| ----------------- | --------- | -------------------------------------------------------- |
| `res.app`         | `Object`  | 当前 Express 应用实例，等同于 `req.app`                  |
| `res.headersSent` | `Boolean` | 响应头是否已经发出，发出后就不能再改状态码和头           |
| `res.locals`      | `Object`  | 本次请求范围内的变量，中间件之间传值、模板渲染时可直接用 |
| `res.req`         | `Object`  | 与本响应对应的请求对象 `req`                             |
| `res.statusCode`  | `Number`  | 当前状态码（Node 原生属性），默认 `200`                  |

## 4. 常用方法

### 4.1 发送响应（会结束本次请求）

| 方法                                                 | 说明                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| `res.send([body])`                                   | 万能发送：字符串、Buffer、对象/数组都行，自动设 Content-Type |
| `res.json([body])`                                   | 以 JSON 格式发送，`Content-Type: application/json`           |
| `res.jsonp([body])`                                  | 以 JSONP 格式发送，回调参数名默认 `callback`                 |
| `res.sendStatus(statusCode)`                         | 设置状态码，并把状态描述文字作为响应体（如 `404` → Not Found） |
| `res.end([data] [, encoding])`                       | 直接结束响应，不做任何处理，适合无响应体的场景               |
| `res.sendFile(path [, options] [, fn])`              | 发送文件，按扩展名自动设 Content-Type；path 须为绝对路径或配合 `root` |
| `res.download(path [, filename] [, options] [, fn])` | 以附件形式发送文件，浏览器会弹出下载                         |
| `res.render(view [, locals] [, callback])`           | 渲染模板并发送 HTML                                          |
| `res.redirect([status,] path)`                       | 重定向，默认状态码 `302`                                     |

### 4.2 设置状态与响应头（不会结束请求）

| 方法                          | 说明                                                    |
| ----------------------------- | ------------------------------------------------------- |
| `res.status(code)`            | 设置状态码，可链式调用                                  |
| `res.set(field [, value])`    | 设置响应头，也可以传对象一次设多个；别名 `res.header()` |
| `res.get(field)`              | 读取已设置的响应头（不区分大小写）                      |
| `res.append(field [, value])` | 追加响应头，已存在时不会覆盖                            |
| `res.type(type)`              | 设置 Content-Type，可写 `'json'`、`'html'`、`'.png'` 等 |
| `res.attachment([filename])`  | 设置 `Content-Disposition: attachment`，提示浏览器下载  |
| `res.location(path)`          | 只设置 `Location` 头，不改状态码也不结束响应            |
| `res.links(links)`            | 设置 `Link` 头，常用于分页                              |
| `res.vary(field)`             | 往 `Vary` 头里添加字段                                  |
| `res.format(object)`          | 按请求的 `Accept` 头做内容协商，返回不同格式            |

### 4.3 Cookie

| 方法                                  | 说明                                               |
| ------------------------------------- | -------------------------------------------------- |
| `res.cookie(name, value [, options])` | 设置 Cookie，value 可以是字符串或对象（会转 JSON） |
| `res.clearCookie(name [, options])`   | 清除 Cookie，`path`、`domain` 要和设置时一致       |

常用 `options`：

| 选项       | 说明                                       |
| ---------- | ------------------------------------------ |
| `maxAge`   | 有效期（毫秒），比 `expires` 好用          |
| `expires`  | 过期时间（`Date`）                         |
| `httpOnly` | `true` 时前端 JS 读不到，防 XSS 偷 Cookie  |
| `secure`   | 只在 HTTPS 下发送                          |
| `sameSite` | `'strict'` / `'lax'` / `'none'`，防 CSRF   |
| `path`     | 生效路径，默认 `'/'`                       |
| `domain`   | 生效域名                                   |
| `signed`   | 签名 Cookie，需配合 `cookie-parser` 的密钥 |

### 4.4 事件监听

`res` 继承自 Node 的 `http.ServerResponse` 和 `stream.Writable`，也是一个 `EventEmitter`，所以可以用 `res.on(event, listener)` 监听响应生命周期。`res.once()` 只触发一次，`res.off()` / `res.removeListener()` 取消监听。

| 事件     | 触发时机                                                     | 常见用途                     |
| -------- | ------------------------------------------------------------ | ---------------------------- |
| `finish` | 响应最后一块数据已交给操作系统，即服务端已经“发完”           | 记录访问日志、统计耗时       |
| `close`  | 响应结束或底层连接被关闭；在较新的 Node 版本里正常完成后也会触发 | 清理资源、检测客户端中途断开 |
| `error`  | 写入过程中出错                                               | 流式输出时兜底处理错误       |
| `drain`  | `res.write()` 返回 `false` 后，缓冲区排空、可以继续写入      | 大量数据手动写入时做背压控制 |
| `pipe`   | 有可读流 `pipe(res)` 到它时                                  | 较少使用                     |
| `unpipe` | 可读流取消 pipe 时                                           | 较少使用                     |

> `finish` 只代表服务端发完，不代表客户端一定收到了。判断客户端是否中途断开：在 `close` 里看 `res.writableFinished` 是否为 `false`。

相关属性：

| 属性                   | 说明                                             |
| ---------------------- | ------------------------------------------------ |
| `res.writableEnded`    | 是否已经调用过 `res.end()`                       |
| `res.writableFinished` | 数据是否已经全部写出（`finish` 触发前为 `true`） |

## 5. 代码示例

### 5.1 发送文本、HTML 和 JSON

```js
app.get('/text', (req, res) => {
  res.send('纯文本');                 // Content-Type: text/html; charset=utf-8
});

app.get('/html', (req, res) => {
  res.send('<h1>标题</h1>');
});

app.get('/user', (req, res) => {
  res.json({ id: 1, name: '苏黎世' }); // Content-Type: application/json
});
```

> `res.send(对象)` 内部也会调用 `res.json()`，但写 `res.json()` 意图更清楚，还能正确处理 `null`、数字等值。

### 5.2 状态码 + 链式调用

```js
app.post('/users', (req, res) => {
  const user = createUser(req.body);
  res.status(201).json(user);
});

app.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  if (!user) {
    return res.status(404).json({ message: '用户不存在' }); // 记得 return
  }
  res.json(user);
});

app.delete('/users/:id', (req, res) => {
  deleteUser(req.params.id);
  res.sendStatus(204); // 无响应体
});
```

### 5.3 设置响应头

```js
app.get('/api/data', (req, res) => {
  res.set({
    'Cache-Control': 'no-store',
    'X-Request-Id': req.id,
  });
  res.append('Set-Cookie', 'a=1; Path=/');
  res.type('json').send('{"ok":true}');
});
```

### 5.4 Cookie

```js
app.post('/login', (req, res) => {
  res.cookie('token', 'abc123', {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    maxAge: 7 * 24 * 60 * 60 * 1000, // 7 天
  });
  res.json({ ok: true });
});

app.post('/logout', (req, res) => {
  res.clearCookie('token', { httpOnly: true, secure: true, sameSite: 'lax' });
  res.json({ ok: true });
});
```

### 5.5 发送文件与下载

```js
const path = require('path');

app.get('/avatar', (req, res) => {
  res.sendFile('avatar.png', { root: path.join(__dirname, 'public') }, (err) => {
    if (err) res.status(404).end();
  });
});

app.get('/report', (req, res) => {
  // 浏览器下载时显示的文件名为“月度报告.pdf”
  res.download(path.join(__dirname, 'files', 'report.pdf'), '月度报告.pdf');
});
```

> 千万不要把用户输入直接拼进 `sendFile` 路径，要用 `root` 选项，防止 `../` 路径穿越。

### 5.6 重定向

```js
app.get('/old', (req, res) => {
  res.redirect('/new');            // 302 临时重定向
});

app.get('/moved', (req, res) => {
  res.redirect(301, '/new-home');  // 301 永久重定向，状态码写在前面
});
```

### 5.7 res.locals 在中间件间传值

```js
app.use((req, res, next) => {
  res.locals.user = req.session?.user;
  res.locals.startTime = Date.now();
  next();
});

app.get('/profile', (req, res) => {
  res.render('profile'); // 模板里可以直接使用 user
});
```

### 5.8 内容协商 res.format

```js
app.get('/info', (req, res) => {
  res.format({
    'text/plain': () => res.send('hello'),
    'text/html': () => res.send('<p>hello</p>'),
    'application/json': () => res.json({ message: 'hello' }),
    default: () => res.status(406).send('Not Acceptable'),
  });
});
```

### 5.9 流式响应

```js
const fs = require('fs');

app.get('/big.log', (req, res) => {
  res.type('text/plain');
  fs.createReadStream('/var/log/big.log').pipe(res); // res 本身是可写流
});
```

### 5.10 用 res.on 记录请求耗时（日志中间件）

```js
app.use((req, res, next) => {
  const start = process.hrtime.bigint();

  res.on('finish', () => {
//换算成了以毫秒为单位的耗时
    const ms = Number(process.hrtime.bigint() - start) / 1e6;
    console.log(`${req.method} ${req.originalUrl} ${res.statusCode} ${ms.toFixed(1)}ms`);
  });

  next();
});
```

> `morgan` 等日志库就是靠监听 `finish` / `close` 来拿到最终状态码和耗时的。

### 5.11 检测客户端中途断开，及时停止工作

```js
app.get('/slow-report', async (req, res) => {
  const controller = new AbortController();

  res.on('close', () => {
    if (!res.writableFinished) {
      console.log('客户端提前断开');
      controller.abort(); // 取消还在进行的查询/请求
    }
  });

  try {
    const data = await buildReport({ signal: controller.signal });
    res.json(data);
  } catch (err) {
    if (!controller.signal.aborted) res.status(500).json({ message: '生成失败' });
  }
});
```

### 5.12 SSE（服务端推送）中清理定时器

```js
app.get('/events', (req, res) => {
  res.set({
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    Connection: 'keep-alive',
  });
  res.flushHeaders();

  const timer = setInterval(() => {
    res.write(`data: ${new Date().toISOString()}\n\n`);
  }, 1000);

  res.on('close', () => clearInterval(timer)); // 客户端关闭页面就停
});
```

## 6. 常见坑：重复发送响应

一次请求只能响应一次，第二次会报：

```
Error [ERR_HTTP_HEADERS_SENT]: Cannot set headers after they are sent to the client
```

常见原因和改法：

```js
// ❌ 错误：if 里发完没有 return，继续往下又发了一次
app.get('/a', (req, res) => {
  if (!req.query.id) {
    res.status(400).send('缺少 id');
  }
  res.send('ok');
});

// ✅ 正确
app.get('/a', (req, res) => {
  if (!req.query.id) {
    return res.status(400).send('缺少 id');
  }
  res.send('ok');
});
```

在错误处理中间件里，不确定时先检查：

```js
app.use((err, req, res, next) => {
  if (res.headersSent) return next(err); // 交给 Express 默认处理
  res.status(500).json({ message: '服务器错误' });
});
```

## 7. Express 4 与 Express 5 的区别

Express 5 已是当前默认版本（`npm i express` 安装的就是 5.x），`res` 上有这些变化：

| Express 4 写法                  | Express 5                                                 |
| ------------------------------- | --------------------------------------------------------- |
| `res.send(404)`（数字当状态码） | 已移除，改用 `res.sendStatus(404)`                        |
| `res.send(body, status)`        | 已移除，改用 `res.status(status).send(body)`              |
| `res.json(obj, status)`         | 已移除，改用 `res.status(status).json(obj)`               |
| `res.redirect(url, status)`     | 已移除，改用 `res.redirect(status, url)`                  |
| `res.redirect('back')`          | 已移除，改用 `res.redirect(req.get('Referrer') \|\| '/')` |
| `res.sendfile()`（小写 f）      | 已移除，改用 `res.sendFile()`                             |
| `res.status(非法值)`            | 现在会直接抛错，只接受 100~999 的整数                     |
| `res.clearCookie` 传 `maxAge`   | 现在会忽略 `maxAge` / `expires`                           |

## 8. 发送方法对比

| 方法               | 自动设 Content-Type | 结束响应 | 适合场景                      |
| ------------------ | ------------------- | -------- | ----------------------------- |
| `res.send()`       | ✅ 按数据类型推断    | ✅        | 通用：文本、HTML、Buffer      |
| `res.json()`       | ✅ JSON              | ✅        | REST API（最常用）            |
| `res.sendStatus()` | ✅ 文本              | ✅        | 只需返回状态码                |
| `res.end()`        | ❌                   | ✅        | 无响应体，如 `204`、`304`     |
| `res.sendFile()`   | ✅ 按扩展名          | ✅        | 在浏览器中直接展示文件        |
| `res.download()`   | ✅ + 附件头          | ✅        | 让浏览器下载文件              |
| `res.render()`     | ✅ HTML              | ✅        | 服务端模板渲染                |
| `res.redirect()`   | ✅                   | ✅        | 跳转到另一个地址              |
| `res.write()`      | ❌                   | ❌        | 分块/流式输出，最后要 `end()` |

## 9. 最佳实践

1. **发送后立刻 `return`**，避免 `ERR_HTTP_HEADERS_SENT`。
2. **API 统一用 `res.status(code).json(...)`**，状态码语义要对：`201` 创建、`204` 无内容、`400` 参数错误、`401` 未登录、`403` 无权限、`404` 不存在、`500` 服务器错误。
3. **统一错误响应格式**，比如 `{ code, message }`，放在错误处理中间件里集中处理。
4. **敏感 Cookie 加 `httpOnly`、`secure`、`sameSite`**。
5. **`sendFile` / `download` 用 `root` 选项**，不要直接拼接用户输入的路径。
6. **请求级数据放 `res.locals`**，不要挂到全局或 `app.locals`。
7. **大文件用流**（`pipe(res)` 或 `sendFile`），不要先整个读进内存再 `send`。
8. **长连接和耗时任务监听 `res.on('close')`**，客户端断开就清理定时器、取消查询。
9. **新项目按 Express 5 写法**：重定向状态码在前，用 `sendStatus` 而不是 `send(数字)`。