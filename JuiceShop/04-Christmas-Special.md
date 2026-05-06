Christmas Special - Juice Shop (⭐⭐⭐⭐)

挑战目标

订购 2014 年的圣诞节特别优惠商品（Christmas Super Surprise Box）

漏洞类型

IDOR / 参数篡改

---

完整复现步骤（含爆破细节）

第一步：抓取添加商品的正常请求

1. 关闭 Intercept（Intercept is off）
2. 在 Juice Shop 首页随便找一个商品（如 Apple Juice），点击 Add to Basket
3. 打开 Burp → Proxy → HTTP History
4. 找到最新的一条 POST 请求，特征：
   · Method: POST
   · URL: /rest/basket/1/items
   · Body: {"productId":1,"basketId":1,"quantity":1}
   或者抓到get请求之后先放行一次就能看到post请求，原因是这个add动作不止会发送一条指令，一共有三条，最先一条get是预制指令，后面一条才是post指令
6. 右键该请求 → Send to Intruder

---

第二步：配置 Intruder 爆破

2.1 设置爆破位置

1. 切换到 Intruder 标签页
2. 点击 Positions 子标签
3. 点击 Clear §（清除默认标记）
4. 在请求 Body 中，选中 productId 后面的数字（比如 1）
5. 点击 Add §，变成 §1§

最终 Body 格式：

```json
{"productId": §1§, "basketId": 1, "quantity": 1}
```

2.2 设置爆破范围

1. 点击 Payloads 子标签
2. Payload type 选择 Numbers
3. Number range 设置：
   · From: 1
   · To: 100（如果没找到可扩大到 200）
   · Step: 1
4. Number format：保持空（不填充）

2.3 开始攻击

1. 点击顶部菜单 Start attack
2. 弹窗中点击 Start attack 确认
3. 等待 100 个请求发送完成（约 5-10 秒）

---

第三步：从爆破结果中找到圣诞商品

方法一：搜索关键词（最快）

1. 在 Intruder 结果窗口，按 Ctrl + F
2. 搜索 Christmas
3. 找到匹配的那一行，查看对应的 productId 值

方法二：按响应长度排序

1. 点击 Length 列头排序
2. 找到长度明显不同的行（通常是 Length 较大的）
3. 点击该行，在下方 Response 窗口查看内容
4. 搜索是否有 "Christmas Super Surprise Box"

---

第四步：将商品加入购物车

方法一：直接在浏览器中添加（推荐）

1. 关闭 Intercept（Intercept is off）
2. 浏览器访问：
   ```
   http://localhost:3000/rest/basket/1/items
   ```
   不行的话用下面这个方法：

方法二：用 Repeater 手动添加

1. 在 Intruder 结果中找到对应的 productId（假设是 34）
2. 回到 Repeater 标签，粘贴之前抓到的 POST 请求
3. 将 productId 改成找到的数字
4. 点击 Send
5. 确认右侧响应为 {"status":"success"}

---

第五步：完成挑战

1. 打开 Juice Shop，点击右上角购物车图标
2. 确认 Christmas Super Surprise Box (2014 Edition) 已在购物车中
3. 点击 Checkout 结算
4. 完成支付流程
5. 打开 Score Board，确认 Christmas Special 变绿 ✅

---

原理说明（面试用）

IDOR（不安全的直接对象引用）：服务端在处理添加到购物车请求时，只校验了用户是否登录，没有校验该商品 ID 是否应该对该用户可见（如下架商品、过期商品、未发布商品）。因此攻击者可以通过枚举 productId，把任意有效商品强行加入自己的购物车。

一句话修复建议：

· 服务端维护一个“可购买商品白名单”
· 处理添加购物车请求时，校验 productId 是否在白名单中

---
