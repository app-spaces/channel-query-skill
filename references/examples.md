# examples

只保留当前仍有代表性的真实链路。目标是让 Agent 知道“应该怎么走”，不是维护一份会快速过期的长参数样板。

通用约束：
- 所有 Tool 调用都使用 `{ "request": { ... } }`
- 先做时间归一化，再做名称到 ID 解析，再调用正式查询
- 涉及账号、密码、登录地址、登录 IP、收款地址等敏感字段，默认不输出；必要时先脱敏
- `query_settlement_list` 当前存在过滤不稳定风险，命中结果要二次校验，不可盲信

---

## 1. 供应商重名 -> 个人汇总

### 用户问题
查小太妹 4 月 1 日到 4 月 6 日的个人汇总。

### 正确链路
1. `query_supplier_list` 解析供应商
2. 如命中多个候选，先澄清 `supplierId`
3. `query_personal_summary`

### 示例请求
```json
{
  "request": {
    "keyword": "小太妹",
    "pageNum": 1,
    "pageSize": 20
  }
}
```

```json
{
  "request": {
    "startTime": "2026-04-01",
    "endTime": "2026-04-06",
    "supplierId": "GYS10007272",
    "pageNum": 1,
    "pageSize": 100
  }
}
```

### 输出重点
- 回写实际时间范围
- 回写 `supplierId`
- 只摘 `summary` 的核心指标，不机械铺满所有字段

---

## 2. 产品名称 -> 结算列表 -> 评分

### 用户问题
查抖漫 2 月 1 日到 2 月 7 日这批结算的评分。

### 正确链路
1. `query_channel_product_list` 解析 `productId`
2. `query_settlement_list` 定位候选结算单
3. 如命中多张结算单，先澄清 `settlementNo`
4. `query_settlement_score`

### 关键点
- `query_channel_product_list` 的 `key` 支持匹配产品 ID 或产品名称，且默认只返回未删除产品
- 如果用户给的是马甲包名称或包名，先改走 `query_channel_product_package_list`，必要时再取其中的 `productId` 继续下钻
- 评分不是按产品一步直出
- 结算列表当前存在过滤不稳定风险，必须核对返回行里的 `productId/supplierId/channelChildId`
- 评分查询正式使用 `settlementNo`

---

## 2B. 马甲包名称 / 包名 -> 反查产品 -> 正式查询

### 用户问题
查包名 `bk18j` 对应的产品，并继续看产品汇总。

### 正确链路
1. `query_channel_product_package_list` 用 `key=bk18j` 解析马甲包候选
2. 如命中多个候选，先澄清 `id` 或 `productId`
3. 取目标候选里的 `productId`
4. `query_product_summary` 或其他正式查询

### 关键点
- `query_channel_product_package_list` 的 `productId` 是可选精确过滤条件，和 `key` 同传时是 `AND`
- 马甲包接口本身返回的是候选，不等于已经完成正式业务查询
- 如果用户同时要看产品候选和马甲包候选，分别调两个枚举接口，不走旧合并接口

---

## 3. settlementNo -> 列表定位 id -> 详情

### 用户问题
帮我看一下 `JS20260203225304378` 的结算详情。

### 正确链路
1. `query_settlement_list` 用 `settlementNo` 定位记录
2. 读取结算报告 `id`
3. `query_settlement_detail`

### 示例请求
```json
{
  "request": {
    "settlementNo": "JS20260203225304378",
    "pageNum": 1,
    "pageSize": 10
  }
}
```

```json
{
  "request": {
    "id": "69820bd097aa6a307a839310"
  }
}
```

### 输出重点
- 结算详情依赖 `id`，不能只靠 `settlementNo`
- 优先提炼 `promotionInfo`、`analysisData`、`financeInfo`
- detail 内附带评分信息，不等同于 `query_settlement_score` 结果

---

## 4. 供应商 -> 子渠道 -> 分配单

### 用户问题
查小太妹下面默认子渠道的分配单。

### 正确链路
1. `query_supplier_list` -> `supplierId`
2. `list_sub_channels` -> `channelChildId`
3. `query_distribute_list`

### 关键点
- 不能把“默认子渠道”直接当成 `channelChildId`
- 输出时重点看 `channelCode`、`promotionLink`、`cooperationMode`、`adSlotType`

---

## 5. 结算推广明细

### 用户问题
看这张结算单 2026-02-02 的推广明细。

### 正确链路
1. 先拿结算报告 `id`
2. `query_settlement_promotion_day`

### 示例请求
```json
{
  "request": {
    "id": "69820bd097aa6a307a839310",
    "time": "2026-02-02",
    "pageNum": 1,
    "pageSize": 50,
    "sortBy": "time",
    "sortDir": "asc"
  }
}
```

### 关键点
- 正式按天查询优先显式带 `time`
- 不带 `time` 的结果不应当作稳定依据
- `pageData.content[]` 是明细，`totalData` 是汇总

---

## 6. 支出 / 财务支付联合筛选

### 用户问题
查 4 月 1 日到 4 月 7 日的已支付结算支出。

### 正确链路
- 结算支出 -> `query_settlement_expense`
- 财务支付 -> `query_finance_payment_list`

### 示例请求
```json
{
  "request": {
    "settlementStartDate": "2026-04-01",
    "settlementEndDate": "2026-04-07",
    "approveStatus": "PAID",
    "pageNum": 1,
    "pageSize": 50
  }
}
```

### 关键点
- 时间字段使用 `settlementStartDate/settlementEndDate`
- 这两条链路的联合筛选目前相对稳定
- 输出里保留 `approveStatus`、金额字段、主键字段即可

---

## 7. 已知异常示例：结算列表过滤疑似失效

### 用户问题
查供应商 GYS10006162、产品 YC-121、状态 9、2026-02-01 到 2026-02-07 的结算列表。

### 当前结论
- 即使传了 `supplierId + productId + status + startTime/endTime`
- `query_settlement_list` 仍可能混入不匹配记录

### Agent 应对方式
1. 正常调用 `query_settlement_list`
2. 对返回行逐条校验关键过滤条件
3. 若发现明显不匹配记录，输出中明确提示“疑似后端过滤异常”
4. 保留异常样本，供后端复核

### 不应该做的事
- 不应把返回成功直接等同于过滤正确
- 不应在这种结果上继续做高置信业务结论
