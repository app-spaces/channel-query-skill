---
name: channel-query-skill
description: 渠道系统 MCP 查询 Skill。处理汇总、供应商、账号、分配单、子渠道、产品、结算、推广、评分、支出等查询；使用时先做时间归一化与名称到 ID 解析，再调用正式 Tool。适用于“查渠道数据、查汇总、查结算、查供应商、查分配单、查评分、查支出”等场景。
---

# Channel Query Skill

所有 Tool 调用都必须把参数包在最外层 `request` 对象里。

## 如何选择 references

先看这一段，再按需读取对应文件，不要一次把所有 references 都读进来。

- 需要先快速上手、了解使用边界、标准流程与常见异常，读 `references/usage-guide.md`
- 需要确认某个 Tool 的入参、返回壳、关键字段、路由关系，读 `references/tool-mapping.md`
- 需要确认时间格式、ID 口径、公式、状态枚举、特殊结构，读 `references/field-notes.md`
- 需要组织对外回复、歧义澄清、失败提示、后端异常提示，读 `references/output-templates.md`
- 需要看真实可执行链路和正确调用顺序，读 `references/examples.md`
- 需要做验收、回归、改动后复查，读 `references/test-cases.md`

默认优先级：`usage-guide.md` > `tool-mapping.md` > `field-notes.md` > `examples.md` / `test-cases.md` > `output-templates.md`。

## 前置条件

当前 skill 依赖渠道系统 MCP tools。

- 若当前会话中未接入这组 MCP tools，则不要继续执行正式查询
- 应直接提示：`当前未接入渠道系统MCP服务，请先配置渠道MCP服务。`
- 是否可用，以当前工具集中能否看到对应渠道查询 tools 为准，不以某个固定 server 名为唯一判断依据

## 版本标识

- 当前版本：`v2026.04.09`
- 版本用途：用于标记当前 skill 的规则口径、引用结构与回归基线
- 何时升级版本：当 Tool 路由、关键字段口径、前置约束、异常处理规则、references 结构发生实质变化时
- 建议规则：先改内容，再手动递增版本；当前阶段用日期版最省事，也最容易和回归文档对齐

## 适用 Tool

### 枚举与解析
- `query_supplier_list`
- `list_sub_channels`
- `query_channel_product_list`
- `query_channel_product_package_list`
- `list_decision_owners`

### 正式查询
- `query_personal_summary`
- `query_product_summary`
- `query_supplier_account`
- `query_distribute_list`
- `query_settlement_list`
- `query_settlement_detail`
- `query_settlement_promotion_day`
- `query_settlement_promotion_realtime`
- `query_settlement_score`
- `query_settlement_expense`
- `query_finance_payment_list`

## 必须遵守的规则

1. 先把自然语言时间转换成明确时间范围，再做实体解析，再调用正式查询 Tool。
2. 只要下游 Tool 需要 ID，就不能直接把中文名塞进正式查询参数。
3. 用户已经给出精确 ID 时，可以跳过枚举解析。
4. 同名、多候选、歧义命中时，必须先澄清，不能默认取第一条。
5. `query_toll_and_free_score` 已停用，不再路由到它。
6. 输出时要写清实际时间范围、实际筛选条件、分页信息，以及关键 ID。
7. 方法说明以 `references/tool-mapping.md` 为准；真实返回若带额外字段，可保留原值，但不要擅自改写业务含义。

## ID 解析规则

### 1) 供应商
- `supplierId` 通过 `query_supplier_list` 解析
- 常见格式为 `GYS...`
- 支持 `keyword`、`id`、`name`、`status`、`type`

### 2) 子渠道
- `channelChildId` 必须先拿到 `supplierId`，再调用 `list_sub_channels`
- 不能把“默认子渠道”之类的中文名称直接塞给 `channelChildId`

### 3) 产品与马甲包
- 产品枚举统一通过 `query_channel_product_list` 解析，使用 `key` 模糊匹配 `productId` 或产品名称，且默认只看未删除产品
- 马甲包枚举统一通过 `query_channel_product_package_list` 解析，使用 `key` 模糊匹配马甲包名称或包名；如已知 `productId`，可额外传入做精确过滤，语义为 `productId AND key`
- 产品候选优先只回写必要字段：`productId`、`name`、`status`、`type`、`model`
- 马甲包候选优先只回写必要字段：`id`、`productId`、`name`、`packageName`、`productName`
- 如果用户给的是马甲包名称，但下游正式查询要的是 `productId`，先解析马甲包，再从候选里取 `productId` 继续下钻
- 若用户同时要看产品候选和马甲包候选，分别调用两个枚举 Tool，不走旧的合并接口
- `list_products_and_packages` 仅视为兼容保留接口，Skill 默认不再使用

### 4) 决策负责人
- `ownerId` 通过 `list_decision_owners` 解析
- 返回是 `DropDown` 树，重点取 `value`

### 5) 结算记录
- `query_settlement_detail`、`query_settlement_promotion_day`、`query_settlement_promotion_realtime` 都优先使用结算报告 `id`
- 如果用户只给了 `settlementNo`，先用 `query_settlement_list` 定位记录，再取 `id`
- `query_settlement_score` 直接使用 `settlementNo`

## 时间格式规则

- `query_personal_summary` / `query_product_summary`：`startTime` / `endTime`，格式 `yyyy-MM-dd`
- `query_settlement_list`：各类区间字段都用 `LocalDateTime`，格式 `yyyy-MM-ddTHH:mm:ss`
- `query_settlement_expense` / `query_finance_payment_list`：`settlementStartDate` / `settlementEndDate`，格式 `yyyy-MM-dd`
- `query_settlement_score`：`createdDateStart` / `createdDateEnd`，格式 `yyyy-MM-dd`
- `query_settlement_promotion_day`：`time` 为单日，格式 `yyyy-MM-dd`
- `query_settlement_promotion_realtime`：可选 `newStopTime`，格式 `yyyy-MM-dd`

## 查询路由

### 汇总类
- 查个人维度汇总 -> `query_personal_summary`
- 查产品维度汇总 -> `query_product_summary`

### 供应商类
- 查供应商列表 / 解析 supplierId -> `query_supplier_list`
- 查供应商账号 -> `query_supplier_account`

### 分配与枚举类
- 查分配单 -> `query_distribute_list`
- 查子渠道 -> `list_sub_channels`
- 查产品 -> `query_channel_product_list`
- 查马甲包 -> `query_channel_product_package_list`
- 查决策负责人 -> `list_decision_owners`

### 结算与财务类
- 查结算列表 -> `query_settlement_list`
- 查结算详情 -> `query_settlement_detail`
- 查按天推广 -> `query_settlement_promotion_day`
- 查实时推广 -> `query_settlement_promotion_realtime`
- 查结算评分 -> `query_settlement_score`
- 查结算支出 -> `query_settlement_expense`
- 查财务支出列表 -> `query_finance_payment_list`

## 标准执行流程

1. 识别用户要查哪一类数据。
2. 把相对时间改写为明确日期或日期时间。
3. 判断正式查询是否依赖 `supplierId` / `channelChildId` / `productId` / `ownerId` / `settlementNo` / `id`。
4. 如缺少 ID，先调用对应枚举 Tool 做解析。
5. 若命中多个候选，先澄清。
6. 组装正式查询请求，统一包成 `{ "request": { ... } }`。
7. 读取返回壳，提炼核心字段。
8. 输出时回写：时间范围、筛选条件、总数、分页、关键结果、关键 ID、`request_id`（若接口提供）。

## 输出规范

### 简版
适合“先给我结果”。
- 1 句结论
- 关键时间范围
- 关键筛选条件
- 2 到 5 个核心指标

### 标准版
默认使用。
- 查询类型
- 时间范围
- 筛选条件
- 总数 / 分页
- 汇总结果或主结果
- 必要的 ID 与状态

### 明细版
适合用户要求“展开看看”。
- 保留接口原始关键枚举值
- 保留关键 ID、编号、状态、金额、时间
- 若真实返回包含文档未单列的附带字段，可保留原值，但不要自行解释成新的业务口径

## 已知业务口径

1. `costUnitCnySum` / `recycleRatioSum` 不是简单求和，而是按去重渠道平均。
2. `analysisData.costUnitPrice = 系统结算金额(CNY) / 新增(原始值)`。
3. `analysisData.recycleRatio = VIP充值(原始值)(CNY) / 系统结算金额(CNY)`。
4. `analysisData.profitPrice = VIP充值(CNY) - 系统结算金额(CNY)`。
5. `query_personal_summary` / `query_product_summary` 返回中的 `queryStartDate` / `queryEndDate` 是展示字段；若与用户请求范围不完全一致，不要直接判定为筛选失效。

## 参考资料

按需读取，选择规则见上面的“如何选择 references”：
- `references/usage-guide.md`：使用说明、前置条件、快速上手、常见查询路径与异常处理
- `references/tool-mapping.md`：15 个 Tool 的输入、输出、路由与关键字段
- `references/field-notes.md`：ID 口径、时间格式、状态与枚举、公式与特殊结构
- `references/output-templates.md`：轻量回复模板（仅管输出结构与异常提示，不承载字段权威口径）
- `references/examples.md`：真实链路示例
- `references/test-cases.md`：验收与回归清单

## 最终目标

把自然语言查询稳定转换成：
- 正确的时间范围
- 正确的实体 ID
- 正确的 Tool 选择
- 正确的返回结构解读
- 可复核、可继续下钻的结果输出
