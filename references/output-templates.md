# output-templates

## 1. 标准输出骨架

```md
查询类型：{query_type}
时间范围：{time_range}
筛选条件：{filters}
返回条数：{total}
当前页：{page_num}
每页：{page_size}

核心结果：
{result_blocks}
```

---

## 2. 个人汇总 / 产品汇总

```md
已完成{summary_type}查询。

时间范围：{start_date} ~ {end_date}
筛选条件：{filters}
返回条数：{total}

汇总指标：
- 消耗：{cost_sum}
- 新增：{new_users_sum}
- 充值人数：{recharge_user_count_sum}
- VIP充值：{vip_recharge_amount_sum}
- 金币充值：{coin_recharge_amount_sum}
- 打款金额：{pay_amount_cny_sum}
- 盈亏：{profit_cny_sum}
- 成本汇总：{cost_unit_cny_sum}
- 回收比汇总：{recycle_ratio_sum}

示例明细：
- 渠道码：{channel_code}
- 供应商：{supplier_name}（{supplier_id}）
- 子渠道：{sub_channel_name}（{sub_channel_id}）
- 产品：{product_name}（{product_id}）
- 负责人：{owner_name}（{owner_id}）

备注：若返回里附带 `queryStartDate` / `queryEndDate` 等展示字段，不代表本次筛选失效。
```

---

## 3. 供应商候选 / 供应商结果

```md
已完成供应商查询。

关键字：{keyword}
返回条数：{total}

候选结果：
{supplier_rows}
```

单条候选建议格式：

```md
- {supplier_name}（supplierId={supplier_id}，type={type}，status={status}）
```

---

## 4. 供应商账号

```md
已完成供应商账号查询。

筛选条件：supplierId={supplier_id}，keyword={keyword}，status={status}
返回条数：{total}

账号信息：
- 账号ID：{account_id}
- 账号：{account}
- 供应商：{supplier_name}（{supplier_id}）
- 子渠道：{channel_child_name}（{channel_child_id}）
- 模板：{template_name}（{template_id}）
- 域名：{domain}
- 登录地址：{login_url}
- 状态：{account_status}
- 渠道码：{channel_codes}
- 最近登录：{login_date}
```

---

## 5. 分配单

```md
已完成分配单查询。

筛选条件：supplierId={supplier_id}，channelChildId={channel_child_id}，productId={product_id}，status={status}
返回条数：{total}

示例分配单：
- 分配单ID：{config_id}
- 状态：{config_status}
- 产品：{product_name}（{product_id}）
- 子渠道：{channel_child_name}（{channel_child_id}）
- 渠道码：{channel_code}
- 推广链接：{promotion_link}
- 合作模式：{cooperation_mode}
- 广告位：{ad_slot_name}（{ad_slot_type}）
- 决策负责人：{decision_maker}（{decision_maker_id}）
```

---

## 6. 结算列表

```md
已完成结算列表查询。

筛选条件：{filters}
业务状态：code={code}，status={status_flag}，msg={msg}
返回条数：{total}
当前页：{page_num}
每页：{page_size}

示例结算单：
- 结算单：{settlement_no}
- 结算报告ID：{settlement_id}
- 类型：{settlement_type}
- 状态：{settlement_status}
- 供应商：{supplier_id}
- 产品：{product_id}
- 子渠道：{channel_child_id}
- 系统结算金额：{sys_settlement_amount}
- 实际结算金额：{actual_settlement_amount}
- 提审时间：{approval_submit_time}
- 打款时间：{pay_time}
```

---

## 7. 结算详情

```md
已完成结算详情查询。

目标结算单：{settlement_no}
结算报告ID：{settlement_id}
状态：{settlement_status}

基础信息：
- 类型：{settlement_type}
- 供应商：{supplier_id}
- 产品：{product_id}
- 子渠道：{channel_child_id}
- 推广码：{promotion_code}
- 推广区间：{promotion_start} ~ {promotion_stop}

财务信息：
- 实际结算USDT：{settlement_usdt}
- 实际结算CNY：{settlement_cny}
- 系统结算USDT：{sys_settlement_usdt}
- 系统结算CNY：{sys_settlement_cny}
- 汇率：{exchange_rate}
- 收款类型：{pay_type}
- 收款地址：{payment_account}

分析信息：
- 获客成本：{cost_unit_price}
- 回收比：{recycle_ratio}
- 盈亏：{profit_price}
- 详情附带评分：{detail_score}
```

---

## 8. 推广明细（日 / 实时）

```md
已完成结算推广明细查询。

查询类型：{promotion_type}
结算单：{settlement_no}
结算报告ID：{settlement_id}
查询时间：{time_or_range}
状态：{status}

汇总：
- 新增：{add_number}
- 充值人数：{recharge_number}
- 充值金额：{recharge_money}
- 展示：{show_num}
- 安装：{install_num}
- 点击：{click_num}
- 系统结算金额：{sys_settlement_amount}
- 获客成本：{cost_unit_price}

示例明细：
- 时间：{detail_time}
- 推广码：{promotion_code}
- 新增：{detail_add_number}
- 充值金额：{detail_recharge_money}
- 系统结算金额：{detail_sys_amount}
```

---

## 9. 结算评分

```md
已完成结算评分查询。

结算单：{settlement_no}
创建日期范围：{created_date_range}
返回条数：{total}

示例评分：
- 创建日期：{created_date}
- 总分：{score}
- 是否收费：{is_paid}
- 是否续费：{renew}
- 处理方案：{solutions}
- 次留：{second_day_retention}
- 观影率：{view_rate}
- 回收比：{recycle_ratio}
- 广告点击率：{ad_click_rate}
- 获客成本：{cost_unit_price}
- 7日留存：{seven_day_retention}
```

---

## 10. 结算支出 / 财务支出

```md
已完成支出查询。

查询类型：{expense_type}
时间范围：{start_date} ~ {end_date}
筛选条件：approveStatus={approve_status}，settlementType={settlement_type}，key={key}
返回条数：{total}

示例记录：
- 支付记录ID：{payment_id}
- 结算单：{settlement_no}
- 产品：{product_name}（{product_id}）
- 供应商：{supplier_id}
- 渠道：{channel_code} / {channel_name}
- 支付状态：{approve_status_value}
- 系统结算金额：{system_amount}
- 实际结算金额：{actual_amount}
- 财务打款金额：{finance_payout}
- 支付时间：{pay_time}
```

---

## 11. 歧义澄清模板

### 11.1 供应商歧义
```md
当前匹配到多个供应商，无法自动判断。
请指定 supplierId 或补充更完整名称：
{candidates}
```

### 11.2 产品歧义
```md
当前匹配到多个产品，无法自动判断。
请指定 productId 或补充更完整名称：
{candidates}
```

### 11.3 子渠道歧义
```md
当前匹配到多个子渠道，无法自动判断。
请指定 channelChildId 或补充更完整名称：
{candidates}
```

### 11.4 负责人歧义
```md
当前匹配到多个负责人，无法自动判断。
请指定 ownerId 或补充更完整名称：
{candidates}
```

### 11.5 结算单歧义
```md
当前时间范围内命中多张结算单，无法自动判断要查看哪一张。
请指定 settlementNo 或补充更精确条件：
{candidates}
```

---

## 12. 通用失败模板

```md
查询失败：{message}
如需排查，请补充本次使用的筛选条件、时间范围，以及返回中的业务状态码或 request_id。
```
