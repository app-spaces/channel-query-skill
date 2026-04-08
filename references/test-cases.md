# test-cases

面向真实 Agent 执行的验收清单。目标不是验证“文档是否好看”，而是验证 Skill 能不能把自然语言稳定转成正确的 MCP 调用。

## 0. 预置条件

1. MCP Server 已接入并可正常 `tools/list`
2. Skill 已加载
3. 所有 Tool 都按 `{ "request": { ... } }` 调用
4. 已知 `query_toll_and_free_score` 停用，不再走旧评分入口

## 1. 核心路由验收

### TC-01 供应商名称 -> 个人汇总
- 输入：查小太妹 4 月 1 日到 4 月 6 日的个人汇总
- 期望：先 `query_supplier_list`，再 `query_personal_summary`
- 检查点：输出要回写 `supplierId`、时间范围、分页、summary

### TC-02 产品名称 -> 产品汇总
- 输入：查抖漫 2 月份产品汇总
- 期望：先 `list_products_and_packages`，再 `query_product_summary`
- 检查点：输出读 `summary`，不把产品中文名直接塞给正式查询

### TC-03 供应商 -> 账号
- 输入：查小太妹的供应商账号
- 期望：先解析 `supplierId`，再 `query_supplier_account`
- 检查点：能输出账号、模板、域名、登录地址、状态

### TC-04 供应商 -> 子渠道 -> 分配单
- 输入：查小太妹下面默认子渠道的分配单
- 期望：`query_supplier_list` -> `list_sub_channels` -> `query_distribute_list`
- 检查点：不能把“默认子渠道”直接当 `channelChildId`

### TC-05 产品 -> 结算单列表 -> 评分
- 输入：查抖漫 2 月 1 日到 2 月 7 日这批结算的评分
- 期望：`list_products_and_packages` -> `query_settlement_list` -> `query_settlement_score`
- 检查点：评分不是按产品一步直出；多结算单时先澄清 `settlementNo`

### TC-06 settlementNo -> 详情
- 输入：查 `JS20260203225304378` 的结算详情
- 期望：先 `query_settlement_list` 定位 `id`，再 `query_settlement_detail`
- 检查点：详情查询不能只用 `settlementNo`

### TC-07 settlementNo / id -> 推广明细
- 输入：查这张结算单 2026-02-02 的推广明细
- 期望：拿到 `id` 后调用 `query_settlement_promotion_day`
- 检查点：正式按天查询建议显式传 `time`

### TC-08 时间段 -> 结算支出
- 输入：查 4 月 1 日到 4 月 7 日的已支付结算支出
- 期望：走 `query_settlement_expense`
- 检查点：时间字段为 `settlementStartDate/settlementEndDate`

### TC-09 时间段 -> 财务支出
- 输入：查 4 月财务已支付记录
- 期望：走 `query_finance_payment_list`
- 检查点：不要误走 `query_settlement_expense`

## 2. 参数与格式验收

### TC-10 所有 Tool 的 request 包裹
- 检查点：每次调用最外层都必须是 `request`

### TC-11 汇总时间格式
- 检查点：`query_personal_summary` / `query_product_summary` 使用 `yyyy-MM-dd`

### TC-12 结算列表时间格式
- 检查点：`query_settlement_list` 使用 `yyyy-MM-ddTHH:mm:ss`

### TC-13 结算支出 / 财务支出时间格式
- 检查点：使用 `settlementStartDate/settlementEndDate`，格式 `yyyy-MM-dd`

### TC-14 评分时间格式
- 检查点：`query_settlement_score` 使用 `createdDateStart/createdDateEnd`

### TC-15 推广按天时间格式
- 检查点：`query_settlement_promotion_day.time` 使用 `yyyy-MM-dd`

## 3. 输出解释验收

### TC-16 汇总类字段解释
- 检查点：`costUnitCnySum`、`recycleRatioSum` 按去重渠道平均，不写成简单求和

### TC-17 结算列表返回壳解释
- 检查点：从 `code/msg/status/time/data` 中读取真正分页数据，不误把顶层当列表

### TC-18 评分字段解释
- 检查点：`secondDayRetention/viewRate/recycleRatio/adClickRate/costUnitPrice/sevenDayRetention` 统一按 `RateDTO.value + score` 输出

### TC-19 特殊结构解释
- 检查点：
  - `cooperationDataConfig` 解释为 `Map<CooperationMode, List<DataConfig>>`
  - `dataMap` 解释为结算扩展字段
  - 扣量时间段规则解释为 `Map<String,Integer>`
  - `adSlotType` 按 `TYPE_1 ~ TYPE_6` 的新枚举含义输出

### TC-20 汇总类附带日期字段
- 检查点：若 `queryStartDate/queryEndDate` 与请求范围不完全一致，不直接判定筛选失效

## 4. 歧义处理验收

### TC-21 供应商重名
- 期望：必须先澄清，不取第一条

### TC-22 产品重名
- 期望：必须先澄清 `productId`

### TC-23 子渠道多命中
- 期望：必须先澄清 `channelChildId`

### TC-24 同时间段多结算单
- 期望：必须先澄清 `settlementNo`

## 5. 最小回归集

每次改 Skill 后至少回归以下 5 条：
1. 供应商 -> 个人汇总
2. 产品 -> 产品汇总
3. 供应商 -> 子渠道 -> 分配单
4. settlementNo -> 详情
5. 产品 -> 结算单 -> 评分
