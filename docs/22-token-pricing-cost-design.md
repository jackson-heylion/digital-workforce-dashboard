# 22｜Token 用量计价与 Agent 成本计算（B03 增量需求 / 待实施评审）

> **状态（2026-10-10）**：项目发起人明确提出“要支持 token 计算方式计算价格成本”。**支持按 Token 用量与模型价格计算成本的能力已经明确**；这不等于 B03 当日成本的所有显示/汇总规则、v1.1/v2 传输字段、模型价格来源或真实部署已经批准。与 [21 指标台账](21-metrics-field-confirmation-register.md) 的 B03 保持一致。
> **不修改** [Run v1 OpenAPI](../openapi/agent-run-reporting-v1.json)、[v1.1 草案](../openapi/agent-run-reporting-v1.1-draft.json)、[原始 HTML](../prototype/index.html)。以下为工程建议和接口候选，不是上线功能。

## 一、目标与真实性

- **MVP 目标**：能依据模型/供应商真实返回的 Token usage 与**带有效期和版本**的费率卡计算 Agent 的**模型调用估算成本**，再汇总到 Run、Agent、集团。若收到供应商可验证的逐笔计费记录，另存真实已出账费用。
- **成本标签分离**：Token×配置价格 = `ESTIMATED`，不冒充 `BILLED`；未知费用为 `UNAVAILABLE`（null），不能是 0。价格无法匹配、模型未知、Usage 缺失时不生成可信金额。
- **范围限制**：本计算主要覆盖**模型推理 API 费用**；工具/API、联网搜索、向量存储、基础设施、人工复核等可能有**非 Token 费用**，除非另有可靠数据，不并入“模型 Token 估算”。是否扩展为“总成本”需另议。
- **不能硬编码厂商现价**：定价可能按模型版本、区域、请求上下文长度、batch/实时、缓存写入/命中及商务协议变化。管理员维护且版本化；禁止事后用新价重写旧记录。

## 二、仓库现状与准确性缺口

| 事实来源 | 已有能力 | 当前缺口 |
| --- | --- | --- |
| Run v1.0 | 可选 `input_tokens`、`output_tokens`，每次外层 Agent 执行的 `agent_code/run_id` 与幂等键 | **无模型 ID/供应商/缓存命中量/模型调用次数或计价版本**。仅适用于能证明一个 Run 由同一计费模型调用构成的有限场景；不得仅因登记有一个 `Agent version` 就套模型价格 |
| v1.1 **未批准草案** | 可选 `cost.amount/currency/quality/source/basis_id`，区分 BILLED/ESTIMATED | 只有结果金额而非全部每模型调用 usage；**不能完整计算多模型 Agent 的准确用量费用** |
| 原型 HTML | 固定 `¥308` / `单均 ¥0.46` 与 `perfData.today × perfData.cost` | 全是演示数据；**不用于生产计价** |

## 三、计价对象与采集分层

**建议最小粒度 = 一次实际的模型 API 请求（model_call）**，而 Agent Run 可以调用零次/一次/多次不同模型。层级为 `agent -> run -> model_call -> measured_usage -> price_version -> cost_entry`。

1. **简化接入档（单模型）**：调用方能证明每个 Run 恰有一个已知模型计费主体，且该 Run `input_tokens/output_tokens` 是供应商真实使用量，允许从 Run 汇总映射成一个 `model_call` 计价明细（带明确的模型标识与价格配置）。**不是所有 Agent 都适用**。模型无法确定、计量可能混合时成本标记 `UNAVAILABLE`。
2. **完整接入档（多模型/缓存/工具调用）**：在 Agent 调用侧 wrapper/Gateway 采集每个实际模型 API 请求的模型 ID、供应商、计费区域、发生时间、真实 usage（可空的细分项）、稳定 `call_id`。可用**独立轻量 model-call 记录接口或未来版本化可选扩展**，不向现有严格 v1 请求暗加字段；两者如何选待批准。
3. **Run 状态与费用独立**：`FAILED`/`CANCELLED` 的 Run 也可能已经花费模型 Token；不因 Run 失败就清零成本。不含任何模型调用的 Run 成本可能为零，**但只有观测完整时才认定真零**。

**候选模型调用用量记录（展示概念，不是正式接口 Schema）**：

```json
{
  "agent_code": "finance-review-agent",
  "run_id": "run-20261010-001",
  "call_id": "model-call-01",
  "provider": "model-studio",
  "model_id": "provider-model-version",
  "billing_region": "cn-beijing",
  "occurred_at": "2026-10-10T14:35:00+08:00",
  "mode": "REALTIME",
  "usage": {
    "input_tokens_total": 12000,
    "input_tokens_cache_read": 2000,
    "input_tokens_cache_write": 0,
    "output_tokens_total": 1700
  }
}
```

**注意**：上例 `cache_read` 包含在 `input_tokens_total` 内，**不是额外相加**。实际供应商缓存 Token / reasoning Token 的包含关系不一，先经供应商 Adapter 转换为**互不重叠的计费分桶**：`input_normal`、`input_cache_read`、`input_cache_write`、`output_billed`，若源响应无法可靠拆分，则只在相应模型价格规则允许时使用无缓存基础公式，否则**标为无法准确估算**。输出 reasoning tokens 如已包含在 output total，禁止再次求和。

## 四、价格卡设计

建议 `dw_model_price_rate`（字段/DDL 均为候选）：

| 字段 | 用途 |
| --- | --- |
| `price_version_id` / `source_url` / `confirmed_by` | 追溯价格卡来源/审核人 |
| `provider`, `model_id`, `billing_region` | 选择对应平台、**真实计费模型**与区域 |
| `billing_mode`, `tier_selector` | REALTIME/BATCH、上下文输入区间/特殊计价档位（若适用） |
| `currency`, `per_token_unit` | 计价币种及单位，建议显式记录每 1,000,000 tokens；**不得默认所有供应商币种相同** |
| `input_regular_rate`, `input_cache_read_rate`, `input_cache_write_rate`, `output_rate` | 四种费率分桶；供应商无对应分桶时记录适用性而不是随意置零 |
| `effective_from`, `effective_to`, `revision` | 历史价格有效期和只读版本，不以新配置覆盖旧价 |

对跨多个 price tier 的模型，Adapter 先依据供应商规则选择**单次请求适用的档位**；有的供应商按请求输入规模决定全单价，有的按月量或特殊模式计价，不得统一假定逐段累进。

## 五、公式与质量

单次模型调用费用：

```text
normal_input = input_total - cache_read - cache_write
amount = (normal_input * regular_input_rate
       + cache_read * cache_read_rate
       + cache_write * cache_write_rate
       + output_billed * output_rate) / price_unit
```

以上公式**仅用于价格卡确实采用这四种分桶、且用量定义匹配的模型**；具体平台可能有其他模态/最低消费/缓存存储/工具调用附加费用，要另补计价组件。单次请求/Run 同一币种成本用 **BigDecimal / DECIMAL** 累加，禁止 double/float。中间至少保留足够小数精度，**不要逐条四舍五入到分后才汇总**。

**纯示意演算（以下不是任何厂商实际价格）**：普通输入 10,000 Token × ¥2/百万 + 输出 2,000 Token × ¥8/百万 = **¥0.036**，质量标记 `ESTIMATED`，同时存 `price_version_id`；既不是 ¥0.04 已出账，也不能据此推断总业务成本。

储存建议 `dw_model_call_usage`：`agent_id, run_id, call_id` 联合唯一、供应商/模型/usage 原始核验摘要、规范分桶、usage_source、计价日期；`dw_model_cost_entry`：call_id、price_version_id、CNY/USD 等 currency、计算前与后金额、质量/来源、计算版本、计算时间、覆盖状态。**真实账单对齐必须另记录 BILLED 项**；不能同时把同一调用的估算和账单作为总成本双计。

**无费用 / 全部覆盖 / 部分覆盖**单独标记。缺失模型价格、Token/usage_source 可信度不足或接口报表只有部分模型请求时，视为**部分估算或不可计算**，严禁默认为 0。失败请求也可能消耗 Token，只有供应商实际用量返回才可准确计算；不要用字符长度/本地 Tokenizer 无证明地冒充计费 Token。

## 六、聚合与后补

- **同一 `call_id` 幂等**：`(agent_id,run_id,call_id)` 唯一；外层 Run 状态事件和模型调用级费用不能在两套汇总中分别再加一次。对最终未结束或迟到的调用费用，基于可重算的事实表/水位增量纠正，不使用永不回滚的前端累计计数。
- **日期归属仍待用户选**：可按所属外层 Run `started_at`（和 B01 对齐）、模型实际调用时间或供应商账单入账时间，三者不可偷换；也要确认时区、TEST/PROD、跨日与历史补价。
- **价格修改处理**：历史费用保留原 `price_version_id` 与计算快照，明确修正规则后才生成修订/调整条目，不因后台更新价格卡让已发布报表静默重算。
- **账单覆盖**：有 PROVIDER_BILL 时独立存财务事实，展示 `BILLED` 与 `ESTIMATED` 的关系及覆盖率；即使供应商账单比估算不同，**不能将估算擅自升级为 BILLED**。
- **UI**：B03「模型 Token 估算费用」优先于含糊的「总成本」，显示币种、费用质量、计价依据和覆盖状态；是否和已出账分栏、无数据文案、KPI 位置、单均成本 B04、月趋势 B08/C08 都待确认。

## 七、建议分期（待批准）

| 阶段 | 目标 | 前置依赖 |
| --- | --- | --- |
| P0 | 单模型 Run 的真实 input/output Token + 手工版本化模型费率，生成 **ESTIMATED** 模型费用；缺模型/费率/Token 显示不可用 | 验证至少一个真实 Agent 的返回 usage、模型标识和缓存计费语义；补充版本化可选模型 ID 的接入设计 |
| P1 | 多模型 Run 每调用使用量、缓存/思考输出归一、并行工具模型请求、按模型调用幂等及重算 | 批准新传输 Schema 或单独调用明细接口；源平台可提供细粒度数据 |
| P2 | 真实账单对照、优惠/阶梯/退款、汇率与成本覆盖率、已出账和估算对齐 | 真实供应商结算数据 + 经批准的费用分摊与账单校正规则 |

## 八、确认边界与验收测试清单（候选）

- [ ] 相同 `run_id/call_id` 重传 10 次只计一次费用；新模型调用的不同 ID 可累加。
- [ ] 一个 Run 同时用两个模型（不同单价），能正确分模型计价；只靠外层 Run Token 总量不假装准确。
- [ ] 缓存命中输入已含在 input_total 时正确扣除，reasoning 已含在 output_total 时不重复加。
- [ ] 价目表不同生效版本/不同区域/阶梯与币种能选择正确版本；历史费率不被覆盖。
- [ ] FAILED/CANCELLED Run 的真实用量仍可计价，费用不可得与真实 0 能区分。
- [ ] 同一笔费用估算与已出账不能双重计入合计；账单晚到不会未经审批悄悄更改已确认历史事实。
- [ ] 生产无 Usage 或价格卡时禁用静态「¥308」及 `perfData.cost` 演示数据。

## 九、尚待逐项确认（B03 与 F04/F08/S07）

**已经明确**：必须支持**Token 用量 × 模型价格**生成可追溯估算成本。

**仍待确认**：B03 的首页成本展示方案（只有估算/账单分列/兼容两者）、币种与汇率、费用统计日、模型单价维护/审批机制、付费和免费额度/优惠、使用量/缓存明细的来源、接入字段的协议版本、P0 是否要求所有 Agent 具备模型标识、部分覆盖与零数据展示、能否纳入工具/基础设施、单均成本 B04 与趋势 B08/C08 的正式公式。旧 v1 兼容要求不变。

## 参考链接（技术依据，价格随供应商变动）

- [Alibaba Cloud Model Studio 模型调用价格](https://www.alibabacloud.com/help/en/model-studio/model-pricing)：部分模型有阶梯、缓存命中/创建分价、区域差异和优惠。
- [OpenAI API 官方计价](https://developers.openai.com/api/docs/pricing)：按模型区分普通输入、缓存输入和输出 Token 费率，部分工具独立收费。
- [百炼上下文缓存计价](https://docs.modelstudio.console.alibabacloud.com/en/model-studio/context-cache)：缓存命中及写入可能与普通输入单价不同。
