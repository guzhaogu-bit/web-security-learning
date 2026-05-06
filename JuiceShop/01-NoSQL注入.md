# NoSQL Manipulation - Juice Shop (⭐⭐⭐⭐)

## 挑战目标
同时更新多条商品评论（Update multiple product reviews at the same time）

## 漏洞类型
NoSQL 注入（MongoDB）

## 环境信息
- 靶场：Juice Shop
- 难度：四星
- 日期：2026-05-06

## 复现步骤

### 1. 登录
- 使用 SQL 注入登录 admin 账号
- 邮箱：`' or 1=1--`
- 密码：任意

### 2. 准备一条评论
- 进入任意商品页面（如 Apple Juice）
- 写一条自己的评论
- 记住这条评论的作者名和内容

### 3. 抓包
- 打开 Burp Suite，开启 Intercept
- 点击评论旁的 'Edit' 按钮，修改评论内容
- 点击 Submit 提交
- Burp 抓到请求

### 4. 修改请求
- 确认 HTTP 方法是 `PATCH`（不是 PUT）
- 确认 URL 是 `/rest/products/reviews`
- 确认 Body 中有 `id` 字段
- 将 `"id": "原作者名"` 改为 `"id": {"$ne": null}`(注���这边的双引号要去掉）
- 将 `"message"` 改为任意内容（如 `"Hacked by NoSQL"`）

### 5. 发送并刷新
- 点击 Forward 发送请求
- 关闭 Intercept（Intercept is off）
- 回到浏览器，刷新商品页面

### 6. 验证结果
- 商品页面中 所有评论 都变成了你修改后的内容
- 打开 Score Board，`NoSQL Manipulation` 变绿 ✅

## Payload 详解

### 原始请求
```json
{
  "id": "Jim",
  "message": "This is a great product!"
}
```

### 攻击请求
```json
{
  "id": {"$ne": null},
  "message": "Hacked by NoSQL"
}
```

## 原理
- MongoDB 将 `{"$ne": null}` 解析为操作符，不是普通字符串
- `$ne` = "not equal"（不等于）
- 查询条件变成：找到所有 authors name 不等于 null 的评论
- 因为所有评论的作者名都不是 null，所以匹配了全部评论
- 配合 PATCH 方法，实现批量更新
