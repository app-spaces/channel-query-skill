# examples

以下示例优先使用真实可执行链路，避免写成理想化的一步到位调用。

## 1. 按供应商查个人汇总

### 用户问题
查小太妹 4 月 1 日到 4 月 6 日的个人汇总。

### 正确链路
1. 先用 `query_supplier_list` 解析 `supplierId`
2. 命中多个供应商时先澄清
3. 再调用 `query_personal_summary`

### 第一步
```json
{
  "request": {
    "keyword": "小太妹",
    "pageNum": 1,
    "pageSize": 20
  }
}
```

### 第二步
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
- 回写时间范围
- 回写 `supplierId`
- 展示 `summary` 中的核心指标

---

## 2. 按产品查产品汇总

### 用户问题
查抖漫 2 月份的产品汇总。

### 正确链路
1. 先用 `list_products_and_packages` 解析 `productId`
2. 再调用 `query_product_summary`

### 第一步
```json
{
  "request": {
    "productKeyword": "抖漫",
    "pageNum": 1,
    "pageSize": 20
  }
}
```

### 第二步
```json
{
  "request": {
    "startTime": "2026-02-01",
    "endTime": "2026-02-29",
    "productId": "YC-121",
    "pageNum": 1,
    "pageSize": 50
  }
}
```

### 输出重点
- 从 `summary` 里读汇总指标
- `list[]` 主要看 `productInfo + cost + 新增 + 充值`

---

## 3. 按供应商查账号

### 用户问题
看一下小太妹的供应商账号。

### 正确链路
1. 先用 `query_supplier_list` 解析 `supplierId`
2. 再调用 `query_supplier_account`

```json
{
  "request": {
    "supplierId": "GYS10007272",
    "pageNum": 1,
    "pageSize": 20
  }
}
```

### 输出重点
- 账号 / 模板 / 域名 / 登录地址 / 状态
- `cooperationDataConfig` 原样输出，不擅自解释成别的业务口径

---

## 4. 供应商 -> 子渠道 -> 分配单

### 用户问题
查小太妹下面默认子渠道的分配单。

### 正确链路
1. `query_supplier_list` -> `supplierId`
2. `list_sub_channels` -> `channelChildId`
3. `query_distribute_list`

### 第一步
```json
{
  "request": {
    "keyword": "小太妹",
    "pageNum": 1,
    "pageSize": 20
  }
}
```

### 第二步
```json
{
  "request": {
    "supplierId": "GYS10007272",
    "keyword": "默认",
    "pageNum": 1,
    "pageSize": 20
  }
}
```

### 第三步
```json
{
  "request": {
    "supplierId": "GYS10007272",
    "channelChildId": "00004962",
    "pageNum": 1,
    "pageSize": 50
  }
}
```

### 输出重点
- 不能把“默认子渠道”直接塞给 `channelChildId`
- 展示 `channelCode`、`promotionLink`、`cooperationMode`、`adSlotType`、`decisionMaker`

---

## 5. 按产品查评分

### 用户问题
查抖漫 2 月 1 日到 2 月 7 日这批结算的评分。

### 正确链路
1. `list_products_and_packages` 解析 `productId`
2. `query_settlement_list` 定位结算单
3. `query_settlement_score` 查评分

### 第一步
```json
{
  "request": {
    "productKeyword": "抖漫",
    "pageNum": 1,
    "pageSize": 20
  }
}
```

### 第二步
```json
{
  "request": {
    "startTime": "2026-02-01T00:00:00",
    "endTime": "2026-02-07T23:59:59",
    "productId": "YC-121",
    "pageNum": 1,
    "pageSize": 20,
    "sortBy": "createdAt",
    "sortDir": "desc"
  }
}
```

### 第三步
```json
{
  "request": {
    "settlementNo": "JS20260203225304378",
    "createdDateStart": "2026-02-01",
    "createdDateEnd": "2026-02-07",
    "pageNum": 1,
    "pageSize": 20
  }
}
```

### 输出重点
- 评分是按结算单查，不是按产品一步直出
- `RateDTO` 要保留 `value + score`
- 如同时间段命中多张结算单，要先澄清目标 `settlementNo`

---

## 6. 按结算单查详情

### 用户问题
帮我看一下 `JS20260203225304378` 的结算详情。

### 正确链路
1. 先用 `query_settlement_list` 通过 `settlementNo` 定位记录
2. 拿到结算报告 `id`
3. 再调用 `query_settlement_detail`

### 第一步
```json
{
  "request": {
    "settlementNo": "JS20260203225304378",
    "pageNum": 1,
    "pageSize": 10
  }
}
```

### 第二步
```json
{
  "request": {
    "id": "69820bd097aa6a307a839310"
  }
}
```

### 输出重点
- 结算详情必须带 `id`
- 重点看 `promotionInfo`、`analysisData`、`financeInfo`、`approvals`

---

## 7. 按结算单查推广明细

### 用户问题
看这张结算单 2026-02-02 的推广明细。

### 正确链路
先拿到结算报告 `id`，再查推广按天明细。

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

如需实时明细：

```json
{
  "request": {
    "id": "69820bd097aa6a307a839310",
    "pageNum": 1,
    "pageSize": 50,
    "sortBy": "time",
    "sortDir": "asc"
  }
}
```

### 输出重点
- `pageData.content[]` 是明细
- `totalData` 是汇总
- 正式按天查询建议优先传 `time`

---

## 8. 按时间段查结算支出

### 用户问题
查 4 月 1 日到 4 月 7 日的已支付结算支出。

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

### 输出重点
- 展示 `approveStatus`
- 展示金额字段：系统结算、实际结算、财务打款
- 区分这不是 `query_finance_payment_list`

---

## 9. 按时间段查财务支出列表

### 用户问题
查 4 月份财务已支付记录。

```json
{
  "request": {
    "settlementStartDate": "2026-04-01",
    "settlementEndDate": "2026-04-30",
    "approveStatus": "PAID",
    "pageNum": 1,
    "pageSize": 50
  }
}
```

### 输出重点
- `query_finance_payment_list` 比 `query_settlement_expense` 多了财务支付视角字段
- 重点展示 `channelSettlementReportId`、`settlementSysUserId`、`remarksUrls`
