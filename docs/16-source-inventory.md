# 16｜源系统与数字员工接入盘点表（等待负责人填写）

> **历史方案，已退出 2026-10-10 确认的 MVP 范围。** 用户明确要求系统只做**Agent 登记、Agent Run 日志采集、看板展示**；本文关于业务 Task/Outcome、Connector 管理、审批/能力绑定、财务工时/ROI、五场景源业务对账或管理层 KPI 重排的建议**均不可当成本期开发需求或验收项**。请以 [17 最新 MVP 规格](17-agent-log-dashboard-mvp.md) 为准；原始 HTML 的 UI-F0 视觉复刻仍保持。


> **本表是业务/研发讨论工作底稿**，用于确定首批真实接入能力。所有“待核实”都是真实的未知项，并非已验证的生产事实。**勿在公开仓库填写实际 Token、密钥、内网 URL、真实员工姓名、业务单据内容或敏感数据**；这些只能在有权限的内部系统收集。
>
> 参考：[14 管理中心](14-employee-management.md) · [15 数据接入规范](15-data-integration.md)。

## A. 数字员工台账的基础范围（尚未验证）

原型展示 9 个示例员工，含 5 个原型声称“真实在用”的场景。它不是可直接导入的正式台账。启动正式登记前须由业务 Owner 校验：

| 项 | 需要确认的答案 | 当前 |
| --- | --- | --- |
| 数字员工定义 | 是否按业务职责/服务区分，而非一个 Agent 就算一人？ | 待确认 |
| 登记方式 | 管理员手工、Agent 平台自动发现、混合？ | 待确认；建议混合 |
| 员工编码 | 是否已有公司统一数字员工 ID/编码规范？ | 待确认 |
| 业务/技术 Owner | 是否一个业务 Owner + 一个技术 Owner？ | 待确认 |
| 执行平台 | 现有平台类别与测试/生产环境有哪些？ | 待确认 |
| 上岗审核 | 业务结果、技术回执、费用数据分别谁签认？ | 待确认 |
| 员工部门 | 主要服务部门与技术建设部门能否区分？ | 待确认 |
| 历史迁移 | 是否需要导入已上线历史执行流水与成本？ | 待确认 |

## B. 五个场景的数据源初盘（均待独立验证）

| 场景 | 建议统一任务单位 | 需要查明的业务结果系统 | 当前已知可靠 ID | 可用接口/事件 | 初始 Owner/接入状态 |
| --- | --- | --- | --- | --- | --- |
| 智能报销 | 一项费用提单委托 | 费控平台受理回执；后续审核/支付阶段独立 | 待核实 | 待核实 | 待核实 / 未验证 |
| 菜品调整 | 一项变更申请 | 菜品业务服务写入与所需生效/分发回执 | 待核实 | 待核实 | 待核实 / 未验证 |
| 采购情报 | 一次查询/一份报告（分类） | 报告交付/可用性确认 | 待核实 | 待核实 | 待核实 / 未验证 |
| 门店巡检 | 门店 × 周期 × 计划 | 巡检完成/采集校验结果 | 待核实 | 待核实 | 待核实 / 未验证 |
| 稽查排班 | 一个排班周期发布单 | 负责人审核与发布系统的回执 | 待核实 | 待核实 | 待核实 / 未验证 |

不要把原型或文档给出的“成功定义建议”直接当成源系统已经有相应 API。建议先对菜品调整、智能报销做一次真实样本核验，因为二者分别覆盖同步写入与异步人审。

## C. 每个源系统必填的接入调查单（复制为一份）

```yaml
source_system: "<非敏感的系统代码>"
environment: "DEV / TEST / PROD"
business_owner_role: "<岗位，不填真实姓名>"
technical_owner_role: "<岗位，不填真实姓名>"
connector_mode: "WEBHOOK / KAFKA / READ_ONLY_PULL / OTHER / UNKNOWN"
source_resource_type: "AGENT / WORKFLOW / SERVICE / JOB / RULE_ENGINE / OTHER"
source_task_id_field: "<字段代码或待确认>"
source_run_id_field: "<字段代码或待确认>"
source_event_id_field: "<字段代码或待确认>"
business_receipt_id_field: "<字段代码或待确认>"
business_success_condition: "<业务签署标准或待确认>"
has_human_approval: "YES / NO / UNKNOWN"
has_reversal_or_correction: "YES / NO / UNKNOWN"
supports_incremental_or_replay: "YES / NO / UNKNOWN"
average_task_volume_per_day: "<脱敏区间/未知>"
event_lag_expectation: "<同步时延/未知>"
usage_or_billing_source: "<费用凭证来源/未知>"
manual_time_baseline: "<抽样/核验来源/未知>"
can_reconcile_to_source: "YES / NO / UNKNOWN"
sensitive_data_risks: "<类型，不填真实值>"
status: "UNKNOWN / CONTACTED / SAMPLE_RECEIVED / VERIFIED"
```

## D. 推荐第一轮验证样本（不上传公开仓库）

业务/技术 Owner 在权限允许的环境提供**脱敏后的结构字段及验收结论**，不提供原文账单或密钥：

1. 成功任务一份：请求 ID → Run/工具链 → 最终业务回执；
2. 失败/拒绝任务一份：Agent/技术执行结果与业务失败分离；
3. 人工等待/审批任务一份：含等待、复核与处理时间；
4. 重试任务一份：多 Run 对应唯一业务 Task；
5. 撤销/更正或超时任务一份：确认如何回补统计。

**有条件才提供**：源端按日完成量摘要与价格账单概要（均脱敏且在内部安全环境处理）。不存在某种场景时标“不适用”，不可编造用例。

## E. 第一轮“是否可接入”的判定

| 判定 | 需要满足 |
| --- | --- |
| `CAN_VERIFY_OUTCOME` | 有稳定任务 ID + 可信源业务回执 + 状态/时间语义明确，可与平台对账 |
| `TECH_ONLY` | 只有 Agent/Workflow Run、Token、调用日志，没有最终业务结果 |
| `CAN_PULL_READONLY` | 暂无事件接口，但可通过经过授权的只读 API/增量同步取得最终业务结果 |
| `UNAVAILABLE` | 缺权限/ID/样本/数据安全准入，暂不可接入真实经营汇总 |

技术指标与商业指标分别验真：`TECH_ONLY` 可以暂时展示技术运行量，但必须明确标注“不代表业务任务完成”，不能冒充 `CAN_VERIFY_OUTCOME`。

## F. 待项目发起人优先拍板的最小问题

> **一名数字员工应代表一个业务服务，还是一个 Agent/Workflow 实例？**
>
> 推荐“一个业务服务为一名数字员工；可绑定多个 Agent/Workflow，底层资源可复用，但业务任务要唯一归属”。这个决定直接影响数据库主键、接入映射、集团上线数和指标去重。

第二问（第一问确认后）：数字员工登记是手工、自动发现还是混合？推荐业务登记 + 自动候选发现/绑定，不从 Agent 数量自动生成正式员工。
