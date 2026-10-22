
## shadow
Shadow DOM（当前方案）：只隔离 CSS 和 DOM，JS 仍在宿主页上下文。因此能直接使用宿主的 localStorage 保存访客 ID（widget.js:51-56），直接开 WebSocket，不产生第二次文档请求，体积和性能都更好；与页面交互路径短，window.ShiguangChat 可直接调用。

## iframe
 iframe 方案：能连 JS、location、cookie 一起隔离，但需要独立聊天页和 postMessage 协议桥，跨域场景下存储和 WebSocket 都更麻烦，还多一次文档加载。