
```js
export const requestContext: RequestHandler = (req, res, next) => {
  const incomingId = req.headers['x-request-id'];
  const requestId =
    typeof incomingId === 'string' && incomingId.length > 0
      ? incomingId
      : randomUUID();

  res.locals.requestId = requestId;
  res.setHeader('X-Request-Id', requestId);

  const startedAt = process.hrtime.bigint();

  res.on('finish', () => {
    const durationMs = Number(process.hrtime.bigint() - startedAt) / 1_000_000;
    const meta = {
      requestId,
      method: req.method,
      path: req.originalUrl.split('?')[0],
      statusCode: res.statusCode,
      durationMs: Math.round(durationMs * 10) / 10,
    };

    if (req.originalUrl.startsWith('/api/health')) {
      logger.debug(meta, 'HTTP request');
    } else {
      logger.info(meta, 'HTTP request');
    }
  });

  next();
};
```
主要作用是为每个 HTTP 请求建立全链路追踪标识（Request ID）

读取上游 ID：检查请求头 req.headers['x-request-id']。如果请求经过了网关（如 Nginx、Spring Cloud Gateway）或前端已经生成了 Trace ID，则直接复用该 ID，保证全链路日志串联。

兜底生成：如果请求头中没有有效的字符串 ID，则调用 Node.js 内置的 crypto.randomUUID() 生成一个新的 UUID v4。

上下文挂载与回传：

res.locals.requestId = requestId：将 ID 挂载到当前请求的生命周期上下文 res.locals 中，方便后续的 Controller 或错误处理中间件直接获取并打印到业务日志里。

res.setHeader('X-Request-Id', requestId)：将该 ID 写入响应头返回给客户端（前端），一旦接口报错，前端可以直接拿着这个 ID 找后端排查问题。