# Allowlist Bypass - Juice Shop (⭐⭐⭐⭐)

## 漏洞类型
开放重定向（Open Redirect）/ 白名单绕过

## 最终成功 Payload
http://127.0.0.1:3000/redirect?to=https://baidu.com?to=https://github.com/juice-shop/juice-shop

## 原理
- 白名单校验只检查 URL 末尾是否包含允许的域名
- 通过 `?to=` 附加合法地址，使校验通过
- 实际浏览器解析时跳转到第一个 `to=` 参数的值

## 修复建议
- 使用严格的白名单匹配（精确域名比对）
- 避免使用字符串 `endsWith()` 这类部分匹配函数
- 对用户输入的 URL 进行规范化（canonicalization）后再校验
