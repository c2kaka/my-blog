---
title: "退到边界上：AI 时代的测试新范式"
pubDate: "2026-05-10"
description: "少测实现路径与 mock，多测 API 契约、数据最终状态与业务不变量。为什么在 AI 写代码的时代，边界测试比函数级 TDD 更像真正的护栏。"
hero: "/images/software-test.jpg"
tags: ["AI", "测试", "软件工程"]
layout: "../../layouts/BlogPostLayout.astro"
---

**“把测试退到边界上”**的意思是：

> 不要主要测试 AI 生成的代码“内部是怎么实现的”，而是测试系统对外暴露出来的**结果、契约和不可违反的约束**。

换句话说：

- 少测：某个函数有没有调用另一个函数
- 少测：内部是不是用了某个类、某个方法、某个 mock
- 少测：实现路径是否符合你预想的设计

而是多测：

- API 返回结果是否符合业务契约
- 数据库最终状态是否正确
- 关键业务 invariant 是否始终成立
- 用户流程是否真的跑通
- 安全、权限、审计、幂等、并发等边界条件是否被守住

---

# 一、什么叫“边界”？

这里的“边界”不是代码文件的边界，而是**系统和外部世界交互的边界**。

常见边界包括：

## 1. API 边界

比如 HTTP API、RPC 接口、GraphQL endpoint。

你不关心内部是：

```text
Controller -> Service -> Repository
```

还是：

```text
Handler -> UseCase -> DAO
```

你关心的是：

```text
POST /payments
```

之后：

- 返回值对不对
- 状态码对不对
- 数据库记录对不对
- 审计日志有没有
- 重复请求是否幂等
- 余额是否正确扣减

---

## 2. 数据边界

比如数据库、消息队列、缓存、对象存储。

你不测：

```python
PaymentService.process() 是否调用了 AuditLogger.log()
```

而是测：

```text
如果 payment 成功创建，那么 audit_logs 表中必须存在对应记录。
```

这就是从 **behavior verification** 转向 **state verification**。

---

## 3. 业务边界

比如订单、支付、权限、会员、风控。

你不测：

```text
check_access 里面是否调用了 get_user_role()
```

而是测：

```text
普通用户永远不能访问 admin_panel。
被禁用的 admin 也不能访问 admin_panel。
session 过期后任何用户都不能访问受保护资源。
```

---

## 4. 系统边界

比如端到端流程：

```text
用户注册 -> 登录 -> 下单 -> 支付 -> 发货
```

你不关心中间拆成几个模块，只关心最终系统行为是否正确。

---

# 二、为什么 AI 时代更应该这样做？

传统 TDD 经常写这种测试：

```python
def test_payment_logs_security_event(mocker):
    logger = mocker.patch("app.SecurityLogger.log")

    payment_service.process_payment(user_id=1, amount=100)

    logger.assert_called_once()
```

这个测试的问题是：它把实现路径写死了。

它隐含规定了：

- 必须有 `PaymentService`
- 必须调用 `SecurityLogger.log`
- 必须在当前函数中直接调用
- 不能通过消息队列异步写日志
- 不能换成 `AuditService`
- 不能把日志写入数据库后再批量处理

对人类团队来说，这种测试可以帮助约束模块行为。

但对 AI 来说，它会产生两个问题：

## 1. 锁死实现空间

AI 可能想重构成更合理的架构，但测试要求它必须沿着你指定的路径走。

于是 AI 的任务变成：

> 不管业务上是否合理，先让 mock 绿。

这会鼓励它迎合测试，而不是寻找更好的实现。

---

## 2. 测不到真正重要的东西

上面的测试只证明了一件事：

```text
某个 logger 方法被调用了。
```

但它没有证明：

- 日志真的写进去了
- 日志内容正确
- 日志不能被篡改
- 支付失败时是否记录失败原因
- 重复支付时是否重复记录
- 审计记录是否和 payment record 可关联

所以更好的测试是：

```python
def test_successful_payment_creates_audit_record(client, db):
    response = client.post("/payments", json={
        "user_id": 1,
        "amount": 100,
        "idempotency_key": "abc-123"
    })

    assert response.status_code == 201

    payment = db.query("SELECT * FROM payments WHERE idempotency_key = 'abc-123'")
    audit = db.query("SELECT * FROM audit_logs WHERE payment_id = ?", [payment.id])

    assert payment.status == "succeeded"
    assert audit is not None
    assert audit.event_type == "payment_succeeded"
    assert audit.signature is not None
```

这个测试不关心内部有没有 `PaymentService`，也不关心有没有 `SecurityLogger`。

它只关心：

> 成功支付以后，系统边界上的结果是否满足业务约束。

---

# 三、落地原则：从“测路径”改成“测约束”

可以用这个转换公式：

```text
不要问：这个函数有没有按我想的方式调用另一个函数？

而要问：系统最终有没有进入一个业务上正确的状态？
```

---

## 对比一：权限检查

### 不推荐：测内部调用

```python
def test_check_access_calls_role_service(mocker):
    role_service = mocker.patch("RoleService.get_role")
    check_access("user_1", "admin_panel")
    role_service.assert_called_once_with("user_1")
```

这个测试只验证调用关系。

AI 很容易为了通过测试写出符合调用链、但业务漏洞很多的代码。

---

### 推荐：测权限契约

```python
def test_normal_user_cannot_access_admin_panel(client):
    token = login_as(role="user")

    response = client.get(
        "/admin",
        headers={"Authorization": f"Bearer {token}"}
    )

    assert response.status_code == 403
```

再补充边界：

```python
def test_disabled_admin_cannot_access_admin_panel(client):
    token = login_as(role="admin", disabled=True)

    response = client.get(
        "/admin",
        headers={"Authorization": f"Bearer {token}"}
    )

    assert response.status_code == 403
```

```python
def test_expired_session_cannot_access_admin_panel(client):
    token = login_as(role="admin", session_expired=True)

    response = client.get(
        "/admin",
        headers={"Authorization": f"Bearer {token}"}
    )

    assert response.status_code == 401
```

这里测试的不是函数怎么写，而是系统必须遵守的权限 contract。

---

## 对比二：订单支付

### 不推荐：测服务调用顺序

```python
def test_order_payment_flow(mocker):
    charge = mocker.patch("PaymentGateway.charge")
    update = mocker.patch("OrderRepository.update_status")
    notify = mocker.patch("NotificationService.send")

    process_order_payment(order_id=1)

    charge.assert_called_once()
    update.assert_called_once_with(1, "paid")
    notify.assert_called_once()
```

问题是：

- 你锁死了调用顺序和模块结构
- 但没有验证最终订单状态、支付状态、通知状态是否一致
- AI 可能只是在迎合 mock

---

### 推荐：测业务不变量

```python
def test_paid_order_has_matching_payment_record(client, db):
    response = client.post("/orders/1/pay", json={
        "payment_method": "card",
        "idempotency_key": "pay-001"
    })

    assert response.status_code == 200

    order = db.get_order(1)
    payment = db.get_payment_by_order_id(1)

    assert order.status == "paid"
    assert payment.status == "succeeded"
    assert payment.amount == order.total_amount
```

更进一步，加 invariant：

```python
def test_order_cannot_be_paid_twice(client, db):
    payload = {
        "payment_method": "card",
        "idempotency_key": "pay-001"
    }

    first = client.post("/orders/1/pay", json=payload)
    second = client.post("/orders/1/pay", json=payload)

    assert first.status_code == 200
    assert second.status_code == 200

    payments = db.get_payments_by_order_id(1)

    assert len(payments) == 1
```

这里真正守住的是业务边界：

> 一个订单不能因为重复请求而被扣两次钱。

这比“有没有调用某个函数”重要得多。

---

# 四、怎么具体落地？

可以分成五层。

---

# 1. 先定义业务 invariants

**Invariant** 可以理解为：

> 无论代码怎么实现、怎么重构、怎么扩展，都必须永远成立的业务事实。

比如支付系统：

```text
1. 成功支付后，订单状态必须是 paid。
2. paid 订单必须存在一条成功的 payment record。
3. 一个 idempotency_key 只能产生一次实际扣款。
4. 支付金额必须等于订单应付金额。
5. 支付失败不能把订单标记为 paid。
6. 每次支付尝试都必须有审计记录。
7. 审计记录必须能关联到用户、订单和 payment。
```

权限系统：

```text
1. 未登录用户不能访问受保护资源。
2. 普通用户不能访问管理员资源。
3. 被禁用用户不能访问任何需要认证的资源。
4. session 过期后不能继续访问。
5. 权限变更后，旧 token 不能继续获得已撤销权限。
```

库存系统：

```text
1. 库存不能为负数。
2. 已支付订单必须成功锁定库存。
3. 取消订单必须释放库存。
4. 同一商品并发下单不能超卖。
```

这些就是边界测试的核心。

---

# 2. 把 invariant 写成可执行测试

比如库存不能为负：

```python
def test_inventory_never_goes_negative(client, db):
    db.create_product(id=1, stock=1)

    response_1 = client.post("/orders", json={
        "product_id": 1,
        "quantity": 1
    })

    response_2 = client.post("/orders", json={
        "product_id": 1,
        "quantity": 1
    })

    product = db.get_product(1)

    assert product.stock >= 0

    successful_orders = db.get_successful_orders(product_id=1)
    assert len(successful_orders) <= 1
```

注意这里不是在测某个库存扣减函数，而是在测：

> 不管内部怎么写，最终库存不能小于零，成功订单数量不能超过库存能力。

---

# 3. 优先测系统公开接口

尽量从外部入口打进去：

- HTTP API
- CLI command
- Message handler
- Job runner
- Public SDK method
- 数据导入导出接口

例如不要直接测：

```python
reserve_inventory(product_id, quantity)
```

优先测：

```python
POST /orders
```

因为真实用户不会直接调用你的内部函数。

AI 可以自由重构内部函数，只要外部契约不变即可。

---

# 4. 减少 mock，使用真实依赖或测试替身

传统 unit test 很爱 mock：

```python
mock_payment_gateway
mock_logger
mock_repository
mock_cache
```

但 mock 多了之后，测试验证的是“代码和 mock 的互动”，不是系统真实状态。

AI 很容易写出“刚好满足 mock”的代码。

更推荐：

## 数据库用真实测试库

比如：

- SQLite in-memory
- PostgreSQL test container
- 临时 schema
- transaction rollback

## 外部服务用 fake server

例如支付网关可以 fake：

```text
POST /fake-payment-gateway/charge
```

它返回固定结果，并记录请求。

这样你可以测：

```text
系统是否真的发出了正确的扣款请求？
扣款成功后本地状态是否正确？
扣款失败后是否回滚？
```

而不是只测某个方法被调用。

---

# 5. 加 property-based testing

如果 example-based test 是：

```text
给定一个具体输入，期待一个具体输出。
```

property-based testing 是：

```text
随机生成大量输入，验证某个性质始终成立。
```

比如金额计算：

```python
from hypothesis import given, strategies as st

@given(
    price=st.decimals(min_value=0, max_value=10000, places=2),
    quantity=st.integers(min_value=1, max_value=100),
    discount=st.decimals(min_value=0, max_value=1, places=2)
)
def test_total_amount_is_never_negative(price, quantity, discount):
    total = calculate_total(price, quantity, discount)

    assert total >= 0
```

对于 AI 生成代码，这种测试特别有价值，因为它不是堵一个具体 case，而是在堵一类错误。

---

# 五、一个实际落地模板

你可以给 AI 这样的任务格式：

```markdown
你可以自由实现内部结构，但必须满足以下边界契约。

## 对外 API

POST /orders/{order_id}/pay

请求：

{
  "payment_method": "card",
  "idempotency_key": "string"
}

成功响应：

{
  "order_id": "string",
  "status": "paid",
  "payment_id": "string"
}

## 必须满足的 invariants

1. 同一个 idempotency_key 重复请求不能产生多次扣款。
2. 支付成功后，order.status 必须是 paid。
3. payment.amount 必须等于 order.total_amount。
4. 支付失败时，order.status 不能变成 paid。
5. 每一次支付尝试都必须写入 audit_logs。
6. audit_logs 必须包含 user_id、order_id、payment_id、event_type、created_at。
7. 并发请求同一个订单时，最多只能有一笔成功 payment。

## 测试要求

请先为以上 invariants 写集成测试或 property-based tests。
不要测试内部函数调用。
不要使用 mock 验证某个 service 是否被调用。
可以使用 fake payment gateway，但必须验证最终数据库状态。
```

这就是“把测试退到边界上”的典型用法。

---

# 六、判断一个测试是否“退到了边界上”

你可以用下面这个 checklist。

## 好的边界测试通常满足：

- 测的是用户可见行为，或业务最终状态
- 不依赖内部类名、函数名、调用顺序
- 重构内部实现时，测试不需要大改
- 能捕捉真实业务风险
- 能防止 AI 用投机实现绕过去
- 关注 invariant，而不是 implementation detail

---

## 不好的路径测试通常有这些味道：

- 大量 `mock.assert_called_once`
- 测试里出现很多内部类名
- 一重构代码结构，测试就全红
- 测的是“有没有调用”，不是“结果是否正确”
- 测试比业务逻辑还脆弱
- AI 只要迎合 mock 就能通过

---

# 七、不是完全不要 unit test

这里不是说 unit test 没用了。

更准确的说法是：

> AI 时代，不要把 unit test 当作主要的正确性护栏。

unit test 仍然适合：

- 纯函数
- 算法逻辑
- 金额计算
- 日期计算
- parser
- serializer
- validation
- 小范围确定性逻辑

例如：

```python
def test_calculate_tax():
    assert calculate_tax(amount=100, rate=0.1) == 10
```

这种测试很好。

但对于复杂业务流程，比如：

- 支付
- 权限
- 订单
- 库存
- 审计
- 风控
- 多服务协作

更应该把主要测试放在：

- contract test
- integration test
- property-based test
- E2E invariant test

---

# 八、一个简单总结

可以这样理解：

| 传统 TDD | 边界测试 |
|---|---|
| 先写函数级测试 | 先定义业务契约 |
| 关注实现路径 | 关注最终状态 |
| 常用 mock | 尽量用真实依赖或 fake service |
| 测函数调用 | 测业务 invariant |
| 适合人类增量开发 | 更适合 AI 自由探索实现 |
| 容易锁死架构 | 允许内部重构 |
| 容易被 AI Goodhart | 更难被投机绕过 |

---

# 最实用的一句话

以后写测试时，先问自己：

> 如果 AI 把内部代码全部重构一遍，只要业务结果是对的，这个测试还应该通过吗？

如果答案是 **是**，那它大概率是一个好的边界测试。

如果答案是：

> 不行，它必须调用这个类、这个方法、这个 mock。

那你可能是在测路径，而不是在测边界。
