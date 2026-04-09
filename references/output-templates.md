# output-templates

用途：这里只保留轻量回复模板，不再承载字段权威口径。

- 字段定义、入参/出参、返回壳差异，以 `tool-mapping.md` 和 `field-notes.md` 为准。
- 真实异常与过滤问题，以最新回归结果和 examples/test-cases 为准。
- 涉及账号、密码、登录地址、登录 IP、收款地址等敏感字段，默认不输出；确需展示时先脱敏。

---

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

`result_blocks` 建议只保留 2 到 5 个最关键结果，不要机械平铺所有字段。

常见写法：

### 1.1 汇总类
```md
- 消耗：{cost_sum}
- 新增：{new_users_sum}
- VIP充值：{vip_recharge_amount_sum}
- 打款金额：{pay_amount_cny_sum}
- 盈亏：{profit_cny_sum}
```

### 1.2 列表类
```md
- 示例记录：{primary_id}
- 状态：{status}
- 供应商：{supplier_id}
- 产品：{product_id}
- 金额：{amount}
```

### 1.3 明细类
```md
- 目标记录：{primary_id}
- 状态：{status}
- 关键金额：{amount}
- 关键时间：{time_value}
- 关键附加信息：{extra_note}
```

---

## 2. 歧义澄清模板

### 2.1 供应商歧义
```md
当前匹配到多个供应商，无法自动判断。
请指定 supplierId 或补充更完整名称：
{candidates}
```

### 2.2 产品歧义
```md
当前匹配到多个产品，无法自动判断。
请指定 productId 或补充更完整名称：
{candidates}
```

### 2.3 子渠道歧义
```md
当前匹配到多个子渠道，无法自动判断。
请指定 channelChildId 或补充更完整名称：
{candidates}
```

### 2.4 负责人歧义
```md
当前匹配到多个负责人，无法自动判断。
请指定 ownerId 或补充更完整名称：
{candidates}
```

### 2.5 结算单歧义
```md
当前时间范围内命中多张结算单，无法自动判断要查看哪一张。
请指定 settlementNo 或补充更精确条件：
{candidates}
```

---

## 3. 通用失败模板

### 3.1 未接入 MCP 服务
```md
当前未接入渠道系统MCP服务，请先配置渠道MCP服务。
```

### 3.2 通用查询失败
```md
查询失败：{message}
如需排查，请补充本次使用的筛选条件、时间范围，以及返回中的业务状态码或 request_id。
```

---

## 4. 后端异常提示模板

### 4.1 过滤疑似失效
```md
本次接口虽然返回成功，但结果中出现了与筛选条件明显不一致的记录。
当前筛选条件：{filters}
异常样例：{bad_rows}
结论：疑似后端过滤逻辑异常，当前结果仅可部分参考，建议转后端复核。
```

### 4.2 字段生效口径不一致
```md
本次实测发现文档字段与真实生效字段不完全一致。
请求参数：{request_fields}
实测现象：{observed_behavior}
当前建议：按 {effective_fields} 作为临时生效口径，并保留回归证据。
```

### 4.3 返回壳异常
```md
接口返回成功/失败口径与常规分页壳不一致。
请记录：tool={tool_name}，业务状态={status_block}，request_id={request_id}
如需继续排查，请附上原始请求参数与首条异常返回。
```