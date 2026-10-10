# Digital Workforce Dashboard｜数字员工看板

> **当前已确认 MVP（2026-10-10）**：**Agent 登记 → 采集 Agent 执行日志 → 展示看板**。不做复杂数字员工管理、业务回执或工时/财务对账。
>
> 已确认：**集团管理层优先**；保持原始 HTML 的**页面布局 1:1 复刻**；采用 **Vue 3 + TypeScript + Vite**。后端框架/数据接入具体实现细节仍是实施建议。当前仓库有原型和方案文档，**尚未完成 Vue 应用、后端服务和生产接入**。

## 从这里开始

- **[17 · 极简 Agent 看板 MVP（当前范围，以此为准）](docs/17-agent-log-dashboard-mvp.md)**：功能范围、字段、最小 API、数据口径、开发顺序与安全边界。
- **[原始可交互 HTML](prototype/index.html)**：严格保留，不修改；设计和演示的视觉对照基准。
- [原型查看说明](prototype/README.md)：`python3 -m http.server 8000 --directory prototype`，访问 `http://localhost:8000/`（图表需要可访问 ECharts CDN）。
- [ADR-001 · 前端技术路线](docs/10-frontend-adr.md) | [UI 复刻规范](docs/08-ui-reproduction.md) | [Vue 开发任务](docs/11-frontend-backlog.md) | [GitHub Issues](https://github.com/jackson-heylion/digital-workforce-dashboard/issues)。

## 只做三件事

| 功能 | 最小能力 | 不做 |
| --- | --- | --- |
| **Agent 登记** | 新增/编辑/停用；名称、平台、外部 Agent ID、部门、负责人、说明 | 多能力绑定、流程编排、上岗双审批、资产自动发现 |
| **Agent 日志采集** | HTTP 上报或按源平台做简单只读适配；Run ID、状态、开始/结束时间、耗时、可选错误/Token；分页查看 | 源业务系统回执整合、复杂 Connector 系统、事件对账、全量 Prompt 存储 |
| **看板展示** | 注册 Agent 数、执行次数、技术成功率、平均耗时、执行趋势、失败记录、Agent 详情 | 业务成功率、节省工时、ROI、未核验费用 |

**MVP 最简口径**：**一条登记 Agent = 看板一张数字员工卡；执行次数 = Agent Run 次数**。Run 的成功/失败是**技术执行结果**，不是业务系统最终交付。Agent 登记“启用”不等于最近有运行活动，首页标记最近采集时间。

**原型 1:1 与真实口径不冲突**：UI-F0 先照原型四页、两一级导航及五个静态演示场景复刻；UI-F1 在**相同布局**换上实际日志数据，把原型“节省工时/成本/任务”等不可靠经营含义改为执行次数、平均耗时、失败数量或未接入状态。示例数字只在隔离 Demo 环境出现。

## 最小架构（建议，非已批准后端选型）

```text
已有 Agent 平台 / 自研 Agent 服务
          │  HTTPS Run 摘要上报（或单个只读适配器）
          ▼
   Agent 日志接收 API
          ▼
   MySQL: Agent / Run / (可选 Log)
          │
          ├── Agent 登记/详情 API
          ├── Run 日志查询 API
          └── Dashboard 统计/趋势 API
          ▼
    Vue 3 + TypeScript + Vite + ECharts
```

建议后端使用一个轻量的 Spring Boot 服务（可配合现有 Java 基础设施），首期普通 MySQL 及分页/索引足够，不必建设 Kafka/Flink/Doris/复杂微服务。是否真的需要 Agent 平台主动上报、由调用方上报还是只读拉取，取决于现有源系统能力。

## MVP 开发任务

1. **UI-F0**：[FE-00～FE-08](docs/11-frontend-backlog.md)：原型视觉/交互忠实还原，与真实数据接入解耦。
2. **Agent 登记/日志/查询**：参照 [17 号方案](docs/17-agent-log-dashboard-mvp.md) 与对应 GitHub Issues 完成最小后端。
3. **UI-F1**：真实 Agent 目录、Run、趋势、失败日志替代 Demo Provider；保证真实 0 / 无数据 / 延迟不同显示。

当前无生产日志、Token 计费数据或原系统业务回执，因此**不能声称已接入某些原型标记“真实在用”的员工**。

## 文档导航

| 文档 | 状态与用途 |
| --- | --- |
| [17 当前 MVP](docs/17-agent-log-dashboard-mvp.md) | **当前范围最高优先级** |
| [10 前端 ADR](docs/10-frontend-adr.md) | 已确认 Vue 3 + TS + Vite |
| [08 UI 基准](docs/08-ui-reproduction.md) / [12 交互](docs/12-frontend-interactions.md) / [13 测试](docs/13-ui-acceptance.md) | UI-F0 原型还原参考；UI-F1 部分旧的业务核验要求需按 17 修订 |
| [11 开发清单](docs/11-frontend-backlog.md) / [05 决策及里程碑](docs/05-roadmap-and-decisions.md) | 当前与后续任务 |
| [01 原型审阅](docs/01-prototype-review.md) | 原型事实，不代表生产数据 |
| [02 产品需求](docs/02-product-prd.md) / [03 架构](docs/03-architecture.md) / [04 指标字典](docs/04-metrics-and-contracts.md) | 按极简 MVP 收缩后的建议 |
| [06 管理层扩展](docs/06-management-dashboard.md) / [07 原数据架构](docs/07-executive-data-architecture.md) / [09 工程化讨论](docs/09-frontend-data-implementation.md) | 历史扩展设想，**不是 MVP 必须实现** |
| [14 复杂员工管理](docs/14-employee-management.md) / [15 复杂数据接入](docs/15-data-integration.md) / [16 五场景盘点](docs/16-source-inventory.md) | 旧方案存档，**不用于首期开发验收** |

## 安全提醒

此仓库**公开**，其中依项目发起人要求上传了完整原型，包含集团内部业务流程说明；请确认公开许可，必要时改私有。实际生产日志/Prompt/Token/密钥、账号、业务明细与客户个人数据都不得提交仓库或公开演示。正式日志接收和查看必须认证授权并默认脱敏。

## License

尚未选择许可证。仓库公开可见不等于已授权对原型或公司业务内容进行再分发。
