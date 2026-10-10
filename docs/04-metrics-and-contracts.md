# 04｜Agent Run 指标与日志契约（极简 MVP）

> **产品口径审批约束（2026-10-10）**：用户要求每个指标和字段的展示、计算、来源与标准都逐项讨论确认。请查阅 [21 · 指标字段待确认台账](21-metrics-field-confirmation-register.md)。本文件中的 KPI 公式只是此前研发建议，**不能视为逐项已获用户确认的产品口径**。

> **Run 上报标准 v1**：[18 人类可读规范](18-agent-run-reporting-standard.md) | [OpenAPI](../openapi/agent-run-reporting-v1.json)。该标准是最新接入契约。
>
> 最新范围以 [17](17-agent-log-dashboard-mvp.md) 为准：**只登记 Agent、采集 Run 日志、展示技术运营指标**。下列字段/公式是建议版本 v0.2；接入时必须按真实平台数据验证。**不再要求源业务最终回执、独立业务 Task、工时与 ROI 核验。**

## 1. 最小实体

- **Agent**：平台中登记的一个外部 Agent 实例；`platform + environment + external_agent_id`（多空间需补 namespace）对应唯一内部 `agent_id`。
- **Run**：这个 Agent 的**一次技术执行**；由源 `run_id` 唯一标识，可能已完成/失败/取消/运行中；没有 Run ID 的源先定义稳定替代键，否则不能可信去重。
- **Log（可选）**：Run 中的时间、级别、脱敏短消息与错误；不能把每条日志记成一次 Run。
- **部门/Owner**：Agent 登记字段，按授权查询；改部门后的历史归属口径先采用“按当前归属”，如果需要历史回溯再补快照，必须展示此限制。

## 2. KPI 字典

| 名称 | 数学定义 | 统计时间 | 缺值规则 |
| --- | --- | --- | --- |
| **Agent 总数（A01 已确认方案 A）** | **全部未归档的已登记 Agent 计数**（启用/暂停/停用均包含；部署副本和版本升级不增加人数）；具体去重 ID/跨环境粒度仍待确认 | Agent 登记当前状态；不按 Run 是否活跃筛选 | 无登记可自然得到 0；前端副标题真实在用数按 A02 另议 |
| **真实在用数（A02 已确认方案 A）** | 仅统计登记记录中由负责人/管理员**主动认定「已正式投入使用」**的 Agent；**近期有无 Run 不影响认定**。暂停/停用/TEST/归档处理等边界仍待确认 | Agent 登记使用状态；不能从 enabled 或 Run activity 自动推断 | 无明确认定不得自动纳入；正式状态枚举/权限/显示细则尚待确认 |
| **部门分组与人数（A03 已确认方案 A）** | **每个未归档 Agent 仅按一个主归属部门计数**，多个部门使用也不重复；各组为对应主部门人数。与 A01 统计同一资产集合，未分配部门的显示/归并待确认 | Agent 登记中主归属部门信息；是否同步企业组织目录待确认 | 无部门、跨组织调整、历史口径及部门授权未确认；不默认建设多部门使用映射 |
| **员工卡管理与执行状态（A04 已确认 C）** | **主标签按 Agent 登记管理状态展示**；「真实在用」为 A02 独立标记；**仅在可靠获得 RUNNING 及其终态时可辅显「执行中」**，另外展示最近执行时间。不以管理启用/上次运行推断在线/离线 | Agent 登记管理字段 + **可选** Run RUNNING/终态及事件时间 | 没有可靠 RUNNING 不能推断正在执行；不会因选择 C 把 v1 终态一次上报改为必报 RUNNING；失效阈值/状态枚举/颜色等仍待确认 |
| **名称/图标/职责简介（A05 已确认 A）** | 卡片显示名称与图标、详情展示职责简介（布局/空值未定）；取 Agent 登记中的人工维护资料，不从运行日志计算，也不把展示名作为 Agent 身份键 | 负责人或管理员在 Agent 登记时手工维护的 display_name/icon/description；不自动同步第三方 Agent 平台 | 显示名重名/必填/限长、头像选取/上传/默认、简介长度/敏感信息、历史修改与样式均未确认 |
| **当前 Agent 版本（A06 已确认 A）** | **登记中人工维护的单个当前版本字符串**，员工卡/详情直接展示，不按 Run 计算；不使用当前版本标注历史运行 | 负责人/管理员维护的 Agent 登记版本；不从平台自动同步，不强制逐 Run 版本快照 | 格式/必填/缺省、编辑与回滚历史、环境差异、旧 Run 版本归属仍待确认；Agent 版本不是协议 `schema_version` |
| Run 执行次数 | 时间窗内 `COUNT(DISTINCT agent_id, source_run_id)` | 推荐 started_at；缺时间时须声明回退口径 | 无源接入与已接入真实 0 分开 |
| 已结束 Run 数 | `SUCCEEDED + FAILED + CANCELLED` | finished_at | 无终态为 0 |
| 技术成功率 | `SUCCEEDED / (SUCCEEDED + FAILED)` | finished_at，同一时间窗 | 分母为 0 → null，不输出 100% |
| 技术失败次数 | `COUNT(status=FAILED)` | finished_at | 分母不是成功率计算依据 |
| 平均执行耗时 | `AVG(duration_ms)`，仅有效已完成/失败且有 duration 的 Run | finished_at | 无有效样本 → null |
| Token 消耗（可选） | Sum(input_tokens + output_tokens) | 归属 Run，明确计量来源 | 未提供不显示 |

**执行次数并非业务任务完成数，技术成功率并非业务成功率**。重复上报同一 Run，不增加执行次数；真实重试生成新的源 Run ID，按新的执行次数计入。

## 3. 标准 Run 摘要（建议，示例数据均为虚构）

```json
{
  "schema_version": "1.0",
  "agent_code": "agent-sample-01",
  "run_id": "run-sample-0001",
  "status": "SUCCEEDED",
  "started_at": "2026-10-10T08:02:00+08:00",
  "finished_at": "2026-10-10T08:02:03+08:00",
  "duration_ms": 3000,
  "input_tokens": null,
  "output_tokens": null,
  "error_code": null,
  "error_message": null
}
```

调用身份/令牌通过服务端到服务端 `Authorization: Bearer` 校验，且应绑定授权 `agent_code`/环境，不能信任请求体自报身份。**服务端字段统一使用 `source_run_id` 存储，请求入参统一使用 `run_id`**；两者的映射由接收层完成。终态需要 `finished_at`，`started_at` 全状态必填；详见 [18](18-agent-run-reporting-standard.md)。

## 4. 状态与时间

- `RUNNING → SUCCEEDED / FAILED / CANCELLED`。如果源只发终态，允许直接写入终态；旧的 RUNNING 重试数据不得把终态覆盖。
- `UNKNOWN` 可以表示来源状态无法映射，不可计为 SUCCEEDED。
- `started_at`/ `finished_at` 源时间有时缺失，服务器写入 `received_at`，不可擅自当成真实完成时间。各 KPI 统一展示所用时间基准。
- 统计区域默认 `Asia/Shanghai`，时间窗半开 `[start, end)`，API 显示 `as_of` 和数据覆盖/最新接收时间。
- 命名为“在线”要求额外的健康协议；仅最近有一条 Run 只能称“最近活跃”。

## 5. 建议接口响应（虚构数据）

```json
{
  "as_of": "2026-10-10T08:30:00+08:00",
  "timezone": "Asia/Shanghai",
  "metric_scope": "AGENT_RUN",
  "metrics": {
    "registered_agents": {"value": 9, "quality": "DEMO"},
    "run_count": {"value": 663, "quality": "DEMO"},
    "run_success_rate": {"value": 0.972, "quality": "DEMO"},
    "avg_duration_ms": {"value": 4600, "quality": "DEMO"}
  },
  "latest_log_received_at": "2026-10-10T08:29:59+08:00"
}
```

生产 `quality` 不允许为 `DEMO`。未接入时可以给 `value=null, quality=UNAVAILABLE`，不能把示例 9 人/663 次填进去。

## 6. 数据质量验证

- 对相同筛选窗/部门，按 Agent 分组的 Run 数相加等于集团 Run 总数；
- `(agent_id, source_run_id)` 重复上报不能产生两条；
- `RUNNING` 不混入成功率分母；成功与失败统计时间口径一致；
- 没有耗时时 avg_duration_ms 为 null；
- 缺 Run 日志、数据延迟、真实 0 分开；
- 正式看板没有“核验节省工时”“已结算成本”“真实业务成功率”等未经源系统证实的项目。

扩展的业务 Task/Run/成本账本逻辑归档在 [14](14-employee-management.md)、[15](15-data-integration.md)，**非 MVP 要求**。
