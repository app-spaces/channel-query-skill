# field-notes

## 1. 高频 ID 口径

- `supplierId`：供应商 ID，常见前缀 `GYS`
- `productId`：产品 ID，常见形态如 `YC-001`、`BF-001`、`SF-001`、`DX-002`
- 产品枚举默认走 `query_channel_product_list`，`key` 支持匹配产品 ID 或产品名称，且默认只看未删除产品
- 马甲包枚举默认走 `query_channel_product_package_list`，`key` 支持匹配马甲包名称或包名；如已知 `productId`，可再叠加精确过滤
- `channelChildId`：子渠道 ID，必须依赖 `supplierId` 先枚举
- `ownerId`：决策负责人 ID，来自 `list_decision_owners`
- 结算详情 / 推广明细使用结算报告 `id`
- 结算评分使用 `settlementNo`

## 2. 时间字段口径

### 2.1 汇总类
`query_personal_summary` / `query_product_summary`
- `startTime`
- `endTime`
- 格式：`yyyy-MM-dd`

### 2.2 结算列表
`query_settlement_list`
- `starTime` / `stopTime`
- `approvalStarTime` / `approvalStopTime`
- `payStarTime` / `payStopTime`
- 格式：`yyyy-MM-ddTHH:mm:ss`

### 2.3 结算支出 / 财务支出
`query_settlement_expense` / `query_finance_payment_list`
- `settlementStartDate`
- `settlementEndDate`
- 格式：`yyyy-MM-dd`

### 2.4 结算评分
`query_settlement_score`
- `createdDateStart`
- `createdDateEnd`
- 格式：`yyyy-MM-dd`

### 2.5 推广明细
- `query_settlement_promotion_day.time`：`yyyy-MM-dd`
- `query_settlement_promotion_realtime.newStopTime`：`yyyy-MM-dd`

## 3. 通用返回壳

### 3.1 多数列表 Tool
- `list`
- `total`
- `pageNum`
- `pageSize`

### 3.2 `query_settlement_list`
- `code`
- `msg`
- `status`
- `time`
- `data`

说明：真正的分页对象在 `data` 里。

## 4. 状态与枚举口径

### 4.1 结算状态
当前已知：
- `0 = 待结算`
- `1 = 审核中`
- `8 = 通过`
- `9 = 驳回`
- `10 = 不结算`
- `11 = 已打款`

### 4.2 支付状态
`query_settlement_expense` / `query_finance_payment_list`
- `PENDING_PAY`
- `PAID`
- `CLOSE`

### 4.3 分配单状态
`query_distribute_list`
- `0 = 待分配`
- `1 = 已分配`
- `2 = 已关闭`

### 4.4 合作模式
常见枚举：
- `CPA`
- `CPC`
- `CPM`
- `CPT`
- `CPI`
- `CPS`
- `CPCCPA`

### 4.5 供应商类型
`query_supplier_list.type`
- `ALLIANCE`
- `KOL`
- `WEBMASTER`
- `AGENT`
- `HORN`
- `BOX`
- `DOWNWEB`
- `PLAYER`
- `OTHER`

### 4.6 产品模式
`query_settlement_list.model`
- `FREE`
- `PAID`
- `SEMI_PAID`

### 4.7 广告位类型
`query_distribute_list.adSlotType`
- `TYPE_1`：文字广告
- `TYPE_2`：Banner
- `TYPE_3`：弹窗
- `TYPE_4`：启动页
- `TYPE_5`：图标
- `TYPE_6`：其他

## 5. 汇总与分析字段口径

### 5.1 个人汇总默认对外口径
适用范围：`query_personal_summary`

- `list[].costUnitCny`：个人汇总默认“成本”口径
- `list[].recycleRatio`：个人汇总默认“回收比”口径
- `list[].roi`：个人汇总默认“ROI”口径
- `list[].profitCny`：个人汇总默认“盈亏CNY”口径
- `list[].payAmountCny`：个人汇总默认“打款金额CNY”口径
- `list[].cost`：原始返回中的成本相关字段，不作为默认“个人汇总成本”口径；除非用户明确指定要看 `cost`
- `summary.costUnitCnySum`：个人汇总层“成本”口径，按去重渠道平均
- `summary.recycleRatioSum`：个人汇总层“回收比”口径，按去重渠道平均
- `summary.roi`：接口可能返回，但从 Skill 层默认屏蔽，不作为汇总输出字段
- `summary.profitCnySum`：个人汇总层盈亏CNY口径
- `summary.payAmountCnySum`：个人汇总层打款金额CNY口径

### 5.2 结算分析字段公式
- `analysisData.costUnitPrice = 系统结算金额(CNY) / 新增(原始值)`
- `analysisData.recycleRatio = VIP充值(原始值)(CNY) / 系统结算金额(CNY)`
- `analysisData.profitPrice = VIP充值(CNY) - 系统结算金额(CNY)`

### 5.3 评分字段
`query_settlement_score` 中：
- `secondDayRetention`
- `viewRate`
- `recycleRatio`
- `adClickRate`
- `costUnitPrice`
- `sevenDayRetention`

以上字段都是 `RateDTO`，统一读取：
- `value`
- `score`

### 5.4 Deducted 字段口径
- 响应字段名中只要包含 `Deducted`，统一按“扣量后指标”解释
- 含义：该指标已经扣除一定比例，不是原始值
- 对外表述建议写成：扣量后新增、扣量后金额、扣量后留存等
- 若同一响应里同时存在原始字段与 `Deducted` 字段，禁止混算、混比、混解释

## 6. 特殊结构

### 6.1 `cooperationDataConfig`
出现于：`query_supplier_account`

结构：`Map<CooperationMode, List<DataConfig>>`

用途：供应商账号的合作模式数据配置。

### 6.2 `dataMap`
出现于：`query_settlement_list`，真实返回中 `query_settlement_detail` 也可能带出。

结构：`Map<String,Object>`

当前已知已使用键：
- `KEY_STOP_USER`

### 6.3 扣量时间段配置
出现于：`query_distribute_list` 的 `deductionConfig`

`DeductionConfig.TimeRangeRule.startTime/endTime` 结构：
- `Map<String,Integer>`

推荐键：
- `hour`
- `minute`
- `second`

## 7. 真实返回解释规则

1. 汇总类返回中的 `queryStartDate` / `queryEndDate` 可视为展示字段，不要单凭它们与请求范围不完全一致就判定筛选失败。
2. `query_settlement_detail` 返回的是单对象，不是分页。
3. `query_settlement_promotion_day` / `query_settlement_promotion_realtime` 返回的是 `PromotionInfoDTO`，重点看 `pageData`、`totalData`、`status`、`newUserStatList`。
4. `query_finance_payment_list` 与 `query_settlement_expense` 都是支出查询，但一个偏财务支付视角，一个偏结算支付视角。
5. `detail.analysisData.scoreInfo` 是结算详情附带评分信息；如用户明确要查评分明细，以 `query_settlement_score` 为准。

## 8. 已停用 / 兼容保留接口

- `query_toll_and_free_score` 已停用
- 不再作为默认评分入口
- 按产品查评分时，统一走：产品解析 -> `query_settlement_list` -> `query_settlement_score`
- `list_products_and_packages` 仅作为兼容保留的旧合并接口存在时参考，Skill 默认不走它
- 涉及产品 / 马甲包解析时，统一优先走独立的 `query_channel_product_list` / `query_channel_product_package_list`
