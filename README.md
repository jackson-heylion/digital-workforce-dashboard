# Digital Workforce Dashboard｜数字员工看板

> **当前已确认 MVP（2026-10-10）**：**Agent 登记 → 采集 Agent 执行日志 → 展示看板**。不做复杂数字员工管理、业务回执或工时/财务对账。
>
> 已确认：**集团管理层优先**；保持原始 HTML 的**页面布局 1:1 复刻**；采用 **Vue 3 + TypeScript + Vite**；**Agent 调用处通过一个标准 HTTP 上报接口采集 Run 日志**。后端框架/数据接入具体实现细节仍是实施建议。当前仓库有原型和方案文档，**尚未完成 Vue 应用、后端服务和生产接入**。

## 指标逐项讨论（未确认不得定稿）

- **[21 · 指标字段逐项确认台账](docs/21-metrics-field-confirmation-register.md)**：按原型总览、员工卡/详情、五场景和监控中心共 **53 项展示/管理条目**，另列 **12 类采集字段、12 项跨指标标准**。每项都有「拟展示、候选计算、拟来源、待确认标准」；**A01/A02/A03/A05–A09/A11 已确认方案 A，A04 确认方案 C**：未归档登记人数、人工标记在用、主归属部门、状态分层；名称/图标/简介、当前版本、技术平台/场景来源和展示型负责人由登记维护；**A09「登记时间」由看板创建 Agent 记录时系统生成**，不当作实际上线时间。**A10 已明确取消场景演示**（新 Vue Demo/正式版均不含五场景页、入口与绑定）；**A11 方案 A 已确认：顶栏仅保留客户端时钟，去掉无真实健康证据的「系统运行中」**；**B01 方案 A 已确认**：今日执行次数按 `started_at` 启动日统计唯一技术 Run（HTTP 补报不重复计数）；**B02 方案 A 已确认**：技术成功率按 `finished_at` 归属，使用 `SUCCEEDED/(SUCCEEDED+FAILED)` 计算，排除 RUNNING/CANCELLED，不等于业务成功或一次通过；**A12 方案 B 已确认**：技术成功率目标可配置但初始不设置任何目标，绝不默认采用原型 ≥95%；只可展示已批准且生效的目标数值，目标配置的权限/范围/生效/达标和具体实现仍待议。**B03 P0 记录范围已确认**：每次 Agent Run 先记录实际输入/输出 Token、计费模型标识、按版本化模型单价计算的该次估算费用（价格版本可追溯）。 **B03-P0-01 选方案 A：模型供应商和模型标识必须来自调用侧针对当次 Run 的实际模型身份，不使用登记默认值推断**；**B03-P0-02 已选方案 A**：仅记录真实模型 SDK/API Usage，输入/输出 Token 都是可选的非负整数，缺值不补零、仍接收 Run，无法计算费用时记为不可计算。**B03-P0-03 已选方案 B：单次模型 ESTIMATED 费用由调用方按真实 Token 和模型单价自行计算并随 Run 上报；看板仅接收、校验、保存，不再作为权威计价方**。**B03-P0-04 已选方案 A：各调用方自行维护有来源、版本化的模型单价快照，调用侧据此计价，不强制共享价格卡**。**B03-P0-05 已选方案 B：设计独立版本的最小化 Run 扩展，使用扁平成本及费率依据字段，不直接把未批准的 v1.1 草案 `cost` 嵌套对象当成 P0 标准**；金额具体字段名、必填、币种/精度、版本号和旧客户端兼容细节尚未批准。**B03-P0-06 已选方案 C：仅校验调用方 ESTIMATED 费用的基础格式，P0 不核验价格或 Token 覆盖及金额偏差；调用方对计价质量负责，格式通过不等于财务账单核验**。**B03-P0-07 已选方案 B：同一 Run 允许费用迟到首次补入；首次金额存储后锁定，普通上报不接受变更，不同金额标记冲突而非覆盖或双计**。**B03-P0-08 已选方案 A：估算费用按单次 Run 的 `started_at` 所在统计日归属；跨日执行及迟到首次补费仍归属原启动日，与 B01 同时间事件但不代表样本/金额覆盖一致**。**B03-P0-09 已选方案 A：集团首页主卡「今日模型估算费用」，以有权统计的实际 Agent 技术 Run 为基本范围汇总调用方 ESTIMATED 模型金额，不按 Agent 当前 A02 正式在用标记/暂停或停用状态自动排除历史已产生费用；缺失成本不记零、不能自称 BILLED**。**B03-P0-10 已选方案 C：各有权访问的运行环境费用单独汇总，RUNNING/SUCCEEDED/FAILED/CANCELLED 只要已有一次保存的 ESTIMATED 金额均可进入各自环境候选合计；不同环境不直接混成一个无环境说明的集团总额**。**当前讨论 B03-P0-11：跨币种金额如何汇总和展示**；RUNNING 首次上报时点、环境编码/灰度预发/默认环境、归档历史权限、币种/精度、迟到日报更新/冻结、费用质量 KPI 准入、无费用与真实零、覆盖率、部门/Agent 明细、授权/审计和正式 Schema 继续待定。P0 不强制逐模型调用明细或账单；多模型/缓存无法准确归属时不可假算。**B03 KPI 的估算/账单分栏、币种、统计日、原型卡展示、额外字段及协议版本仍待确认**。设计见 [22 Token 成本计价方案](docs/22-token-pricing-cost-design.md)；时区/窗口/精度/零分母/迟到/环境/数据质量等继续待确认。
- **协作规则**：每项由项目发起人在对话中确认后才更新决议；此前草案的公式和来源只能作为候选，不得因写入 OpenAPI 或 Issue 就当成产品要求。

## 最新确认：调用处上报 Run 日志

- **[20 · 性能、可扩展性、可迁移性](docs/20-performance-extensibility-portability.md)**：单体起步、吞吐/读写优化、容量压测、版本兼容、跨平台/数据库迁移和分阶段演进。
- **[OpenAPI v1.1 扩展草案](openapi/agent-run-reporting-v1.1-draft.json)**：不修改已记录的 v1.0；新增安全摘要、操作类型、人工介入、费用来源和受控场景计量的**可选字段**（未定稿/未实现）。
- **[19 · 原型全量指标/采集覆盖审计](docs/19-prototype-metrics-coverage-audit.md)**：完整清点四页指标、场景统计、演示口径及 v1 缺失字段。**上报协议升级前应先核对该审计**。
- **[18 · Agent Run 上报标准 v1.0](docs/18-agent-run-reporting-standard.md)**：`POST /api/v1/ingest/agent-runs`、字段校验、Bearer 权限、状态转换、幂等、重试、调用方埋点与验收。
- [OpenAPI 3.1 机器可读接口契约](openapi/agent-run-reporting-v1.json)：便于客户端/服务端对照，规范已入库；**尚无已运行的接口实现**。
- 调用方**结束时上报一次终态即可**，要显示“执行中”再使用同一个 `run_id` 上报 RUNNING；上报失败不应影响 Agent 原业务调用。

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

**原始 HTML 只读 + A10 删减例外**：原 HTML 完整保留供历史对照；**新 Vue UI-F0/UI-F1 不实现五场景演示页、相关跳转和 Agent—场景绑定**（由用户 2026-10-10 明确批准），只保留总览、员工详情和监控三页；其余部分对照原型布局，UI-F1 用真实 Agent Run 指标替换原型不可信经营数字。示例数字仅隔离 Demo 使用。

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

建议后端使用一个轻量的 Spring Boot 服务（可配合现有 Java 基础设施），首期普通 MySQL 及分页/索引足够，不必建设 Kafka/Flink/Doris/复杂微服务。**已确认采集策略：在 Agent 调用处增加统一上报**；暂不建设平台主动拉取或通用 Connector。

## MVP 开发任务

1. **UI-F0**：[FE-00～FE-08](docs/11-frontend-backlog.md)：除取消五场景演示页与相关入口外，原型其余视觉/交互忠实还原，与真实数据接入解耦。
2. **Agent 登记/日志/查询**：参照 [17 号方案](docs/17-agent-log-dashboard-mvp.md) 与对应 GitHub Issues 完成最小后端。
3. **UI-F1**：真实 Agent 目录、Run、趋势、失败日志替代 Demo Provider；保证真实 0 / 无数据 / 延迟不同显示。

当前无生产日志、Token 计费数据或原系统业务回执，因此**不能声称已接入某些原型标记“真实在用”的员工**。

## 文档导航

| 文档 | 状态与用途 |
| --- | --- |
| [17 当前 MVP](docs/17-agent-log-dashboard-mvp.md) | **当前范围最高优先级** |
| [18 Run 上报标准](docs/18-agent-run-reporting-standard.md) / [OpenAPI](openapi/agent-run-reporting-v1.json) | **已确认调用侧上报方式**；v1 仅覆盖 Run 基础状态/耗时/usage |
| [19 原型全量指标审计](docs/19-prototype-metrics-coverage-audit.md) | 成本、工时、人工介入、异常恢复和场景指标缺口 |
| [20 性能扩展迁移设计](docs/20-performance-extensibility-portability.md) / [v1.1 OpenAPI 草案](openapi/agent-run-reporting-v1.1-draft.json) | **新提出的演进建议**；兼容 v1.0，非本期已批准新必填字段 |
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
