X-Content-Type-Options: nosniff 是一个 HTTP 安全响应头，作用是禁止浏览器做 MIME 类型嗅探。

背景：浏览器有时会忽略服务器声明的 Content-Type，根据响应内容猜测实际类型（MIME sniffing）。比如一个被标记为 text/plain 的文件，浏览器可能 sniff 出它是 HTML/JS 并执行——这就是 XSS/内容注入风险。
加上 nosniff 后，浏览器会严格按服务器声明的 Content-Type 处理，不再猜测。