# 09｜从 HTML 原型到管理层看板：前端架构与数据接入实施讨论

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
| 页面切换 | `showPage(id)` 控制 `.page.on`，四个页面 `p0 / p11 / p7 / p4`；二级页高亮 `p0` 导航 | Vue Router 4（或轻量状态路由）复刻；建议 URL 可直达员工详情与场景，而视觉表现不变 |
| 员工目录 | `emps` 内联 9 条数据，按信息/财务/采购/内审/运营部门分组 | 迁移到 `demoProvider.ts`，每人分配稳定 ID，避免按中文昵称连接多表 |
| 场景演示 | 五段静态业务说明、流程图和锚点跳转 | 第一轮将既有 DOM 内容移到独立组件；P1 才由后台驱动业务文档 |
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
    pages/ScenarioDemoPage.vue     # 原 p7
    pages/MonitoringPage.vue       # 原 p4
    components/
      KpiCard.vue
      EmployeeCard.vue
      EmployeeGroup.vue
      ChartPanel.vue
      ExecutionFeed.vue
      ScenarioFlow.vue
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
4. `ScenarioDemoPage`：五个标杆场景内容完整复刻与锚点；
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
- 保证 `prototype/index.html` 不变，`frontend/` 独立构建，保留原型四页及五个场景；
- 比对 1440/1280/768/390/375px 截图，固定与原型一致的数据/时间快照；
- 通过导航、员工详情、图表加载与 resize、工作流本地预览、告警演示测试；
- 即使未接后端，Demo 可独立部署和预览，明确显示“演示环境”。

**迭代 B · 数据接入与真实性：**
- 与至少一类业务任务的真实 `source_task_id` 与结果回执跑通；
- 仅在有证据时显示真实成功率、成本和工时；不可获取则显示“未接入/待核验”；
- 业务数据权限、日志脱敏、导出权限、告警操作后端审计完备；
- 接入中/未验证员工与生产上线员工明确区分。

**进入下一阶段的门槛**：截图/交互对齐 + 用户确认无擅自改版；随后再讨论 KPI 重排和管理层的决策洞察体验。
