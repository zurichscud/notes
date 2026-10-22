## 为什么登录时进行IP校验

2. IP 限流防分布式/并发暴力破解：按 IP 可以限制同一来源的尝试频率；虽然攻击者可换 IP（代理池），但成本更高，且配合后续多账号/Redis 限流可叠加。
3. 避免账号锁定 DoS：按用户名限流存在"恶意用户故意输错密码把真实客服账号锁死"的风险；IP 限流把影响限制在攻击来源，正常客服从其他 IP 仍可登录。
![](https://markdown-lai.oss-cn-hangzhou.aliyuncs.com/floral-notepaper/1cdec3f2-7f7a-4dda-ae08-82a317e9509d.png)
