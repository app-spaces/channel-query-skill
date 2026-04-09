# test-cases

面向真实 Agent 执行的验收清单。只保留当前还需要稳定守住的能力点、约束和已知风险。

## 0. 预置条件

1. MCP Server 已接入并可正常 `tools/list`
2. Skill 已加载
3. 所有 Tool 都按 `{ "request": { ... } }` 调用
4. `query_toll_and_free_score` 已停用，不再走旧评分入口
5. 产品 / 马甲包解析默认走 `query_channel_product_list` / `query_channel_product_package_list`，不再走 `list_products_and_packages` 旧合并接口
6. 涉及敏感字段的输出默认脱敏，不把账号密码、登录地址、登录 IP、收款地址直接写进总结

---

## 1. 核心链路验收

### TC-01 供应商名称 -> 个人汇总
- 输入：查小太妹 4 月 1 日到 4 月 6 日的个人汇总
- 期望：`query_supplier_list` -> `query_personal_summary`
- 检查点：输出回写 `supplierId`、时间范围、分页、summary

### TC-02A 产品名称 -> 产品汇总
- 输入：查抖漫 2 月份产品汇总
- 期望：`query_channel_product_list` -> `query_product_summary`
- 检查点：不把中文产品名直接塞给正式查询；产品枚举默认只看未删除产品

### TC-02B 马甲包名称 / 包名 -> 马甲包枚举
- 输入：查某个马甲包或包名对应的产品信息
- 期望：`query_channel_product_package_list`
- 检查点：只回写必要字段 `id/productId/name/packageName/productName`，不机械回显原始对象

### TC-02C 马甲包 -> productId -> 正式查询
- 输入：先给马甲包名称或包名，再继续查产品汇总或结算
- 期望：先 `query_channel_product_package_list`，再取 `productId` 进入正式查询
- 检查点：不再走 `list_products_and_packages` 旧合并接口

### TC-03 供应商 -> 子渠道 -> 分配单
- 输入：查小太妹下面默认子渠道的分配单
- 期望：`query_supplier_list` -> `list_sub_channels` -> `query_distribute_list`
- 检查点：不能把“默认子渠道”直接当 `channelChildId`

### TC-04 产品 -> 结算列表 -> 评分
- 输入：查抖漫 2 月 1 日到 2 月 7 日这批结算的评分
- 期望：`query_channel_product_list` -> `query_settlement_list` -> `query_settlement_score`
- 检查点：多结算单时先澄清 `settlementNo`

### TC-05 settlementNo -> 详情
- 输入：查 `JS20260203225304378` 的结算详情
- 期望：先 `query_settlement_list` 定位 `id`，再 `query_settlement_detail`
- 检查点：详情查询不能只用 `settlementNo`

### TC-06 settlementNo / id -> 推广明细
- 输入：查这张结算单 2026-02-02 的推广明细
- 期望：拿到 `id` 后调用 `query_settlement_promotion_day`
- 检查点：正式按天查询优先显式传 `time`

### TC-07 时间段 -> 结算支出
- 输入：查 4 月 1 日到 4 月 7 日的已支付结算支出
- 期望：走 `query_settlement_expense`
- 检查点：时间字段为 `settlementStartDate/settlementEndDate`

### TC-08 时间段 -> 财务支出
- 输入：查 4 月财务已支付记录
- 期望：走 `query_finance_payment_list`
- 检查点：不要误走 `query_settlement_expense`

### TC-09 供应商 -> 账号
- 输入：查某供应商的账号
- 期望：先解析 `supplierId`，再 `query_supplier_account`
- 检查点：总结输出默认脱敏，不直接暴露敏感字段

---

## 2. 参数与格式验收

### TC-10 request 包裹
- 检查点：每次调用最外层都必须是 `request`

### TC-11 汇总时间格式
- 检查点：`query_personal_summary` / `query_product_summary` 使用 `yyyy-MM-dd`

### TC-12 结算列表时间格式
- 检查点：`query_settlement_list` 使用 `yyyy-MM-ddTHH:mm:ss`

### TC-13 支出 / 财务时间格式
- 检查点：`query_settlement_expense` / `query_finance_payment_list` 使用 `settlementStartDate/settlementEndDate`，格式 `yyyy-MM-dd`

### TC-14 评分时间格式
- 检查点：`query_settlement_score` 使用 `createdDateStart/createdDateEnd`

### TC-15 推广按天时间格式
- 检查点：`query_settlement_promotion_day.time` 使用 `yyyy-MM-dd`

---

## 3. 歧义与守卫验收

### TC-16 供应商重名
- 期望：必须先澄清，不取第一条

### TC-17 产品重名
- 期望：必须先澄清 `productId`

### TC-18 子渠道多命中
- 期望：必须先澄清 `channelChildId`

### TC-19 同时间段多结算单
- 期望：必须先澄清 `settlementNo`

### TC-20 promotion_day 不带 time
- 期望：不把结果当稳定依据；正式查询优先补 `time`

### TC-21 detail 内评分 vs score 接口
- 期望：明确两者不是同一口径，不能混用

---

## 4. 已知异常回归

### TC-22 query_settlement_list 过滤异常识别
- 输入：`supplierId + productId + status + startTime/endTime` 的联合筛选
- 期望：即使接口返回成功，也要核对返回行是否真的满足过滤条件
- 检查点：发现不匹配记录时，输出中要显式提示“疑似后端过滤异常”

### TC-23 文档字段 vs 实际生效字段
- 输入：对比 `startTime/endTime` 与 `starTime/stopTime`
- 期望：记录真实生效字段，不盲信旧文档字段

### TC-24 返回壳差异
- 输入：`query_settlement_list` 与普通分页 Tool 对比
- 期望：正确解析 `code/msg/status/time/data`

---

## 5. 最小回归集

每次改 Skill 后，至少回归以下 10 条：
1. 供应商 -> 个人汇总
2. 产品枚举 -> 产品汇总
3. 马甲包枚举 -> `productId` -> 正式查询
4. 供应商 -> 子渠道 -> 分配单
5. 产品 -> 结算列表 -> 评分
6. settlementNo -> 详情
7. settlementNo / id -> promotion_day（带 `time`）
8. 时间段 -> 结算支出
9. 时间段 -> 财务支出
10. 敏感字段默认脱敏

如本次修改涉及 `query_settlement_list` 路由或过滤解释，额外补回归：
- 联合筛选异常识别
- `startTime/endTime` vs `starTime/stopTime`
