> **B11 方案 A 已确认（2026-10-11）**：集团跨历史累计只能按有权限查询的**真实已采集唯一 Agent 技术 Run** 汇总，不以 UI-F0 九名员工的演示累计相加、API 请求数或缺失的 Agent 全生命周期执行估值替代。后端唯一 Run 作为来源，历史接入起点/迁移完整性、状态/时区/环境与归档权限、迟到补报、前端展示与集团/单 Agent 对账仍待批准；**C01 员工卡累计执行为下一项待确认指标**。原始 HTML 与 v1/v1.1 不变。

> **B10 方案 A 已确认的正式看板数据边界（2026-10-11）**：今日技术 Run 次数以**服务端 B01 去重聚合**为权威，正式 Provider **自动刷新读取服务端快照**，绝不将模拟 DemoTicker、B06 动态文本数量、RUNNING→终态或重传事件用来本地加一。旧文「生产低频刷新」「轮询/SSE」仅为历史方案，**未批准具体轮询频率、SSE/WebSocket、缓存水位、查询失败、页面重新挂载去重、延迟/时区/权限/审计**。详见 [21 台账](21-metrics-field-confirmation-register.md)。 下一项 **B11 全集团历史累计运行量**另行讨论；本 B10 自动刷新不确认历史全量统计。

> **A10 最新用户裁决（2026-10-10）**：场景演示整体取消；以下原型拆解仅用于历史了解。新 Vue UI-F0/UI-F1 不建设 `p7`、`ScenarioDemoPage`、场景演示入口、五场景静态流程/绑定；只做 p0/p11/p4 三页，其余原型视觉继续参考，原始 HTML 不改。以 [21 决策台账](21-metrics-field-confirmation-register.md) 和 [17 MVP](17-agent-log-dashboard-mvp.md) 为准。

# 09｜从 HTML 原型到管理层看板：前端架构与数据接入实施讨论

> **历史方案，已退出 2026-10-10 确认的 MVP 范围。** 用户明确要求系统只做**Agent 登记、Agent Run 日志采集、看板展示**；本文关于业务 Task/Outcome、Connector 管理、审批/能力绑定、财务工时/ROI、五场景源业务对账或管理层 KPI 重排的建议**均不可当成本期开发需求或验收项**。请以 [17 最新 MVP 规格](17-agent-log-dashboard-mvp.md) 为准；原始 HTML 的 UI-F0 视觉复刻仍保持。


> **状态：前端技术选型已确认（2026-10-10）；具体实现细节与生产接入仍需研发评审。** 参见 [ADR-001](10-frontend-adr.md)、[工程任务清单](11-frontend-backlog.md)。
>
> **已确认**：首期集团管理层优先；`prototype/index.html` 必须保留，作为第一轮 UI 的布局/交互验收基准。详见 [08 UI 复刻规范](08-ui-reproduction.md)。
>
> **讨论目的**：在不推翻现有页面的条件下，确定工程化技术路线、演示与生产数据隔离、页面级 API 契约、逐步验收顺序。当前仓库尚无 `frontend/` 应用或生产 API 代码。

## 1. 本轮结论（推荐而非确认）

1. **不直接在原始 HTML 上叠加生产逻辑**：原文件锁定为参考原型，保持长期可对照。
2. **已确认单独建立 Vue 3 + TS + Vite 前端应用**，完全对照原 DOM/CSS/ECharts 实现 UI-F0；先以 Demo Provider 复刻现有交互和数据，不重排页面。
3. **Demo/Production 使用同一组件、不同数据接口**：Demo 仅演示环境启用，不能自动映射至正式运行状态；Production 没有数据要显示空状态与原因，不能沿用原型“663次”等数值。
4. **后台先围绕业务任务（Task）与最终业务回执建模**，管理层需要业务交付、工时与成本的可信聚合；技术 Run/Step 是下钻证据。
5. **先有可访问的静态原型演示，再有可验证的工程化页面，最后才是实时运营平台**；是否开通 GitHub Pages/内部预览地址仍需确认，上传文件本身不代表已上线。
6. **管理层优先不等于立即改版**：已确认的页面基准高于此前关于“四主卡/日志下移”的候选建议；相关改版留在 UI-F2 独立评审。

## 2. 从原始 HTML 检查到的具体实现事实

| 原型细节 | 代码观察 | 工程化处理 |
| --- | --- | --- |
| 页面切换 | `showPage(id)` 控制 `.page.on`，四个页面 `p0 / p11 / p7 / p4`；二级页高亮 `p0` 导航 | 新 Vue 只保留 p0/p11/p4 三页；可直达员工详情，场景路由已取消 |
| 员工目录 | `emps` 内联 9 条数据，按信息/财务/采购/内审/运营部门分组 | 迁移到 `demoProvider.ts`，每人分配稳定 ID，避免按中文昵称连接多表 |
| 场景演示 | 五段静态业务说明、流程图和锚点跳转 | **该功能已取消**，不移植 DOM，不建设独立组件或后台文档功能 |
| 首页图表 | ECharts CDN 5.4.3，首页工时/场景排行固定数据，成本按 `runs×cost` 算示意数 | 首轮复用相同 ECharts 配置；正式模式必须按统计快照和真实成本账本计算 |
| 个人趋势 | `genTrend(total)` 以固定月度权重拆出 9 个月数据 | 仅 Demo 允许；正式模式由 `GET timeseries` 返回每月事实 |
| 执行动态 | `pushFeed` 每 6000ms 用本地列表循环生成日志、随机增加成本、执行次数 | 封装 `DemoTicker` 并在生产模式禁用；生产默认查询或低频刷新即可 |
| 工作流文件 | 浏览器 `URL.createObjectURL` 预览图片/PDF，不持久化到服务器 | 首轮保持本地预览；正式模式增加上传服务、员工文件关联、版本与安全校验 |
| 告警处置 | 点击“处理/忽略”仅改 DOM，并显示“演示” Toast | Demo 仅显示模拟结果；正式模式采用服务端状态机、授权审计与错误回滚 |
| 图表初始化 | 页面隐藏时不初始化详情图表；切换后初始化，延迟触发 resize | Vue 在组件挂载且可见后初始化，`ResizeObserver`/容器 resize 清理，避免图表空白/内存泄漏 |
| 资源依赖 | 唯一外部资产为 jsDelivr ECharts 5.4.3 | 生产应通过 NPM 打包 ECharts 或公司允许的资源源，离线构建/内网访问测试 |

### 无数据 / 数据源异常与 0 严格区分

| 真实状态 | UI 文案示例 | 不允许 |
| --- | --- | --- |
| 尚未接入 | 未接入 | 显示原型硬编码成功率 |
| 数据已接入但未核验 | 待核验 | 伪装已上线 |
| 指标无法计算 | — / 尚无可核验数据 | 假定等于 0 |
| 已核验且数值确为零 | 0（已核验） | 用“—”掩盖 |
| 数据中断/延迟 | 数据延迟，截至 xx:xx | 浏览器持续模拟增长 |
| 演示环境 | DEMO / 演示数据 | 同正式经营数据相加 |

## 3. 两种前端工程化路线

| 路线 | 做法 | 优点 | 风险与适用性 |
| --- | --- | --- | --- |
| **A. 原 HTML 增量增强** | 基于现有 DOM/JS 增加接口和更多交互 | 静态演示最快，初期改动少 | 全局 DOM/变量与模拟数据耦合严重；状态、权限、详情、回放持续增加后维护困难 |
| **B. Vue 原型忠实重构（推荐）** | 保留原型 CSS/结构/图表配置，逐个组件移植；数据经 Provider 注入 | 长期易维护、易做权限、API 和自动化测试；页面外观可不变 | 要做截图比对，不能“按自己理解”重新设计 |
| C. 另起一套集团大屏 | 用新模板改写首页 | 可换视觉形态 | 与已明确的原型布局还原要求冲突，不推荐作为 UI-F0 |

**最终路线已确认**：选 **B**，采用 Vue 3 + TypeScript + Vite 进行原型忠实重构；保留原 HTML 可直接静态预览。组件划分和依赖方案以 [11 开发计划](11-frontend-backlog.md) 为准。

## 4. 前端工程目录草案

```text
prototype/
  index.html                       # 保留原始附件，不修改
frontend/
  package.json
  vite.config.ts
  src/
    main.ts
    router/index.ts                 # 视觉不变，但支持直达详情
    App.vue
    layouts/DashboardShell.vue
    pages/OverviewPage.vue         # 原 p0
    pages/EmployeeDetailPage.vue   # 原 p11
    pages/MonitoringPage.vue       # 原 p4
    components/
      KpiCard.vue
      EmployeeCard.vue
      EmployeeGroup.vue
      ChartPanel.vue
      ExecutionFeed.vue
      AlertList.vue
      WorkflowUploader.vue
    styles/
      tokens.css                    # 原 CSS :root 的准确值
      prototype.css                 # 迁移视觉规则，不自行重写布局
    domain/
      employee.ts
      task.ts
      metrics.ts
    providers/
      dashboardProvider.ts          # 数据契约
      demoProvider.ts               # 演示数字与模拟推送
      apiProvider.ts                # 正式远程 API
    services/
      http.ts
      dashboard.ts
tests/
  e2e/
    screenshots.spec.ts
    navigation.spec.ts
    demo-interactions.spec.ts
```

### 组件改造的“先后”建议

1. `DashboardShell`：完整 topbar/nav/footer + `max-width:1280px`，保持页面几何形态；
2. `OverviewPage`：四张原型 KPI、执行动态、员工分组、3 张图表，样式先固定；
3. `EmployeeDetailPage`：个人数据联动、3 个详细趋势、文件预览；
4. ~~`ScenarioDemoPage`：五个标杆场景内容完整复刻与锚点~~（**A10 已取消，禁止开发**）；
5. `MonitoringPage`：告警及执行流水；
6. `DashboardProvider` 抽象 + API 适配；最后再依据管理层反馈做 UI-F2。

## 5. 核心的 Data Provider 契约

页面组件不关心是 Demo 还是 Production，统一从下列契约读取数据：

```ts
type MetricQuality = 'VERIFIED' | 'ESTIMATED' | 'UNAVAILABLE' | 'STALE' | 'DEMO';
type Metric<T> = {
  value: T | null;
  quality: MetricQuality;
  asOf?: string;
  reason?: string;
  unit?: string;
};
interface DashboardProvider {
  getOverview(filters: DashboardFilters): Promise<Overview>;
  listEmployees(filters: EmployeeFilters): Promise<EmployeePage>;
  getEmployee(id: string, filters: DashboardFilters): Promise<EmployeeDetail>;
  getEmployeeTrends(id: string, filters: DashboardFilters): Promise<TrendData>;
  listTasks(filters: TaskFilters): Promise<TaskPage>;
  listAlerts(filters: AlertFilters): Promise<AlertPage>;
  // Demo only: receive mock tick events; production could use polling or SSE later.
  subscribeUpdates?(callback: (event: DashboardEvent) => void): () => void;
}
```

真正的后端对接约束参见 [04 指标及事件契约](04-metrics-and-contracts.md)、[07 管理层读模型](07-executive-data-architecture.md)。Provider 不应把变更菜品、支付或审批直接写进看板 API；这属于已有业务平台。

## 6. 与管理层优先的结合方式：保留布局，不保留错误业务语义

**UI-F0 完全复刻：** 顶栏、横向导航、Banner、四张 KPI、实时动态、按部门员工卡、三张趋势/排行；员工详情和监控原样。用于视觉和交互验收。

**UI-F1 上生产数据时，允许必要真实性修正（不重排布局）：**
- 把“系统运行中”改为可校验的采集/运行状态；
- 把“实时执行数”换成真实完成任务数，或仍叫“当日执行”但显示 Task/Run 的准确定义；
- 把无基线的节省工时改为“待核验”，即使原型位置仍在；
- 把“真实在用”替换成经过业务 Owner 验证的上线标记；
- 业务流程图与上传等仍在相同位置，但持久化/权限需要后端支持。

**UI-F2 需单独评审：** 改 KPI 顺序、首页把趋势前置、日志下移、管理层月报、部门贡献视图等。不能悄悄当做 UI-F0 的必做。

## 7. 预览、部署与数据安全边界

```text
静态原型预览  ——  prototype/index.html （公开静态资源，含演示数据）
        |
        +—— 建议单独 Preview 流水线：编译 frontend/ Demo，检查视觉回归
        |
        +—— 正式内部部署：frontend/ build + 企业 Ingress + Spring Boot API
                              + IAM/OIDC + 业务数据鉴权 + MySQL
```

- GitHub Pages 可以**作为公开的原型/演示页面**候选，但不能承载集团业务真实数据、访问令牌或带权限的后台接口。是否开启 Pages 仍待项目所有者决定。
- 如果公司原型内容本身不适宜公开，应先审查仓库可见范围；即使不部署 Pages，公开仓库中的 HTML 已可下载。
- Vite 构建不能通过把 `VITE_*` 值当成 Secret 来存放真实后台凭证；用户端 JS 一定可被检查。
- 部门数据权限靠 Spring Boot/API 实施，前端只做 UX 的权限提示。

## 8. 细化工程时需处理的问题（F1 已确认）

| ID | 问题 | 推荐默认 | 当前状态 |
| --- | --- | --- | --- |
| F1 | 正式前端选增量增强还是 Vue 3 组件化？ | **已确认：Vue 3 + TS + Vite，原型 1:1 还原** | **已确认（2026-10-10）** |
| F2 | 演示页是否上线公开 GitHub Pages？ | 仅在确认公司原型信息可公开后启用公开 Demo；也可先内部预览 | **待确认** |
| F3 | 演示与生产如何隔离？ | 独立环境/构建标识；正式环境无模拟运行模式 | **建议，待确认** |
| F4 | 第一批生产对接哪个业务？ | 菜品调整 + 智能报销，分别覆盖工具调用和长流程 | **建议，待确认** |
| F5 | 导航如何支持 URL 直达？ | 外观不改，内部加路由，员工详情按稳定 `employee_id` | **建议，待确认** |
| F6 | 图表的历史跨度和周期如何设定？ | UI-F0 照原型 1–9 月；UI-F1 后按实际可用数据和筛选周期 | **建议，待确认** |

## 9. 推荐的最近一轮可交付验收

**迭代 A · 视觉与交互复刻：**
- 保证 `prototype/index.html` 不变，`frontend/` 独立构建，新 Vue 仅保留三页（总览/详情/监控），场景演示页及入口已取消；
- 比对 1440/1280/768/390/375px 截图，固定与原型一致的数据/时间快照；
- 通过导航、员工详情、图表加载与 resize、工作流本地预览、告警演示测试；
- 即使未接后端，Demo 可独立部署和预览，明确显示“演示环境”。

**迭代 B · 数据接入与真实性：**
- 与至少一类业务任务的真实 `source_task_id` 与结果回执跑通；
- 仅在有证据时显示真实成功率、成本和工时；不可获取则显示“未接入/待核验”；
- 业务数据权限、日志脱敏、导出权限、告警操作后端审计完备；
- 接入中/未验证员工与生产上线员工明确区分。

**进入下一阶段的门槛**：截图/交互对齐 + 用户确认无擅自改版；随后再讨论 KPI 重排和管理层的决策洞察体验。


### 正式 Provider 的 C06 月度趋势数据映射 · 方案 A 已确认（2026-10-11）

**C06 方案 A 已确认（2026-10-11）**：Agent 详情月度执行趋势，按当前 Agent 实际采集、持久化的历史唯一技术 Run，根据可信 `started_at` 所属统计月份归集，同一 Run 的重传/状态变化不额外计次。不能从 UI-F0 `genTrend` 固定权重拆分累计数、使用静态月份数或将业务 Task 数冒充技术 Run。虽然与 B01/C04 的启动时间方向一致，**并不自动确认与 B01/C01/C04/B10 全部状态、环境权限和历史范围相同**。**统计月份及时区、Run 唯一性/跨迁移、RUNNING/FAILED/CANCELLED、迟到补报、历史重算/冻结和数据完整度、环境与部门权限、停用归档历史、缺失月份/真实零、图表类型精度及月份范围、B01/C01/C04/B10 对账和审计均待确认**。不批准具体 SQL/API、额外 Schema、原型 HTML 修改或上线；C07 详情月度估算节省工时另议。

**工程待确认**：服务端聚合与读取接口、时区/月界、状态过滤和跨源去重规则、补报重算/冻结、零与无数据的视觉含义、授权范围及监控审计必须单独批准。原型演示视觉仍独立于正式真实数据；不能以选择 C06 为由生成静态趋势、修改严格 Run v1 或自动开放 C07。


### C07 详情月度估算节省工时 · 方案 A（2026-10-11）

**C07 方案 A 已确认（2026-10-11）**：Agent 详情保留「月度估算节省工时（h）」趋势，**仅对具备经认可的人工处理基线、可核验的适用业务工作量、必要人工投入扣除及可追溯版本化估算规则的 Agent 月份**展示明确标识为 `ESTIMATED` 的小时数；缺乏可信依据的月份显示「暂无可靠估算／不可估算」，不得填入假零、静态模拟趋势，也不得以技术 Run 次数或 `duration_ms` 直接推算人工节省。与 B07 集团月趋势、B09 Top8、C03 员工卡共享**可信证据门槛方向**，但不默认同一公式或口径，C06 仍是技术执行量而非节省工时。**人工基线来源及审批、业务工作量／任务关联与去重、Run 状态及重试、统计月份及时区、人工参与／复核扣除、估算规则版本／生效及历史重算、估算质量与覆盖率、负值和精度、环境与停用归档权限、缺失月份与真实零、B07/B09/C03/C06 一致性及审计均继续待议**。仅确认指标展示方向，不批准业务 Task/Outcome 采集、基线系统、新 SQL/API、严格 v1/v1.1 协议改动、原型修改或上线。**下一项 C08 详情月度成本**待选。

**待议**：人工基线来源及审批、业务工作量与任务关联去重、Run 状态与重试、月份及时区、人工复核扣除、估算版本与历史重算、质量覆盖率、负值精度、环境归档权限、缺失与真实零、B07/B09/C03/C06 一致性及审计。**下一项 C08 待确认**。


### C08 详情月度成本 · 方案 A（2026-10-11）

**C08 方案 A 已确认（2026-10-11）**：Agent 详情「月度成本」仅按**当前 Agent 真实采集并保存的技术 Run 模型费用**汇总，采用可信 `started_at` 所属统计月份，展示标明 `ESTIMATED` 的金额；**按原币种分别统计和展示**，不同币种不直接相加或换汇，不采用原型固定单均成本×Run 次数。未上报费用与真实零金额须区分，费用覆盖率与质量状态不能隐瞒。与 B08 月趋势、B03 当日费用、B04 平均费用遵循保存费用来源原则，但**各指标分母、窗口、状态和权限并不自动视为一致**。**统计月份和时区、费用来源与版本、Run 唯一性和 RUNNING/FAILED/CANCELLED、迟到补报和历史重算、币种与精度、费用覆盖率、无费用与真实零、环境权限及归档 Agent、B03/B04/B08 对账和审计全部继续待议**。仅批准展示与来源方向，不批准新 API/SQL、汇率机制、协议 v1/v1.1、原型修改或上线。下一项 **C09 工作流图片/PDF 上传与预览** 待确认。

**实施细节待议**：统计月份与时区、费用来源及版本、Run 唯一性和状态、迟到补报/历史重算、币种精度、费用覆盖率、无费用与真实零、环境权限与归档 Agent、B03/B04/B08 对账及审计。


### C09 工作流图片/PDF 上传与预览 · 方案 A（2026-10-11）

**C09 方案 A 已确认（2026-10-11）**：Agent 详情**正式支持受控的工作流图片/PDF 附件登记、上传、替换和预览方向**，使用服务端受控存储与经授权访问；附件仅作为工作流说明资料，**不代表技术 Run 或业务任务完成证据**。仅确认功能方向，**是否纳入当前 MVP 交付、具体文件类型与大小、上传/替换/删除主体及权限、存储方案与访问鉴权、版本与审计、恶意文件检测、敏感信息脱敏、PDF/图片安全预览、历史附件迁移、无附件状态、与 Agent 登记关系全部待议**。不因本决定自动批准上传 API、存储结构、MVP 排期、原型修改、严格 v1/v1.1 协议改动或上线。**下一项 C10 最近执行记录**待确认。

**实施细节待议**：MVP 范围、文件类型大小、上传替换删除权限、存储与访问鉴权、版本和审计、恶意文件检测、敏感信息脱敏、预览安全、历史附件迁移、无附件状态及 Agent 登记关联。
