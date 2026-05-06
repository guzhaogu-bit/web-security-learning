Access Log - Juice Shop (⭐⭐⭐⭐)

挑战目标

访问服务器上的日志文件

漏洞类型

路径遍历（Path Traversal / Directory Traversal）

环境信息

· 靶场：Juice Shop
· 难度：四星
· 日期：2026-05-06

---

最终成功条件

项目 值
HTTP 方法 GET
URL /support/logs/access.log%2500.md
关键技巧 空字节截断（Null byte injection）

---

复现步骤

1. 发现入口

· 访问 /support/logs 路径
· 发现存在日志目录，但直接访问 access.log 被 403 拦截

2. 尝试绕过

· 直接请求 access.log → 403 Forbidden（服务器拒绝了访问）
· 尝试路径遍历 ../ → 无效
· 尝试截断绕过 ✨

3. 最终成功 Payload

```
/support/logs/access.log%2500.md
```

4. 验证结果

· Score Board 中 Access Log 变绿 ✅

---

Payload 详解

为什么需要 %2500.md？

1. %2500 是 %00 的 URL 双重编码（%25 = %，00 = 空字节）
2. 服务器解析后会变成 access.log\0.md
3. \0（空字节）告诉程序：字符串在这里结束，忽略后面的 .md
4. 程序以为自己读取的是 .md 文件（放行），实际读取的是 access.log

一句话原理

空字节截断：在文件系统操作中，\0 被视为字符串结束符，程序会忽略后面的内容。

---

踩坑记录

坑 现象 解决
直接访问 403 GET /support/logs/access.log → 403 需要绕过后缀检查
普通路径遍历无效 ../../../../etc/passwd → 404 入口不支持路径遍历
单次编码无效 %00.md → 还是 403 需要双重编码 %2500.md

---

修复建议（面试用）

问题：服务器只允许下载 .md 文件，但空字节截断绕过了后缀检查。

修复：

1. 严格控制文件路径：不要直接拼接用户输入，使用白名单
2. 禁止空字节：在输入验证阶段过滤 %00 或 \0
3. 使用编程语言提供的安全 API：如 Java 的 File.getCanonicalPath() 校验

```python
# 不安全
filename = f"/support/logs/{user_input}"

# 安全（类似方式）
if not user_input.endswith('.md'):
    return 403
if '\0' in user_input or '%00' in user_input:
    return 403
```

---

相关知识点

· 空字节注入（Null Byte Injection）
· URL 编码与双重编码
· 路径遍历的多种绕过方式
· 白名单 vs 黑名单过滤

