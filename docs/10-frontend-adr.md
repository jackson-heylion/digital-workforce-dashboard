# ADR-001｜前端技术路线：Vue 3 + TypeScript + Vite

- **状态：已确认**
- **确认日期：2026-10-10**
- **确认主体：项目发起人在对话中明确选择**
- **适用范围：UI-F0 原型忠实复刻及后续 UI-F1 数据接入**
- **原始参考：[`prototype/index.html`](../prototype/index.html)，Git Blob `92d6df5d024a665cb88fcff9724fcb8ac1787225`**

## 决策

采用 **Vue 3 + TypeScript + Vite**，在独立的 `frontend/` 目录开展工程化；按附件原型**1:1 还原页面结构、样式、信息顺序和已具备的交互**。原始 `prototype/index.html` 必须保留，不当作应用产物输出目录、不修改其内容。

**已经确认**的是前端技术栈与忠实复刻要求；下面的 Vue Router、Pinia、Playwright、ECharts 打包方式、目录命名、验收阈值等是落地实施建议，待在工程任务中通过实际 PoC / 代码评审定稿。原后端技术方案仍属于建议，不因本 ADR 获得自动批准。

## 背景

HTML 原型为单文件 CSS/JS + ECharts CDN，页面由 `showPage()` 切换、数据主要硬编码、每 6 秒模拟动态、工作流文件使用浏览器临时 URL 预览、监控告警只更新前端元素；没有可见的生产 API 或身份鉴权。

原型**两项一级导航**：`总览 · 数字员工 · 效能` (`p0`)、`监控中心` (`p4`)；**两个二级页面**：`p11` 员工详情、`p7` 五场景展示；包含 9 个员工卡示例及 5 个标记为“真实在用”的场景示例。原型中的运营数值**仅为演示样本**，不代表生产状态。

## 实施纪律

1. **先复刻 UI-F0，后治理数据 UI-F1，最后讨论布局迭代 UI-F2**。管理层优先是定位，不能用新大屏模板覆盖原型。
2. 首轮用 Vue SFC 拆分**而不重排 DOM 的视觉层级**；CSS tokens、max-width、图表 option、文字和交互状态必须从原型映射，避免凭想象复刻。
3. 原型的模拟 `emps/perfData/genTrend/pushFeed` 必须进入**仅演示环境**的 Demo Provider。生产 Provider 缺失数据只能输出 `UNAVAILABLE / STALE / ESTIMATED` 等显式质量状态。
4. 在 Demo 环境必须可零后端启动、可重现截图；生产构建默认无自动造数开关，并阻止随意切换到 Demo Provider。
5. 不因更换框架而新增大规模微服务、重构现有 Agent 编排或改变业务审批权责。

## 技术选型分级

| 项目 | 结论 | 说明 |
| --- | --- | --- |
| Vue 3 | **已确认** | Composition API + SFC 建议实践 |
| TypeScript | **已确认** | 领域模型、接口响应与组件 props 统一类型 |
| Vite | **已确认** | 开发与构建工具，构建输出至 `frontend/dist` |
| ECharts | **沿用原型图表** | 实现建议作为前端依赖本地打包，不依赖运行时外部 CDN |
| Vue Router 4 | **推荐** | 源原型使用 DOM 显隐；路由映射仍需保留两级导航的视觉语义 |
| Pinia | **可选** | 如跨页面员工选择/筛选等状态复杂才引入；不强制为少量状态新增库 |
| 测试 | **推荐 Vitest + Playwright** | 分别做格式化计算单测、交互和多视口视觉回归 |
| Node/package manager | **按团队受支持版本固定** | 在代码工程生成时明确 `engines` 与 lockfile；暂不凭空锁定未经验证的版本 |

## 构建与预览

- Demo：`frontend/` 本地开发服务器 + mock Provider，静态构建可供内部预览；`prototype/index.html` 作为比对页。
- 生产：同一 Vue 组件树 + 真正 API Provider（有 SSO/授权/数据质量），由公司内部静态站点/网关托管。
- **推荐使用 Router History 模式时**生产 Ingress 配置 SPA fallback 到 `index.html`；若选 GitHub Pages 静态预览，可用 Hash History 或另外配置静态路由。此为部署环境差异，不影响页面设计。
- 公司代码/资料的公开许可需团队确认；**GitHub 仓库公开不等于适合部署生产或公开演示**。

## 取舍/不选方案

- 继续在原单文件叠加生产逻辑：初始化最快，但全局变量、局部 DOM 修改、指标演示与业务事实耦合，不适合持续接入。
- 完全重设计管理驾驶舱：背离“直接还原布局”的明确要求，本阶段不采用。
- 一开始接入所有业务与大数据引擎：不可验证且改变里程碑，留在 UI-F1/后端评审。

## 成功判定（ADR 级别）

- `prototype/index.html` 的 Git Blob 与基线一致；新工程**独立**编译。
- Vue 工程独立启动、生产构建通过；原型四页和五场景内容齐全。
- 同一模拟数据、窗口尺寸、时间与浏览器版本下，主要区域布局和交互可与原型对照验收；不得只有截图而没有按钮/上传/告警行为。
- Demo/Production 不交叉污染，缺数据不回退演示 KPI。
- 详细的开发任务、页面映射和质量门禁见 [11](11-frontend-backlog.md)、[12](12-frontend-interactions.md)、[13](13-ui-acceptance.md)。

## 决策变更

技术栈变更须新增 ADR；布局改动需根据 UI-F2 评审另记，不可仅修改本 ADR 抹掉原决定。
