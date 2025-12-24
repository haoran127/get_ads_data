# Google Ads 游戏发行团队 SOP 与 KPI（D7 ROAS + 付费事件）

> 官方文档入口（资源总览）：[Google Ads API v22 overview](https://developers.google.com/google-ads/api/reference/rpc/v22/overview)

## 0. 适用范围与目标

- **适用范围**：游戏发行团队（iOS + Android），使用 Google Ads 作为主要投放平台。
- **回本口径**：**D7 ROAS**。
- **转化定义**：**付费事件**（以付费价值/收入为 ROAS 分子）。
- **核心目标**：在满足 D7 ROAS 门槛前提下，最大化 **达标花费（Qualified Spend）**。

---

## 1. Google Ads API 能力整理（按发行团队“能做什么”分类）

Google Ads API v22 的能力对发行团队可归纳为两类：**查询/报表**（看清楚）与 **变更/执行**（做动作），并通过 **审计/风控**闭环。

### 1.1 查询与报表（你用来“看清楚发生了什么”）

用 GoogleAdsService 查询各类资源与指标，服务于 D7 ROAS 的拆解、放量空间识别与止损：

- **层级表现（放量/止损的主战场）**
  - 账户：`Customer`
  - 系列/预算：`Campaign`、`CampaignBudget`
  - 广告组：`AdGroup`
  - 广告：`AdGroupAd`、`Ad`
- **素材与素材组合（素材驱动增长核心）**
  - 素材库：`Asset`
  - 素材表现：`AdGroupAdAssetView`
  - 素材组合表现：`AdGroupAdAssetCombinationView`
- **搜索与意图（适用 Search）**
  - 关键词：`KeywordView`
  - 搜索词：`SearchTermView`
- **关键维度拆解（找出“达标放量空间”）**
  - 地理：`GeographicView`、`UserLocationView`
  - 人群：`AgeRangeView`、`GenderView`
  - 时段：`AdScheduleView`
  - 点击与质量排查：`ClickView`
- **转化与价值口径（确保 D7 ROAS 一致）**
  - 转化定义：`ConversionAction`
  - 转化价值规则：`ConversionValueRule`、`ConversionValueRuleSet`

### 1.2 账户与投放对象管理（你用来“搭结构、上量”）

- **账户与权限**：`CustomerClient`（MCC 多账号）、`CustomerUserAccess`
- **系列与预算**：`Campaign`、`CampaignBudget`
- **广告组与广告**：`AdGroup`、`AdGroupAd`、`Ad`
- **素材挂载与结构**：`CampaignAsset`、`AdGroupAsset`、`AssetGroup`

### 1.3 出价与策略（你用来“稳住 D7 ROAS 再放大花费”）

- 出价策略：`BiddingStrategy`
- 季节性调整：`BiddingSeasonalityAdjustment`
- 策略预估/对比：`BiddingStrategySimulation`
- 平台建议池：`Recommendation`（建议进入候选池，人审后执行）

### 1.4 实验与规模化执行（你用来“更快验证并复制胜利模式”）

- 实验：`Experiment`、`ExperimentArm`
- 大规模异步变更：`BatchJob`

### 1.5 审计与风控（你用来“可追溯、可回滚、可问责”）

- 变更追踪：`ChangeEvent`、`ChangeStatus`

---

## 2. 度量口径（必须写进团队 SOP）

### 2.1 D7 ROAS 定义

- \(D7\ ROAS = \frac{D7\ 付费收入}{广告花费}\)
- **分母（花费）**：Google Ads 消耗（统一时区/币种/时间窗）。
- **分子（收入）**：付费事件价值（建议明确是否包含订阅续费、税前/税后口径）。

### 2.2 转化定义：付费事件（ConversionAction）

建议至少维护 2 套可解释口径（用于增长与风控并重）：  
- **付费价值（Value）**：ROAS 分子  
- **付费人数/首购人数**：判断“量是否健康”，避免鲸鱼用户驱动的假象

### 2.3 iOS + Android 的执行差异（团队规则层面）

- **Android**：通常更细粒度，可更快做日内调度与放量。
- **iOS（隐私归因）**：常见延迟与聚合更明显，SOP 要求：
  - 放量更阶梯化（更小步、更长观察）
  - 决策窗口更长
  - 用领先指标辅助判断（早期付费率、首购等）

---

## 3. 团队分工（角色职责 + RACI）

### 3.1 推荐角色配置

- **Growth Lead（增长负责人）**：目标/预算框架、实验路线、放量审批、跨部门对齐（产品/变现/创意/数据）。
- **UA Channel Owner（渠道投放负责人）**：结构搭建、日内调预算/出价、异常处理、实验执行与放量。
- **Creative Strategist（创意策略）**：素材矩阵与假设库、标签体系、测试设计、胜利主题复用。
- **Design/Motion（制作）**：素材产能与模板化迭代。
- **Analyst/BI（数据分析）**：口径统一、看板/告警、实验评估、对账与复盘。
- **Ads Ops/MarTech（可选）**：自动化拉数、规则引擎、批量变更、审计与回滚、权限治理。

### 3.2 RACI（简版）

| 工作项 | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| D7 ROAS 目标与预算框架 | Growth Lead | Growth Lead | UA/BI/变现 | 全团队 |
| 系列结构与日内调度 | UA | UA Lead/Growth | BI/创意 | Growth |
| 素材矩阵与测试计划 | Creative Strategist | Creative Lead/Growth | UA/BI | 全团队 |
| 数据口径/看板/告警 | BI | BI Lead | UA/Growth | 全团队 |
| 自动化与审计/回滚 | Ads Ops | Ads Ops Lead | UA/BI | Growth |

---

## 4. 工作标准（SOP）

### 4.1 命名规范（强制）

统一命名保证“可筛选、可复盘、可自动化”：

- **Campaign**：`Game|Platform|Geo|Goal(D7ROAS)|Channel|Audience|Bidding|Theme|Ver|YYYYMMDD`
- **AdGroup**：`Geo|Audience|Intent|BidLayer|Theme|Ver`
- **Ad/AdGroupAd**：`Theme|Hook|Format|CTA|Ver`
- **Asset**：`Theme|Angle|Hook|Format|Length|Creator|Ver|YYYYMMDD`

### 4.2 测试与结论标准（强制）

- **一次只改一个主变量**：主题/钩子/受众/出价策略/落地页/时段/地域。
- **最小数据门槛**：未达最小曝光/点击/付费样本，不下“胜/负”结论。
- **结论三选一**：扩大、继续观察、终止（必须写原因）。

### 4.3 变更管理与审计（强制）

任何可能显著影响花费或学习的变更（预算/策略切换/结构大调/批量换素材）必须记录：  
- **原因**、**预期影响**、**观察窗口**、**回滚条件**  
并用 `ChangeEvent` 做可追溯复盘。

### 4.4 实验标准（建议）

用 `Experiment`/`ExperimentArm` 或严格对照组；实验设计必须提前写清：  
- 目标指标（D7 ROAS、达标花费）  
- 主要变量  
- 运行时长与最小样本  
- 决策阈值（胜/负/平）  

---

## 5. 工作流程（周循环 + 日内节奏）

### 5.1 周循环（建议）

- **周一：目标与预算框架**（D7 ROAS 门槛 + 国家/平台预算上限 + 止损线），输出本周实验清单（结构/策略/素材）。
- **周三/周四：实验中期评估**，只给三类结论：扩大 / 继续 / 终止。
- **周五：复盘与沉淀**，输出“可复制模板”（胜利主题、胜利结构、胜利策略、胜利国家组合）。

### 5.2 日内节奏（建议两次控盘 + 一次推进）

- **上午控盘（守门）**：看花费节奏、D7 ROAS 预警、异常（转化中断/成本暴涨）→ 优先用 `CampaignBudget` 止血或加码。
- **下午推进（放量与测试）**：达标单元阶梯放量；未达标单元止损/改造；推进素材上新与替换。
- **晚间复核（可选）**：检查新增素材/新系列学习状态，安排次日动作。

---

## 6. 管理职能（把“管理动作”变成可规模化流程）

### 6.1 投放治理（结构与合规）

- 命名合规巡检（Campaign/AdGroup/Ad/Asset）
- 状态巡检（停投、预算受限、学习异常）
- 重复资产与脏数据清理（重复素材/重复结构）

### 6.2 优化治理（规则化决策）

- 把“达标放量/不达标止损/新素材探索”固化为规则
- 大规模执行优先走 `BatchJob`（可追踪、可回滚）

### 6.3 风控治理（回滚与告警）

- 对“花费异常、付费事件中断、D7 ROAS 预警”设置告警
- 关键变更绑定回滚条件（并用审计复盘）

---

## 7. KPI 体系（D7 ROAS + 达标花费）

### 7.1 北极星指标（项目级）

- **D7 ROAS**（硬门槛）
- **达标花费（Qualified Spend）**：满足 \(D7\ ROAS \ge 目标\) 的花费规模

### 7.2 护栏指标（防止放量牺牲质量）

- 付费人数、首购人数、付费率
- 成本稳定性（CPA/CPI 波动、学习期占比）
- 结构健康（预算受限/学习重置频率）

### 7.3 领先指标（提升优化效率）

- 素材有效率：进入“可放量池”的新素材占比
- 实验胜率：达标且可复制的实验占比
- 响应时效：异常发现 → 处置完成时间
- 迭代周期：brief → 上线

### 7.4 岗位 KPI（建议）

- **UA Channel Owner**：达标花费、放量成功率、异常处置时效、实验执行质量（完成率+胜率）。
- **Creative Strategist/制作**：有效素材率、主题复用贡献、素材迭代周期、高胜率素材组合覆盖率。
- **BI/MarTech**：口径一致性（对账误差）、数据时效、告警覆盖率、自动化覆盖率、变更可追溯率。

---

## 8. 各角色 Checklist（可直接执行）

### 8.1 Growth Lead（每周）

- [ ] 设定 D7 ROAS 门槛与预算框架（按平台×国家）
- [ ] 审批本周实验清单（结构/策略/素材）与验收口径
- [ ] 审批放量阶梯与止损线（明确回滚条件）
- [ ] 周五复盘：沉淀可复制模板与下周优先级

### 8.2 UA Channel Owner（每日）

- 上午控盘
  - [ ] 花费节奏 vs 预算
  - [ ] D7 ROAS 预警（含领先指标）
  - [ ] 付费事件是否中断
  - [ ] 预算受限/学习异常排查
- 下午推进
  - [ ] 达标单元按阶梯放量（预算优先，其次出价/策略）
  - [ ] 未达标单元止损/改造（一次只改一个主变量）
  - [ ] 上新素材并完成标签与归档
- 复盘记录
  - [ ] 关键变更写清原因/预期/回滚条件（便于 ChangeEvent 对齐）

### 8.3 Creative Strategist（每日/每周）

- [ ] 维护素材矩阵：主题×钩子×格式×受众
- [ ] 产出本周素材测试计划（变量、样本门槛、决策阈值）
- [ ] 每日复盘素材表现：高胜率主题/组合→推广；低胜率→终止并总结原因
- [ ] 建立“胜利主题库/失败原因库”，推动复用

### 8.4 BI（每日/每周）

- [ ] 维护 D7 ROAS 看板与达标花费拆解（平台×国家×层级×素材）
- [ ] 异常告警：花费暴涨/暴跌、付费事件中断、ROAS 预警
- [ ] 实验评估模板：胜/负/平标准化
- [ ] 周复盘：贡献拆解（结构 vs 素材 vs 策略）

---

## 9. “问题 → 用哪些 API 资源”速查表

| 业务问题 | 优先看哪些资源/视图 | 常见动作（变更） |
|---|---|---|
| 哪些系列能在达标前提下继续放量？ | `Campaign`、`CampaignBudget` | 调 `CampaignBudget`、调整 `BiddingStrategy` |
| 哪些广告/广告组拖累 D7 ROAS？ | `AdGroup`、`AdGroupAd` | 暂停 `AdGroupAd`/`AdGroup` 或改出价 |
| 哪些素材主题贡献最大？ | `Asset`、`AdGroupAdAssetView` | 用 `AdGroupAsset` 推广素材到更多组 |
| 哪些素材组合胜率最高？ | `AdGroupAdAssetCombinationView` | 复制组合并做 A/B（必要时 `Experiment`） |
| 谁在何时改了预算导致 ROAS 波动？ | `ChangeEvent`、`ChangeStatus` | 回滚/修正并补齐变更记录 |
| 如何大规模上新结构/素材？ | `BatchJob` + mutate | 异步批量创建/更新 |

---

## 10. 与本目录其他文档的关系

- 架构与工程化落地参考：`google/AD_AND_ASSET_OPTIMIZATION_ARCHITECTURE.md`
- 快速优化策略参考：`google/OPTIMIZATION_QUICK_GUIDE.md`
- 更完整资源能力清单：`google/GOOGLE_ADS_API_CAPABILITIES.md`
