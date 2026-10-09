# 04｜指标字典、事件契约与最小数据模型

> **规范建议 v0.1**：所有指标均需产品负责人 + 业务 Owner + 数据/财务负责人共同确认后再用于集团汇报。原型静态数字不纳入本规范的真实统计。

## 1. 统一维度与计算边界

每次查询都明确：
- `tenant_id`（可先固定集团）、`employee_id`、`business_department_id`、`scenario_id`、`task_type`、`source_system`；
- `start_at/end_at`（半开区间）、时区（默认建议 Asia/Shanghai）、需要营业日时另存 `business_date`；
- `data_quality`：VERIFIED / ESTIMATED / DEMO；DEMO 只能来自隔离演示环境；
- 数据新鲜度 `last_event_received_at`、完整性标记 `collection_status`；
- 员工生命周期与部门归属按事件发生时快照，避免后续调部门导致历史报表漂移。

## 2. MVP 指标定义

| 指标 | 计算口径 | 特别说明 |
| --- | --- | --- |
| 已验证上线员工数 | `lifecycle=ACTIVE AND connection=CONNECTED AND verified_at IS NOT NULL` | “接入中”“规划中”和长期失联员工单列 |
| 新受理任务数 | 时间窗内唯一 `task_id` 的 `TaskAccepted` 数 | 唯一化，非 Run/Step/API 次数 |
| 完成任务数 | 时间窗内终态成功或终态失败的唯一 `task_id` 数 | 按**完成时间**；取消、审批中单列 |
| 业务成功率 | `SUCCEEDED / (SUCCEEDED + FAILED)`（独立终态任务） | 取消/未完成不入分母；各场景业务成功条件需签署 |
| 一次自动完成率 | `SUCCEEDED 且 attempt_count=1 且 human_intervention_count=0` / 全部终态任务 | 和业务成功率分开；“免确认”和“无人审”不必相等 |
| 人工介入率 | 至少发生一次人工介入的唯一终态任务 / 终态任务数 | 人工节点包含合规强制审核，属于流程特点不是全部坏事 |
| 重试率 | 至少有 2 个 Run 的终态任务 / 终态任务数 | 也显示每任务平均尝试数 |
| 端到端耗时 | `terminal_at - accepted_at` | 包含审批等待；另列执行耗时 P50/P95 |
| 自动执行耗时 | 各 Run/Step 的受控执行时长，去除重复/并行叠加 | 不能将并行工具调用耗时相加当墙钟时间 |
| Token/API 成本 | 按事件使用量 × 对应单价版本 + 外部调用实际费用 | 标记估算/结算及币种；补账可追溯 |
| 总成本 | Token/工具/API/基础设施分摊/人工复核/运维等可归属成本 | 避免在 run/step 两层重复求和；固定成本按公布规则分摊 |
| 理论释放工时 | Σ（基线人工时长 − 实际保留人工处理时长 − 额外复核时长），负值保留以揭示低效 | 仅统计有基线样本的**已交付成功业务任务**，异常/返工额外时间需计入 |
| 已核验净收益 | 已证实的经济收益 − 全部归属成本 | 释放产能不等于实际裁减/现金节约；不能直接把小时乘工资就称为已实现财务收益 |

**展示规范**：任何比率支持点击查看分子/分母/排除项/查询时间；成本注明含税、汇率与价格来源；工时注明基线版本、样本数、核准人和核准日期。

## 3. 单位与去重

- 一个`Task` 代表“一件业务委托”；拆单与批量任务支持 `parent_task_id` 和 `counting_unit`，按场景配置只计父或计子，不能父子同计。
- `Run` 是执行尝试，`Step` 是其中的步骤，`usage_id` 是模型/API 计费事件；一个任务包含多个 Run/Step/Usage。
- 不同事件来源的`source_event_id`允许相同，去重唯一键必须带`source_system`。
- 报表可按任务受理日或完成日看不同漏斗；**成功率默认按完成日**，仪表盘必须写清口径。

## 4. 规范事件（示意，不包含生产 ID）

```json
{
  "schema_version": "1.0",
  "source_system": "dish-service",
  "source_event_id": "evt-demo-0001",
  "event_type": "TASK_OUTCOME_CONFIRMED",
  "occurred_at": "2026-10-09T10:30:00+08:00",
  "tenant_id": "group-demo",
  "employee_id": "emp-dish-adjust",
  "scenario_id": "dish-adjustment",
  "task_id": "task-demo-10001",
  "source_task_id": "change-request-demo-01",
  "run_id": "run-demo-10001-1",
  "trace_id": "00000000000000000000000000000001",
  "status": "SUCCEEDED",
  "business_outcome": {
    "outcome_type": "DISH_CHANGE_EFFECTIVE",
    "result_ref": "ref-demo-123",
    "confirmed_by": "dish-service"
  },
  "metrics": {
    "llm_input_tokens": 0,
    "llm_output_tokens": 0,
    "cost_amount": "0.00",
    "cost_currency": "CNY",
    "cost_type": "ESTIMATED"
  },
  "data_quality": "VERIFIED"
}
```

- `schema_version` 语义化版本；未知事件类型进入隔离队列，不默默丢弃。
- `source_event_id` 应由事件生产方生成并在重试中保持不变；换新 ID 的重复业务回执也由 task 状态机保护。
- `occurred_at` 为原事件时间；`received_at` 由平台服务端写入。
- `data_quality=VERIFIED` 只代表事件来源经身份与业务回执校验，不代表模型事实判断绝对正确；必须能查到结果凭证。
- `metrics` 可为空；**成本账本推荐独立 usage 事件**，避免每个状态事件重复累加成本。
- 示例字段全部是虚构演示值；正式错误码、字段允许集和签名规范需在 API v1 中冻结。

## 5. 建议的标准事件类型

``text
TASK_ACCEPTED
RUN_STARTED
STEP_FINISHED
HUMAN_REVIEW_REQUESTED
HUMAN_REVIEW_RESOLVED
RUN_FINISHED
TASK_OUTCOME_CONFIRMED
TASK_FAILED
TASK_CANCELLED
USAGE_RECORDED
EMPLOYEE_HEARTBEAT
ALERT_RAISED
ALERT_RESOLVED
``

建议先实现严格状态转移表：新建 → 执行中 → 等待人工 / 重试 → 成功 / 失败 / 取消。业务回执迟到、任务重开、结果更正需单独补偿/冲正事件而不是直接改原始日志。

## 6. 最小领域模型（表建议）

| 表 | 用途 | 核心列与唯一约束 |
| --- | --- | --- |
| `dw_employee` | 数字员工目录 | `id` PK, `code` UNIQUE, `name`, `scenario_id`, `business_dept_id`, `build_dept_id`, `owner_id`, `version`, `lifecycle`, `connection_status`, `verified_at` |
| `dw_task` | 业务工作单 | `id`, `employee_id`, `source_system`, `source_task_id`, `task_type`, `status`, `accepted_at`, `terminal_at`, `parent_task_id`; UNIQUE (`source_system`, `source_task_id`) |
| `dw_run` | 执行尝试 | `id`, `task_id`, `attempt_no`, `status`, `started_at`, `ended_at`, `trace_id`; UNIQUE (`task_id`, `attempt_no`) |
| `dw_event_inbox` | 原始事件与幂等 | `id`, `source_system`, `source_event_id`, `event_type`, `occurred_at`, `received_at`, `payload_json`, `process_status`; UNIQUE (`source_system`, `source_event_id`) |
| `dw_usage` | 消耗/成本账本 | `usage_id` UNIQUE, `run_id`, `provider`, `quantity`, `unit`, `price_version`, `amount`, `currency`, `settlement_status` |
| `dw_metric_baseline` | 工时基线与审定 | `employee_id`, `task_type`, `version`, `sample_size`, `manual_seconds`, `review_seconds`, `approved_by`, `effective_from` |
| `dw_alert` / `dw_alert_action` | 告警状态/处置 | 告警去重键、原因、状态、责任人、处理动作与审计 |
| `dw_metric_daily` | 日粒度加速查询 | 按业务日 + 员工 + 任务类型 + 数据质量聚合；保留指标版本 |
| `dw_audit_log` | 治理审计 | 人、动作、目标、前后值摘要、来源、原因、时间 |

所有表应包含适用的租户字段与创建/更新时间；用授权层限制数据查询范围。以上是逻辑模型，不是已验收的生产 DDL。物理主键类型、索引和分区随事件量确定。

## 7. 对外 API 轮廓（待冻结）

```http
POST   /api/v1/ingest/events            # 服务账号签名 + 幂等；支持单条或批量
GET    /api/v1/overview                 # 统一筛选时间与部门，返回KPI与freshness
GET    /api/v1/employees                # 可授权过滤/搜索/分页
GET    /api/v1/employees/{id}           # 目录与统计
GET    /api/v1/employees/{id}/timeseries
GET    /api/v1/tasks                    # 搜索/分页
GET    /api/v1/tasks/{id}/timeline      # task -> runs -> steps -> business result
GET    /api/v1/alerts                  # 筛选/分页
POST   /api/v1/alerts/{id}/ack          # 处置授权 + reason + audit
POST   /api/v1/alerts/{id}/assign       # 指派责任人
POST   /api/v1/alerts/{id}/resolve      # 复核关闭
GET    /api/v1/stream/updates           # SSE，按授权数据范围
```

所有统计 API 响应必须返回 `filters`、`as_of`、`freshness`、`metric_version` 和 `quality`；每个 KPI 附 `numerator/denominator`（适用时）。所有列表实现服务端分页和最小权限。

## 8. 对账与验收 SQL 思路

每天核对：
1. 每源系统的受理业务任务数、完成数与平台 `dw_task` 数量差异；
2. 同一 `source_system/source_task_id` 的重复任务数应为零；
3. `dw_run` 可多条，但任何完成的 `dw_task` 只进入一次任务级统计；
4. 成本 usage ID 不能重复，预算与实际账单差异可解释；
5. 未上报员工显示 STALE，不将“没有错误消息”等同于“运行正常”；
6. T+1 延迟/乱序回补后重新计算受影响日期聚合，有差异审计。
