# tool-mapping

## 1. 总规则

### 1.1 请求包裹规则
所有 Tool 都必须使用：

```json
{
  "request": {
    "...": "..."
  }
}
```

### 1.2 通用分页返回壳
多数列表 Tool 返回：
- `list`
- `total`
- `pageNum`
- `pageSize`

### 1.3 `query_settlement_list` 的特殊返回壳
不是直接返回分页壳，而是：
- `code`
- `msg`
- `status`
- `time`
- `data`（里面才是分页对象）

其中：
- `code=200` 表示成功
- `data.list` / `data.total` / `data.pageNum` / `data.pageSize` 才是分页主数据

---

## 2. 枚举与解析 Tool

### 2.1 `list_approval_user_dept`
用途：查询部门枚举，把部门名称解析成 `secondDeptId`。

#### 输入参数
- 仅要求最外层存在 `request`
- 当前可传空对象：`{"request": {}}`

#### 输出重点
顶层：`list/total`

`list[]` 为 `DropDown`，但字段语义较特殊：
- `label`：部门 ID，也就是应回写的 `secondDeptId`
- `value`：部门名称
- `children`

#### 典型用途
- 部门名称 -> `secondDeptId`
- 为“某部门汇总”“某部门下组员”“某部门下某组员汇总”提供前置 ID

---

### 2.2 `list_decision_member`
用途：查询部门下组员枚举，把组员展示名解析成 `memberId`。

#### 输入参数
- `roleId`，部门 ID，也就是 `secondDeptId`

#### 输出重点
顶层：`list/total`

`list[]` 为 `DropDown`：
- `label`：组员展示名，常见格式如 `lsz4311-李适之`
- `value`：组员 ID，也就是应回写的 `memberId`
- `children`

#### 典型用途
- `secondDeptId` + 组员名称 -> `memberId`
- 为“某部门下某组员”的汇总查询提供前置 ID

---

### 2.3 `list_decision_owners`
用途：查询决策负责人枚举。

#### 输入参数
- `roleId`，可选，不传则按当前用户权限返回

#### 输出重点
- `list`
- `total`

`list[]` 为 `DropDown`：
- `label`
- `value`
- `children`

#### 典型用途
- 负责人名称 -> `ownerId`
- 有树形结构时优先取叶子或明确选中项的 `value`
- 注意不要与 `list_decision_member` 混用

---

### 2.4 `query_supplier_list`
用途：查供应商列表，或把供应商名称解析成 `supplierId`。

#### 输入参数
- `pageNum`
- `pageSize`
- `keyword`，供应商 ID / 名称关键字
- `id`，精确供应商 ID
- `name`，供应商名称
- `status`
- `type`，支持：`ALLIANCE` / `KOL` / `WEBMASTER` / `AGENT` / `HORN` / `BOX` / `DOWNWEB` / `PLAYER` / `OTHER`

#### 输出重点
顶层：`list/total/pageNum/pageSize`

`list[]` 重点字段：
- `id`
- `name`
- `type`
- `status`
- `domainOrPackage`
- `contacts`
- `payType`
- `paymentAccount`
- `rating`
- `isBlacklisted`
- `blacklistReason`
- `memberName`
- `nickName`
- `accountBalance`
- `cooperationStartDate`
- `cooperationProductCount`
- `settlementTimes`
- `totalPaymentAmount`
- `totalVolume`
- `totalRevenue`
- `totalROI`
- `recent30DaysROI`
- `createdAt` / `updatedAt`

`contacts[]`：
- `contactType`
- `contactInfo`
- `updateTime`

#### 典型用途
- 名称 -> `supplierId`
- 校验供应商状态或类型
- 为 `list_sub_channels`、汇总、分配单、账号、结算类查询提供 `supplierId`

---

### 2.5 `list_sub_channels`
用途：在已知 `supplierId` 的前提下，查询子渠道。

#### 输入参数
- `pageNum`
- `pageSize`
- `supplierId`，必填
- `keyword`

#### 输出重点
顶层：`list/total/pageNum/pageSize`

`list[]`：
- `id`
- `supplierId`
- `name`
- `createdAt` / `updatedAt`

#### 典型用途
- `supplierId` -> `channelChildId`
- 不能跳过 `supplierId` 直接查子渠道

---

### 2.6 `query_channel_product_list`
用途：产品枚举查询，优先用于把产品名称或产品 ID 关键字解析成 `productId`。

#### 输入参数
- `pageNum`
- `pageSize`
- `key`，模糊匹配产品 ID 或产品名称

#### 输出要点
- 正式查询使用 `list[].productId` 作为 `productId`
- 默认只返回未删除产品
- 对用户只优先回写必要字段：`productId`、`name`、`status`、`type`、`model`
- 原始返回可能带 `logoUrl`、部门、外部同步字段，Skill 不应直接机械回显整行对象

#### 典型用途
- 产品名称 -> `productId`
- 产品 ID 关键字 -> 产品候选列表
- 为产品汇总、结算、分配单等正式查询提供稳定的 `productId`

---

### 2.7 `query_channel_product_package_list`
用途：马甲包枚举查询，优先用于把马甲包名称 / 包名解析成马甲包候选，并在需要时反查 `productId`。

#### 输入参数
- `pageNum`
- `pageSize`
- `key`，模糊匹配马甲包名称或马甲包标识（包名）
- `productId`，可选；精确过滤，和 `key` 同时传时为 `AND`

#### 输出要点
- 使用 `list[].id` 作为马甲包记录标识
- 下游正式查询仍优先使用 `list[].productId`
- 对用户只优先回写必要字段：`id`、`productId`、`name`、`packageName`、`productName`
- 若返回里附带 `logoUrl`，可在候选展示时保留，但不要把整行原始对象直接外抛

#### 典型用途
- 马甲包名称 / 包名 -> 马甲包候选
- 马甲包名称 / 包名 -> `productId` -> 继续做产品汇总、结算、分配单等正式查询
- 已知 `productId` 时，查询该产品下的马甲包列表

---

## 3. 汇总 Tool

### 3.1 `query_personal_summary`
用途：按个人维度查询汇总。

#### 输入参数
- `pageNum`
- `pageSize`
- `secondDeptId`
- `memberId`
- `ownerId`
- `supplierId`
- `subChannelId`
- `productId`
- `channelCodeList`，支持单个或逗号分隔多个
- `channelCode`，`channelCodeList` 的单值别名
- `startTime`，`yyyy-MM-dd`
- `endTime`，`yyyy-MM-dd`

补充说明：
- 部门过滤用 `secondDeptId`
- 组员过滤用 `memberId`
- 决策负责人过滤用 `ownerId`
- “某部门下某组员”优先组合传 `secondDeptId + memberId`

#### 输出重点
顶层：
- `list`
- `total`
- `pageNum`
- `pageSize`
- `summary`

`list[]` 重点字段：
- `queryDateRange`
- `queryStartDate`
- `queryEndDate`
- `channelCode`
- `cooperationMode`
- `supplierInfo`：`supplierId/supplierName`
- `subChannelInfo`：`subChannelId/subChannelName`
- `productInfo`：`productId/productName/productType/logoUrl`
- `ownerInfo`：`ownerId/ownerName/username`
- `cost`
- `payAmountCny`
- `profitCny`
- `costUnitCny`
- `recycleRatio`
- `roi`
- `landingPageViews`
- `landingPageClicks`
- `uniqueClicks`
- `channelClicks`
- `downloadCount`
- `downloadCountDeducted`
- `installCount`
- `installCountDeducted`
- `newUsersRaw`
- `newUsersDeducted`
- `rechargeUserCount`
- `vipRechargeAmount`
- `vipRechargeAmountDeducted`
- `coinRechargeAmount`
- `updatedAt`

`summary` 重点字段：
- `costSum`
- `landingPageViewsSum`
- `landingPageClicksSum`
- `uniqueClicksSum`
- `channelClicksSum`
- `downloadCountSum`
- `downloadCountDeductedSum`
- `installCountSum`
- `installCountDeductedSum`
- `newUsersRawSum`
- `newUsersDeductedSum`
- `rechargeUserCountSum`
- `vipRechargeAmountSum`
- `vipRechargeAmountDeductedSum`
- `coinRechargeAmountSum`
- `payAmountCnySum`
- `profitCnySum`
- `costUnitCnySum`
- `recycleRatioSum`
- `roi`（接口可能返回，但从 Skill 层默认屏蔽，不作为汇总输出字段）

#### 对外默认口径
适用范围：`query_personal_summary`

- 成本：默认使用 `costUnitCny`；汇总层对应 `summary.costUnitCnySum`
- 回收比：默认使用 `recycleRatio`；汇总层对应 `summary.recycleRatioSum`
- ROI：默认使用 `list[].roi` 作为单渠道 / 单记录口径；`summary.roi` 从 Skill 层默认屏蔽
- 盈亏CNY：默认使用 `profitCny`；汇总层对应 `summary.profitCnySum`
- 打款金额CNY：默认使用 `payAmountCny`；汇总层对应 `summary.payAmountCnySum`
- `cost` 仅作为原始返回中的成本相关字段保留，不作为默认“个人汇总成本”口径；除非用户明确指定

#### 典型用途
- 个人维度投放 / 转化 / 充值 / 盈亏汇总
- 部门 / 组员 / 决策负责人 / 供应商 / 子渠道 / 产品多条件组合筛选

---

### 3.2 `query_product_summary`
用途：按产品维度查询汇总。

#### 输入参数
与 `query_personal_summary` 完全一致，包括 `secondDeptId`、`memberId`、`ownerId` 三类人员维度过滤。

#### 输出重点
顶层：`list/total/pageNum/pageSize/summary`

`list[]` 重点字段：
- `productInfo`：`productId/productName/productType/logoUrl`
- `cost`
- `landingPageViews`
- `landingPageClicks`
- `uniqueClicks`
- `channelClicks`
- `downloadCount`
- `downloadCountDeducted`
- `installCount`
- `installCountDeducted`
- `newUsersRaw`
- `newUsersDeducted`
- `rechargeUserCount`
- `vipRechargeAmount`
- `vipRechargeAmountDeducted`
- `coinRechargeAmount`

`summary` 与 `query_personal_summary` 相同。

#### 典型用途
- 产品层面对比
- 产品汇总后再决定是否继续下钻结算、分配单或评分

---

## 4. 供应商账号 Tool

### 4.1 `query_supplier_account`
用途：查询供应商账号。

#### 输入参数
- `pageNum`
- `pageSize`
- `keyword`，供应商 ID / 账号关键字
- `status`
- `supplierId`

#### 输出重点
顶层：`list/total/pageNum/pageSize`

`list[]` 重点字段：
- `id`
- `account`
- `password`
- `supplierId`
- `supplierName`
- `channelChildId`
- `channelChildIds`
- `channelChildName`
- `templateId`
- `templateName`
- `domain`
- `previewImage`
- `templateEnabled`
- `loginUrl`
- `startTime`
- `status`
- `channelCodes`
- `loginIp`
- `loginDate`
- `cooperationDataConfig`
- `createdAt` / `updatedAt`

#### 典型用途
- 供应商账号运营查询
- 检查子渠道、模板、域名、登录信息、配置项

---

## 5. 结算与财务 Tool

### 5.1 `query_settlement_list`
用途：结算详情列表查询，也是很多结算下钻查询的入口。

#### 输入参数
- `pageNum`
- `pageSize`
- `sortBy`，支持 `costUnitPrice`、`actualCostUnitPrice` 等
- `sortDir`，`asc/desc`
- `key`
- `supplierKey`
- `productKey`
- `settlementNo`
- `status`
- `type`
- `cooperationMode`
- `model`，`FREE/PAID/SEMI_PAID`
- `starTime` / `stopTime`
- `approvalStarTime` / `approvalStopTime`
- `payStarTime` / `payStopTime`
- `isSelfAudit`
- `supplierId`
- `channelChildId`
- `productId`

#### 输出重点
顶层壳：
- `code`
- `msg`
- `status`
- `time`
- `data`

`data`：
- `list`
- `total`
- `pageNum`
- `pageSize`

`data.list[]` 重点字段：
- `id`
- `settlementNo`
- `type`
- `status`
- `settlementUSDTMoney`
- `settlementCNYMoney`
- `supplierId`
- `productId`
- `channelChildId`
- `promotionInfo`
- `promotionData`
- `analysisData`
- `financeInfo`
- `applyUser`
- `approvalReportInfos`
- `isShowApproval`
- `approvalSubmitTime`
- `approvalTime`
- `payTime`
- `dataMap`
- `supplier`
- `product`
- `configStartTime`
- `configEndTime`
- `createdAt`
- `updatedAt`

`promotionInfo`：
- `startTime`
- `stopTime`
- `cooperationMode`
- `promotionCode`
- `cooperationPrices`

`promotionData` 常见字段：
- `downloadNNumber/downloadNumber`
- `clickDownloadNNumber/clickDownloadNumber`
- `onlyClickNNumber/onlyClickNumber`
- `addNNumber/addNumber`
- `rechargeNNumber/rechargeNumber`
- `rechargeNMoney/rechargeMoney`
- `rechargeNGold/rechargeGold`
- `showNNum/showNum`
- `installNNum/installNum`
- `clickNum/clickNumFinalValue`

`analysisData`：
- `costUnitPrice`
- `costUnitPriceFinalValue`
- `nextDayRetentionRate`
- `scoreInfo`：`score/renew/solutions`
- `recycleRatio`
- `profitPrice`
- `actualCostUnitPrice`
- `actualRecycleRatio`
- `actualProfitPrice`

`financeInfo`：
- `selectSettlementInfos`
- `selectIndex`
- `settlementUSDTMoney`
- `settlementCNYMoney`
- `sysSettlementUSDTMoney`
- `sysSettlementCNYMoney`
- `remark`
- `payType`
- `paymentAccount`
- `files`
- `exchangeRate`

#### 典型用途
- 用 `settlementNo` 或产品 / 供应商 / 状态定位结算记录
- 为详情、推广明细、评分查询提供入口信息

---

### 5.2 `query_settlement_detail`
用途：查询单个结算详情。

#### 输入参数
- `id`，必填，结算报告 ID
- `newStopTime`
- `selectIndex`

#### 输出重点
返回 `ChannelSettlementReportInfoDTO`，可视为 `query_settlement_list` 单条记录的完整展开版，并新增：
- `approvals`
- `isOldReport`
- `isCancel`

`approvals[]`：
- `id`
- `settlementNo`
- `remark`
- `type`
- `state`
- `approvalUser`
- `details`
- `createdAt`
- `updatedAt`

#### 典型用途
- 查看单张结算单详情、审批轨迹、财务方案、推广信息

---

### 5.3 `query_settlement_promotion_day`
用途：查询结算单按天或按小时拆分的推广明细。

#### 输入参数
- `id`，必填
- `time`，可选，单日 `yyyy-MM-dd`
- `pageNum`
- `pageSize`
- `sortBy`，默认 `time`
- `sortDir`，`asc/desc`

#### 输出重点
返回 `PromotionInfoDTO`：
- `pageData`
- `totalData`
- `status`
- `newUserStatList`

`pageData` 是 Spring `PageImpl`

`pageData.content[]` 重点字段：
- `time`
- `promotionCode`
- `downloadNNumber`
- `downloadNumber`
- `clickDownloadNNumber`
- `clickDownloadNumber`
- `onlyClickNNumber`
- `onlyClickNumber`
- `addNNumber`
- `addNumber`
- `rechargeNNumber`
- `rechargeNumber`
- `rechargeNMoney`
- `rechargeMoney`
- `rechargeNGold`
- `rechargeGold`
- `showNNum`
- `showNum`
- `installNNum`
- `installNum`
- `clickNum`
- `clickNumFinalValue`
- `sysSettlementUSDTMoney`
- `sysSettlementCNYMoney`
- `costUnitPrice`
- `costUnitPriceFinalValue`

`totalData`：在 `PromotionDTO` 基础上额外看：
- `costUnitPrice`
- `costUnitPriceFinalValue`

`newUserStatList[]`：
- `date`
- `todayNewAndroidUsers`
- `todayNewIOSUsers`
- `yesterdayNewAndroidUsers`
- `yesterdayNewIOSUsers`

#### 典型用途
- 查看某结算单在某天的小时级推广表现
- 正式查询建议优先传 `time`

---

### 5.4 `query_settlement_promotion_realtime`
用途：查询结算单实时推广明细。

#### 输入参数
- `id`，必填
- `newStopTime`
- `pageNum`
- `pageSize`
- `sortBy`，默认 `time`
- `sortDir`，`asc/desc`

#### 输出重点
输出结构与 `query_settlement_promotion_day` 一致，同样返回：
- `pageData`
- `totalData`
- `status`
- `newUserStatList`

差异：数据来源是实时拉单逻辑，`newStopTime` 会影响统计上限。

---

### 5.5 `query_settlement_score`
用途：查询结算评分详情。

#### 输入参数
- `pageNum`
- `pageSize`
- `settlementNo`
- `createdDateStart`
- `createdDateEnd`

#### 输出重点
顶层：`list/total/pageNum/pageSize`

`list[]`：
- `id`
- `settlementNo`
- `isPaid`
- `secondDayRetention`
- `viewRate`
- `recycleRatio`
- `adClickRate`
- `costUnitPrice`
- `sevenDayRetention`
- `score`
- `renew`
- `solutions`
- `createdDate`
- `updateTime`

`RateDTO`：
- `value`
- `score`

#### 典型用途
- 按结算单看评分明细
- 不是按产品一步直出，通常要先通过产品定位结算单

---

### 5.6 `query_settlement_expense`
用途：查结算支出。

#### 输入参数
- `pageNum`
- `pageSize`
- `key`
- `settlementStartDate`
- `settlementEndDate`
- `approveStatus`，`PENDING_PAY/PAID/CLOSE`
- `settlementType`

#### 输出重点
顶层：`list/total/pageNum/pageSize`

`list[]` 重点字段：
- `id`
- `settlementSysUserId`
- `settlementNo`
- `productId`
- `productName`
- `supplierId`
- `channelCode`
- `channelName`
- `contactGroupUrl`
- `payNo`
- `periodStart`
- `periodEnd`
- `cooperationMode`
- `payTime`
- `settlementType`
- `chainContractType`
- `chainPaymentAddress`
- `chainHash`
- `approveStatus`
- `approveRemarks`
- `settlementRemarks`
- `payRemarks`
- `remarksUrls`
- `payMethod`
- `systemSettlementAmount`
- `systemSettlementAmountUsdt`
- `systemSettlementAmountCny`
- `actualSettlementAmountUsdt`
- `actualSettlementAmountCny`
- `financePayout`
- `exchangeRate`
- `price`
- `priceCurrency`
- `createdAt`
- `updatedAt`

---

### 5.7 `query_finance_payment_list`
用途：查财务支出列表。

#### 输入参数
- `pageNum`
- `pageSize`
- `key`
- `settlementStartDate`
- `settlementEndDate`
- `approveStatus`，`PENDING_PAY/PAID/CLOSE`
- `settlementType`
- `settlementSysUserId`

#### 输出重点
顶层：`list/total/pageNum/pageSize`

`list[]` 类型为 `ChannelSettlementPaymentViewDto`，重点字段：
- `settlementSysUserId`
- `channelSettlementReportId`
- `supplierId`
- 以及 `query_settlement_expense` 中 `ChannelSettlementPayment` 的全部主要字段

补充行为：
- `chainPaymentAddress` 返回时会尝试解密
- `remarksUrls` 返回时会补全可访问完整 URL

#### 典型用途
- 财务支付视角查询
- 对比结算支出与财务支出

---

## 6. 分配单 Tool

### 6.1 `query_distribute_list`
用途：分配单查询。

#### 输入参数
- `pageNum`
- `pageSize`
- `keyword`
- `status`，`0待分配/1已分配/2已关闭`
- `supplierId`
- `productId`
- `channelChildId`

#### 输出重点
顶层：`list/total/pageNum/pageSize`

`list[]` 重点字段：
- `id`
- `status`
- `remark`
- `configStartTime`
- `configEndTime`
- `createdAt`
- `updatedAt`
- `supplierId`
- `supplierName`
- `productId`
- `productName`
- `productType`
- `productModel`
- `productLogo`
- `channelChildId`
- `channelChildName`
- `channelCode`
- `deliveryDomain`
- `promotionLink`
- `adSlotId`
- `adSlotType`
- `adSlotName`
- `adWidth`
- `adHeight`
- `adFormat`
- `adSlotRemainingQuantity`
- `chargeModel`
- `cooperationMode`
- `cooperationPrices`
- `deductionConfig`
- `decisionMaker`
- `decisionMakerId`
- `maker`
- `supplierMember`
- `deptId`
- `opsManagers`
- `prdManagers`
- `budgetBalance`
- `trafficBalance`
- `totalCost`
- `totalCostTrue`
- `totalRevenue`
- `roi`
- `totalNewUsers`
- `channelProductPackageId`
- `channelProductPackageName`
- `channelProductPackagePackageName`

#### 典型用途
- 供应商 -> 子渠道 -> 分配单链路
- 查看广告位、合作模式、预算、负责人、马甲包补充信息

---

## 7. 已停用 Tool

### 7.1 `query_toll_and_free_score`
- 当前代码已注释停用
- Skill 不再默认或推荐使用
- 评分查询统一走：产品/筛选条件 -> `query_settlement_list` -> `query_settlement_score`

### 7.2 `list_products_and_packages`
- 视为兼容保留的旧合并接口
- Skill 不再默认或推荐使用
- 产品解析统一改走 `query_channel_product_list`
- 马甲包解析统一改走 `query_channel_product_package_list`
- 若用户同时要两类候选，也分别调用两个新接口，不依赖旧合并返回壳

---

## 8. 推荐路由

### 8.1 名称转 ID
- 部门名 -> `list_approval_user_dept`
- 部门下组员名 -> `list_decision_member`
- 决策负责人名 -> `list_decision_owners`
- 供应商名 -> `query_supplier_list`
- 子渠道名 -> `list_sub_channels`
- 产品名 -> `query_channel_product_list`
- 马甲包名 / 包名 -> `query_channel_product_package_list`

### 8.2 汇总查询
- 个人汇总 -> `query_personal_summary`
- 产品汇总 -> `query_product_summary`

### 8.3 结算链路
- 先找结算单 -> `query_settlement_list`
- 查详情 -> `query_settlement_detail`
- 查推广日明细 -> `query_settlement_promotion_day`
- 查推广实时明细 -> `query_settlement_promotion_realtime`
- 查评分 -> `query_settlement_score`
- 查结算支出 -> `query_settlement_expense`
- 查财务支出 -> `query_finance_payment_list`

### 8.4 常见多段链路
1. 部门 -> 组员 -> 个人汇总
2. 供应商 -> 子渠道 -> 分配单
3. 产品 -> 结算单列表 -> 结算评分
4. settlementNo -> 结算列表定位 -> 详情 / 推广
5. 供应商 -> 账号 -> 配置 / 登录信息
