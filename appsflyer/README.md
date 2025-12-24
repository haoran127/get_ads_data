# 📱 AppsFlyer 移动归因与营销分析平台

## 文档说明

本目录包含 AppsFlyer 平台的全面分析文档，帮助移动广告主理解归因逻辑、API 能力并设计数据驱动的优化方案。

---

## ⚠️ 重要更新提示（开发前必读）

| 变更项 | 说明 | 生效日期 |
|-------|------|---------|
| 🔑 **API V2 Token** | 所有 API 必须使用 V2 Token（JWT Bearer 认证），V1 已停用 | 2023-09-18 |
| 📱 **iOS SDK** | 最新版本 **6.14.6**，需要 Privacy Manifest | 2024-07 |
| 🤖 **Android SDK** | 最新版本 **6.17.1**，支持 Google Play Integrity API | 2025-07 |
| 🍎 **Privacy Manifest** | iOS 应用必须包含隐私清单（SDK 6.13.0+ 内置支持） | 2024-04-07 |

> **开发者注意**：如果您使用的是旧版 API（URL 参数 `api_token=xxx`），必须迁移到 V2 Token 的 Header Bearer 认证方式，否则 API 调用将失败。

---

## 📚 文档结构

### 1. [APPSFLYER_API_CAPABILITIES.md](./APPSFLYER_API_CAPABILITIES.md)
**AppsFlyer API 能力全景分析**

- Pull API / Push API / Server-to-Server API 详解
- 原始数据报告（Raw Data Reports）能力
- 聚合数据报告（Aggregate Data）能力
- Cohort / Retention / LTV API 详解
- SKAdNetwork 数据获取
- Audiences API 与 Data Locker 能力

**适合**：了解 AppsFlyer 数据获取的所有方式

### 2. [APPSFLYER_INTEGRATION_GUIDE.md](./APPSFLYER_INTEGRATION_GUIDE.md)
**AppsFlyer 集成与实施指南**

- SDK 集成最佳实践
- 事件追踪设计与实施
- 深度链接（Deep Linking）配置
- 数据回传与 Postback 设置
- 与广告平台的集成方式

**适合**：技术团队实施 AppsFlyer 集成

### 3. [APPSFLYER_DATA_ANALYSIS_GUIDE.md](./APPSFLYER_DATA_ANALYSIS_GUIDE.md)
**AppsFlyer 数据分析与优化指南**

- 归因数据分析框架
- LTV / ROAS 计算与优化
- 渠道评估与预算分配
- 反作弊数据分析
- 自定义报表设计

**适合**：数据分析师和投放优化师

### 4. [APPSFLYER_POSTBACK_DATA_GUIDE.md](./APPSFLYER_POSTBACK_DATA_GUIDE.md)
**AppsFlyer 回传数据结构与分析指南** 🆕

- 回传数据完整结构解析
- 关键字段说明与用途
- 数据表设计方案
- 分析 SQL 示例（漏斗、ROAS、CTIT）
- 数据质量检查方法

**适合**：数据工程师和后端开发

---

## 🎯 AppsFlyer 核心价值

### 什么是 AppsFlyer？

AppsFlyer 是全球领先的**移动归因与营销分析平台**，帮助广告主：

```
┌─────────────────────────────────────────────────────────────────────┐
│                      AppsFlyer 核心能力                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   🎯 移动归因（Mobile Attribution）                                  │
│      → 追踪用户来自哪个广告渠道/campaign/广告                         │
│      → 支持 iOS（包括 SKAdNetwork）和 Android                        │
│                                                                     │
│   📊 营销分析（Marketing Analytics）                                 │
│      → LTV、ROAS、Retention 等核心指标                               │
│      → 多维度报表与 Cohort 分析                                      │
│                                                                     │
│   🔗 深度链接（Deep Linking）                                        │
│      → OneLink 实现无缝用户引导                                       │
│      → 支持延迟深度链接（Deferred Deep Linking）                      │
│                                                                     │
│   🛡️ 反作弊（Protect360）                                           │
│      → 实时检测虚假安装与点击                                         │
│      → 保护广告预算不被浪费                                          │
│                                                                     │
│   👥 受众管理（Audiences）                                           │
│      → 基于行为创建受众分群                                           │
│      → 导出至广告平台进行再营销                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📈 AppsFlyer 在广告数据链路中的位置

```
广告投放链路：

┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  广告平台     │     │   用户设备    │     │    App       │
│  (投放)      │────▶│   (点击/展示) │────▶│   (安装/事件) │
│              │     │              │     │              │
│ • Meta       │     │ • 点击广告    │     │ • 首次打开    │
│ • Google     │     │ • 查看广告    │     │ • 注册       │
│ • TikTok     │     │ • 跳转应用商店│     │ • 付费       │
│ • Unity Ads  │     │              │     │              │
└──────────────┘     └──────────────┘     └──────────────┘
        │                    │                    │
        │                    │                    │
        ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────┐
│                                                         │
│                    AppsFlyer                            │
│                                                         │
│   1. 采集广告点击/展示数据（通过集成或 Postback）         │
│   2. 采集 App 内安装和事件数据（通过 SDK）               │
│   3. 匹配并归因：这个用户的安装来自哪个广告？             │
│   4. 回传归因数据给广告平台（用于广告平台优化）           │
│   5. 提供报表给广告主（用于分析和决策）                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  广告平台     │     │  广告主BI     │     │  数据仓库     │
│  (回传优化)   │     │  (报表分析)   │     │  (深度分析)   │
└──────────────┘     └──────────────┘     └──────────────┘
```

---

## 🔑 关键概念速览

### 归因（Attribution）

| 概念 | 说明 |
|-----|------|
| **归因窗口** | 用户从点击/查看广告到安装的有效时间范围（如7天点击归因窗口） |
| **Last Click** | 将安装归因给用户安装前最后一次点击的广告 |
| **View-Through** | 将安装归因给用户"看到但未点击"的广告（通常窗口较短，如24小时） |
| **自归因网络（SRN）** | Meta、Google 等平台自己做归因匹配，直接告诉 AppsFlyer 结果 |
| **非自归因网络** | AppsFlyer 收集点击数据并自行做归因匹配 |

### SKAdNetwork（iOS 隐私归因）

| 概念 | 说明 |
|-----|------|
| **Conversion Value** | iOS 提供的 0-63 数值，用于编码用户早期行为（如注册、首充） |
| **Postback** | Apple 在归因窗口结束后发送的归因数据（延迟 24-48 小时） |
| **隐私阈值** | 安装量未达阈值时，部分数据会被 Apple 隐藏 |

### 核心指标

| 指标 | 定义 | 用途 |
|-----|------|------|
| **Installs** | 新增安装数 | 衡量规模 |
| **CPI** | Cost Per Install | 获客成本 |
| **ROAS** | Return on Ad Spend | 广告投资回报率 |
| **LTV** | Lifetime Value | 用户生命周期价值 |
| **Retention** | 留存率（D1/D7/D30等） | 用户质量 |
| **eCPI** | Effective CPI（含 Re-attribution） | 真实获客成本 |

---

## 🛠️ API 能力总览

| API 类型 | 用途 | 数据粒度 | 实时性 |
|---------|------|---------|-------|
| **Pull API** | 主动拉取报表数据 | 聚合/原始 | T+1～实时 |
| **Push API（Postback）** | 实时推送事件到广告主服务器 | 事件级 | 实时 |
| **Data Locker** | 原始数据导出到云存储 | 原始事件 | 近实时（小时级） |
| **Master API** | 账户/App 管理 | 配置级 | 实时 |
| **Audiences API** | 受众创建与导出 | 用户级 | 批量 |
| **SKAN API** | SKAdNetwork 数据获取 | Postback 级 | T+1 |

---

## 📊 典型应用场景

### 场景 1：多渠道 ROAS 对比

```
目标：比较 Meta、Google、TikTok 等渠道的 D7 ROAS

数据来源：
- AppsFlyer Pull API（聚合报表）
- 或 Data Locker + 内部数据仓库

分析维度：
- 渠道 × Campaign × 国家 × 设备
- 时间趋势（日/周/月）

输出：
- 各渠道 ROAS 排名
- 预算分配建议
```

### 场景 2：归因数据回传与广告优化

```
目标：将后端真实付费数据回传给广告平台，优化出价

数据流：
1. AppsFlyer SDK 上报安装和事件
2. 内部系统产生付费订单
3. 通过 S2S API 将付费事件发送给 AppsFlyer
4. AppsFlyer 回传给 Meta/Google（自动 Postback）
5. 广告平台用此数据优化投放

关键配置：
- 事件名映射（如 internal "order" → AppsFlyer "af_purchase"）
- 回传时效（最好实时）
- 去重逻辑
```

### 场景 3：SKAdNetwork 数据分析

```
目标：在 iOS ATT 限制下评估广告效果

数据来源：
- SKAN API 获取 Postback 数据
- 结合 Conversion Value 解码规则

分析内容：
- CV 分布（反推用户质量）
- 各渠道 SKAN 安装量与传统归因对比
- Null CV 比例分析

挑战：
- 数据延迟（24-48小时）
- 隐私阈值导致数据缺失
- 需要自定义 CV 模式（SKAN 4.0）
```

### 场景 4：反作弊监控

```
目标：识别并处理虚假流量

数据来源：
- Protect360 报表
- 原始数据中的 blocked_reason 字段

监控指标：
- 被屏蔽安装数量与比例
- 各渠道作弊率
- 作弊类型分布（设备农场、点击注入等）

行动：
- 与媒体协商扣量
- 调整合作策略
```

---

## 🔗 与其他系统的关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                         数据生态系统                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌──────────────┐        ┌──────────────┐        ┌──────────────┐  │
│   │  广告平台     │        │  AppsFlyer   │        │  数据仓库     │  │
│   │              │◀──────▶│              │◀──────▶│              │  │
│   │  Meta        │ Postback│  归因数据    │ Data    │  用户行为    │  │
│   │  Google      │ S2S API │  营销分析    │ Locker  │  付费数据    │  │
│   │  TikTok      │        │  受众管理    │ S2S API │  LTV计算     │  │
│   └──────────────┘        └──────────────┘        └──────────────┘  │
│          │                       │                       │          │
│          ▼                       ▼                       ▼          │
│   ┌──────────────┐        ┌──────────────┐        ┌──────────────┐  │
│   │  投放优化     │        │  Dashboard   │        │  BI 报表     │  │
│   │  ROAS优化    │        │  AppsFlyer   │        │  自定义分析   │  │
│   │  预算分配    │        │  内置报表    │        │  归因模型    │  │
│   └──────────────┘        └──────────────┘        └──────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📋 快速入门清单

### 对于新接入 AppsFlyer 的团队

- [ ] 完成 SDK 集成并验证安装追踪
- [ ] 配置核心事件（注册、首充、付费等）
- [ ] 设置 Deep Link（OneLink）
- [ ] 配置广告平台集成（Meta、Google 等）
- [ ] 理解归因窗口并按业务需求配置
- [ ] 配置 SKAdNetwork Conversion Value

### 对于数据分析团队

- [ ] 开通 Data Locker 或 Pull API 权限
- [ ] 设计数据同步方案（ETL 频率、字段选择）
- [ ] 建立核心指标看板（Installs、CPI、ROAS、Retention）
- [ ] 配置 S2S 事件回传（后端付费数据）
- [ ] 设计多归因窗口对比分析

### 对于投放优化团队

- [ ] 熟悉 AppsFlyer Dashboard 核心报表
- [ ] 理解 SKAN 数据的局限与解读方式
- [ ] 建立渠道 ROAS 对比分析流程
- [ ] 配置异常监控（花费异常、作弊流量等）

---

## 📚 官方资源

- [AppsFlyer 官方文档](https://support.appsflyer.com/)
- [AppsFlyer Dev Hub（API 文档）](https://dev.appsflyer.com/)
- [AppsFlyer Help Center](https://support.appsflyer.com/hc/)
- [AppsFlyer Blog](https://www.appsflyer.com/blog/)

---

## 📂 目录文件列表

| 文件 | 说明 |
|------|------|
| `README.md` | 本文档，总体概述与索引 |
| `APPSFLYER_API_CAPABILITIES.md` | API 能力详细分析 |
| `APPSFLYER_INTEGRATION_GUIDE.md` | SDK 与系统集成指南 |
| `APPSFLYER_DATA_ANALYSIS_GUIDE.md` | 数据分析与优化指南 |
| `APPSFLYER_POSTBACK_DATA_GUIDE.md` | 回传数据结构与分析指南 🆕 |

---

*本文档持续更新中，如有问题或建议请提出。*

