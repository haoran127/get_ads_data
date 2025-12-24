# 📊 AppsFlyer 数据分析与优化指南

> 基于 AppsFlyer 数据进行移动广告分析和优化的实战指南

---

## 📋 目录

1. [数据分析框架](#数据分析框架)
2. [核心指标体系](#核心指标体系)
3. [LTV 与 ROAS 分析](#ltv-与-roas-分析)
4. [渠道评估与优化](#渠道评估与优化)
5. [Cohort 分析实战](#cohort-分析实战)
6. [SKAdNetwork 数据分析](#skadnetwork-数据分析)
7. [反作弊数据分析](#反作弊数据分析)
8. [数据仓库集成方案](#数据仓库集成方案)

---

## 数据分析框架

### 移动广告数据分析全景

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      移动广告数据分析框架                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                     第一层：规模与效率指标                            │   │
│   │                                                                     │   │
│   │   Installs │ Spend │ CPI │ IPM │ CTR │ CVR                          │   │
│   │   (做大)     (成本)   (效率)                                          │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                     第二层：用户质量指标                              │   │
│   │                                                                     │   │
│   │   Retention(D1/D7/D30) │ Session │ Events │ Registration Rate       │   │
│   │   (留存)                 (活跃)    (行为)    (转化)                    │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                     第三层：价值指标                                  │   │
│   │                                                                     │   │
│   │   Revenue │ ARPU │ ARPPU │ LTV │ ROAS │ ROI                         │   │
│   │   (收入)    (人均) (付费人均) (终身价值) (回报率)                       │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                     第四层：健康度指标                                │   │
│   │                                                                     │   │
│   │   Fraud Rate │ Valid Install Rate │ Organic Ratio │ Incremental    │   │
│   │   (作弊率)     (有效安装率)          (自然量占比)    (增量)            │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 分析维度矩阵

| 维度 | 粒度选项 | 典型分析场景 |
|-----|---------|------------|
| **时间** | 日/周/月/Cohort | 趋势分析、周期性分析 |
| **渠道** | 媒体/Campaign/Adset/Ad | 渠道对比、预算分配 |
| **地区** | 国家/地区 | 市场评估、区域策略 |
| **平台** | iOS/Android | 平台差异分析 |
| **设备** | 设备型号/OS版本 | 技术兼容性、用户画像 |
| **素材** | 创意/素材组合 | 素材效果评估 |
| **事件** | 事件名/事件序列 | 漏斗分析、行为分析 |

---

## 核心指标体系

### 指标定义

#### 规模指标

| 指标 | 公式 | 说明 |
|-----|------|------|
| **Installs** | 新增安装数 | 衡量获客规模 |
| **Re-Attribution** | 老用户回归数 | 再营销效果 |
| **Re-Engagement** | 老用户再激活数 | 召回效果 |

#### 效率指标

| 指标 | 公式 | 说明 |
|-----|------|------|
| **CPI** | Spend / Installs | 单次安装成本 |
| **CPM** | (Spend / Impressions) × 1000 | 千次展示成本 |
| **CTR** | Clicks / Impressions | 点击率 |
| **CVR** | Installs / Clicks | 转化率（点击到安装） |
| **IPM** | (Installs / Impressions) × 1000 | 千次展示安装数 |

#### 用户质量指标

| 指标 | 公式 | 说明 |
|-----|------|------|
| **D1/D7/D30 Retention** | Dn 活跃用户 / D0 安装用户 | 第 N 天留存率 |
| **Registration Rate** | 注册用户 / 安装用户 | 注册转化率 |
| **Purchase Rate** | 付费用户 / 安装用户 | 付费转化率 |
| **First Purchase Rate** | 首充用户 / 安装用户 | 首充率 |

#### 价值指标

| 指标 | 公式 | 说明 |
|-----|------|------|
| **Revenue** | 总收入 | 广告带来的收入 |
| **ARPU** | Revenue / Users | 每用户平均收入 |
| **ARPPU** | Revenue / Paying Users | 每付费用户平均收入 |
| **LTV** | 用户生命周期总价值 | 长期价值 |
| **ROAS** | Revenue / Spend | 广告支出回报率 |
| **ROI** | (Revenue - Spend) / Spend | 投资回报率 |

### 指标看板设计

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           日常监控看板                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   📊 今日概览                                   📈 趋势（近7天）              │
│   ┌─────────────────────────────────────┐    ┌────────────────────────────┐ │
│   │ Spend      │ $50,000 │ ↑ 5%        │    │                            │ │
│   │ Installs   │ 10,000  │ ↑ 8%        │    │    [Spend & Installs 趋势图]│ │
│   │ CPI        │ $5.00   │ ↓ 3%        │    │                            │ │
│   │ D1 ROAS    │ 15%     │ ↑ 2%        │    │                            │ │
│   │ D7 ROAS    │ 45%     │ ↑ 5%        │    │                            │ │
│   └─────────────────────────────────────┘    └────────────────────────────┘ │
│                                                                             │
│   📱 渠道表现 Top 5                                                          │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ 渠道          │ Spend    │ Installs │ CPI   │ D7 ROAS │ 状态       │   │
│   │ Meta          │ $20,000  │ 4,500    │ $4.44 │ 52%     │ 🟢 良好    │   │
│   │ Google        │ $15,000  │ 2,800    │ $5.36 │ 48%     │ 🟢 良好    │   │
│   │ TikTok        │ $8,000   │ 1,800    │ $4.44 │ 35%     │ 🟡 关注    │   │
│   │ Unity Ads     │ $4,000   │ 600      │ $6.67 │ 28%     │ 🔴 预警    │   │
│   │ ironSource    │ $3,000   │ 300      │ $10.00│ 22%     │ 🔴 预警    │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   ⚠️ 异常告警                                                                │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ • Unity Ads Campaign X：CPI 较昨日上升 30%，建议检查                  │   │
│   │ • TikTok D7 ROAS 连续 3 天低于目标，建议优化或减预算                   │   │
│   │ • Meta 某 Adset 作弊率异常升高至 15%                                  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## LTV 与 ROAS 分析

### LTV 计算方法

#### 方法一：简单外推法

```
LTV_D30 预测 = ARPU_D7 × 系数

常用系数参考：
- 游戏：D7 ARPU × 3~5 ≈ D90 LTV
- 电商：D7 ARPU × 2~3 ≈ D90 LTV
- 订阅：首月收入 × 用户留存月数
```

#### 方法二：Cohort 累积法

```python
import pandas as pd
import numpy as np

def calculate_ltv_by_cohort(cohort_data):
    """
    基于 Cohort 数据计算 LTV
    
    cohort_data: DataFrame with columns:
        - cohort_date: 安装日期
        - day: 第几天（0, 1, 3, 7, 14, 30...）
        - users: 用户数
        - revenue: 累积收入
    """
    
    # 计算每天的 ARPU
    cohort_data['arpu'] = cohort_data['revenue'] / cohort_data['users']
    
    # 按 cohort_date 和 day 透视
    ltv_pivot = cohort_data.pivot(
        index='cohort_date', 
        columns='day', 
        values='arpu'
    )
    
    # 计算平均 LTV 曲线
    avg_ltv_curve = ltv_pivot.mean()
    
    return avg_ltv_curve


def predict_ltv(current_day_arpu, current_day, target_day, ltv_curve):
    """
    基于当前 ARPU 预测目标天 LTV
    """
    if current_day >= target_day:
        return current_day_arpu
    
    # 计算增长系数
    growth_ratio = ltv_curve[target_day] / ltv_curve[current_day]
    
    predicted_ltv = current_day_arpu * growth_ratio
    return predicted_ltv
```

### ROAS 分析框架

#### ROAS 计算示例

```python
def calculate_roas(spend, revenue, install_date, analysis_date, lookback_window=7):
    """
    计算 ROAS
    
    参数：
        spend: 花费
        revenue: 收入（需要按归因窗口累积）
        install_date: 安装日期
        analysis_date: 分析日期
        lookback_window: 回看窗口（如 D7 ROAS）
    """
    
    # 检查是否有足够的时间窗口
    days_since_install = (analysis_date - install_date).days
    
    if days_since_install < lookback_window:
        return None  # 数据不成熟
    
    roas = revenue / spend if spend > 0 else 0
    
    return roas


def analyze_roas_by_channel(df):
    """
    按渠道分析 ROAS
    
    df: DataFrame with columns:
        - media_source
        - install_date
        - spend
        - d7_revenue
    """
    
    channel_roas = df.groupby('media_source').agg({
        'spend': 'sum',
        'd7_revenue': 'sum'
    })
    
    channel_roas['d7_roas'] = channel_roas['d7_revenue'] / channel_roas['spend']
    channel_roas = channel_roas.sort_values('d7_roas', ascending=False)
    
    return channel_roas
```

### ROAS 优化策略

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ROAS 优化决策矩阵                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                          ROAS 表现                                          │
│                    高 ◀─────────────▶ 低                                    │
│                                                                             │
│   规   高    │  🟢 扩量          │  🟡 优化素材/受众    │                    │
│   模         │  增加预算，复制   │  测试新方向后扩量    │                    │
│   表         │  成功策略         │                     │                    │
│   现   ──────┼──────────────────┼─────────────────────┤                    │
│        低    │  🟡 保持观察      │  🔴 暂停/大幅调整    │                    │
│              │  小规模测试       │  分析原因后重启      │                    │
│              │  寻找扩量机会     │                     │                    │
│                                                                             │
│   行动指南：                                                                  │
│   • ROAS > 目标 且 规模大 → 加预算 20-30%                                    │
│   • ROAS > 目标 且 规模小 → 分析瓶颈，测试扩量方法                            │
│   • ROAS < 目标 且 规模大 → 优化出价/受众/素材，分步调整                       │
│   • ROAS < 目标 且 规模小 → 暂停，分析后重启或放弃                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 渠道评估与优化

### 渠道综合评分模型

```python
import pandas as pd
import numpy as np

def calculate_channel_score(channel_data, weights=None):
    """
    计算渠道综合评分
    
    channel_data: DataFrame with columns:
        - media_source
        - installs
        - cpi
        - d7_roas
        - d7_retention
        - fraud_rate
    
    weights: dict of metric weights
    """
    
    if weights is None:
        weights = {
            'installs_score': 0.15,      # 规模
            'cpi_score': 0.20,           # 成本效率
            'd7_roas_score': 0.30,       # 价值回报
            'd7_retention_score': 0.20,   # 用户质量
            'fraud_score': 0.15          # 流量健康
        }
    
    df = channel_data.copy()
    
    # 规模评分（越大越好，标准化到 0-100）
    df['installs_score'] = (df['installs'] / df['installs'].max()) * 100
    
    # CPI 评分（越低越好）
    df['cpi_score'] = (1 - (df['cpi'] - df['cpi'].min()) / 
                       (df['cpi'].max() - df['cpi'].min())) * 100
    
    # ROAS 评分（越高越好）
    df['d7_roas_score'] = (df['d7_roas'] / df['d7_roas'].max()) * 100
    
    # 留存评分（越高越好）
    df['d7_retention_score'] = (df['d7_retention'] / df['d7_retention'].max()) * 100
    
    # 反作弊评分（越低越好）
    df['fraud_score'] = (1 - df['fraud_rate']) * 100
    
    # 综合评分
    df['total_score'] = (
        df['installs_score'] * weights['installs_score'] +
        df['cpi_score'] * weights['cpi_score'] +
        df['d7_roas_score'] * weights['d7_roas_score'] +
        df['d7_retention_score'] * weights['d7_retention_score'] +
        df['fraud_score'] * weights['fraud_score']
    )
    
    # 评级
    df['grade'] = pd.cut(
        df['total_score'],
        bins=[0, 40, 60, 80, 100],
        labels=['D', 'C', 'B', 'A']
    )
    
    return df.sort_values('total_score', ascending=False)
```

### 预算分配优化

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        预算分配策略                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   基于 ROAS 的预算分配模型：                                                  │
│                                                                             │
│   1. 设定目标 ROAS（如 80%）                                                 │
│   2. 将渠道分为三类：                                                        │
│                                                                             │
│      A 类（ROAS > 目标 × 1.2）：高回报渠道                                    │
│      → 分配 50% 预算，积极扩量                                               │
│                                                                             │
│      B 类（目标 × 0.8 < ROAS < 目标 × 1.2）：达标渠道                         │
│      → 分配 35% 预算，稳定投放                                               │
│                                                                             │
│      C 类（ROAS < 目标 × 0.8）：待优化渠道                                    │
│      → 分配 15% 预算，测试优化                                               │
│                                                                             │
│   动态调整规则：                                                              │
│   • 每周评估一次，基于 D7 ROAS                                               │
│   • 连续 2 周 C 类 → 暂停投放                                                │
│   • A 类持续表现好 → 逐步增加预算（每次 +10-20%）                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Cohort 分析实战

### Cohort 数据获取

```python
import requests
from datetime import datetime, timedelta

def get_cohort_data(v2_api_token: str, app_id: str, from_date: str, to_date: str, 
                    groupings: list = ['media_source'], 
                    kpis: list = ['users', 'revenue', 'retention'],
                    cohort_days: list = [0, 1, 3, 7, 14, 30]):
    """
    从 AppsFlyer Cohort API 获取数据
    
    ⚠️ 注意：必须使用 V2 API Token（JWT 格式），V1 Token 已于 2023年9月停用
    
    Args:
        v2_api_token: AppsFlyer V2 API Token（从 Dashboard → Settings → API Tokens 获取）
        app_id: 应用 ID
        from_date: 开始日期 (YYYY-MM-DD)
        to_date: 结束日期 (YYYY-MM-DD)
        groupings: 分组维度
        kpis: 指标列表
        cohort_days: Cohort 天数列表
    """
    
    url = f"https://hq1.appsflyer.com/api/cohorts/v1/data/app/{app_id}"
    
    headers = {
        "Authorization": f"Bearer {v2_api_token}",  # V2 Token Bearer 认证
        "Content-Type": "application/json"
    }
    
    payload = {
        "cohort_type": "user_acquisition",
        "min_cohort_size": 1,
        "cohort_day": cohort_days,
        "from": from_date,
        "to": to_date,
        "groupings": groupings,
        "kpis": kpis
    }
    
    response = requests.post(url, headers=headers, json=payload)
    response.raise_for_status()  # 检查 HTTP 错误
    return response.json()
```

### Cohort 留存分析

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

def analyze_retention_cohort(cohort_data):
    """
    分析留存 Cohort
    """
    
    # 转换为 DataFrame
    df = pd.DataFrame(cohort_data['data'])
    
    # 创建留存热力图
    retention_pivot = df.pivot(
        index='cohort_date',
        columns='cohort_day',
        values='retention'
    )
    
    plt.figure(figsize=(12, 8))
    sns.heatmap(
        retention_pivot,
        annot=True,
        fmt='.1%',
        cmap='RdYlGn',
        vmin=0,
        vmax=1
    )
    plt.title('Retention Cohort Analysis')
    plt.xlabel('Days Since Install')
    plt.ylabel('Install Cohort')
    plt.tight_layout()
    plt.savefig('retention_cohort.png')
    
    return retention_pivot


def compare_channel_retention(cohort_data):
    """
    对比不同渠道的留存曲线
    """
    
    df = pd.DataFrame(cohort_data['data'])
    
    # 按渠道和天数聚合
    channel_retention = df.groupby(['media_source', 'cohort_day']).agg({
        'retention': 'mean'
    }).reset_index()
    
    # 绘制留存曲线
    plt.figure(figsize=(10, 6))
    
    for channel in channel_retention['media_source'].unique():
        channel_data = channel_retention[channel_retention['media_source'] == channel]
        plt.plot(
            channel_data['cohort_day'],
            channel_data['retention'],
            marker='o',
            label=channel
        )
    
    plt.xlabel('Days Since Install')
    plt.ylabel('Retention Rate')
    plt.title('Retention by Channel')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.savefig('channel_retention.png')
    
    return channel_retention
```

### LTV Cohort 分析

```python
def analyze_ltv_cohort(cohort_data):
    """
    分析 LTV Cohort
    """
    
    df = pd.DataFrame(cohort_data['data'])
    
    # 计算 ARPU
    df['arpu'] = df['revenue'] / df['users']
    
    # 按渠道对比 LTV 曲线
    ltv_by_channel = df.pivot_table(
        values='arpu',
        index='cohort_day',
        columns='media_source',
        aggfunc='mean'
    )
    
    # 绘制 LTV 曲线
    plt.figure(figsize=(10, 6))
    
    for channel in ltv_by_channel.columns:
        plt.plot(
            ltv_by_channel.index,
            ltv_by_channel[channel],
            marker='o',
            label=channel
        )
    
    plt.xlabel('Days Since Install')
    plt.ylabel('Cumulative ARPU ($)')
    plt.title('LTV by Channel')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.savefig('channel_ltv.png')
    
    return ltv_by_channel
```

---

## SKAdNetwork 数据分析

### SKAN 数据特点

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        SKAN 数据分析要点                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   数据特点：                                                                  │
│   • 延迟：Postback 延迟 24-48 小时                                           │
│   • 聚合：数据按 Campaign 粒度聚合，无用户级明细                                │
│   • 缺失：隐私阈值导致部分数据为 null                                         │
│   • 有限：仅 64 个 CV 值（0-63）编码用户行为                                   │
│                                                                             │
│   分析策略：                                                                  │
│   1. CV 解码：将 CV 值映射回业务指标（收入、事件）                              │
│   2. 比较分析：SKAN 数据 vs 传统归因数据                                       │
│   3. Null CV 处理：分析 Null CV 比例，评估数据质量                             │
│   4. 渠道评估：基于有限数据做渠道 ROI 估算                                      │
│                                                                             │
│   SKAN 4.0 新机会：                                                          │
│   • 多次 Postback（3 次窗口）                                                │
│   • 细粒度 + 粗粒度 CV                                                       │
│   • 更多来源标识符                                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### SKAN 数据分析代码

```python
import pandas as pd
import numpy as np

def decode_conversion_value(cv, cv_mapping):
    """
    解码 Conversion Value
    
    cv_mapping: dict, 如 {
        0: {'revenue': 0, 'event': 'install_only'},
        1: {'revenue': 1, 'event': 'registration'},
        ...
    }
    """
    if cv is None or np.isnan(cv):
        return {'revenue': None, 'event': 'null_cv'}
    
    return cv_mapping.get(int(cv), {'revenue': None, 'event': 'unknown'})


def analyze_skan_data(skan_df, cv_mapping):
    """
    分析 SKAN 数据
    
    skan_df: DataFrame with columns:
        - date
        - media_source
        - campaign
        - installs
        - conversion_value
    """
    
    # 解码 CV
    skan_df['decoded'] = skan_df['conversion_value'].apply(
        lambda cv: decode_conversion_value(cv, cv_mapping)
    )
    skan_df['decoded_revenue'] = skan_df['decoded'].apply(lambda x: x['revenue'])
    skan_df['decoded_event'] = skan_df['decoded'].apply(lambda x: x['event'])
    
    # 计算 Null CV 比例
    null_cv_rate = (skan_df['conversion_value'].isna().sum() / 
                   len(skan_df))
    print(f"Null CV Rate: {null_cv_rate:.1%}")
    
    # 按渠道汇总
    channel_summary = skan_df.groupby('media_source').agg({
        'installs': 'sum',
        'decoded_revenue': 'sum'
    })
    channel_summary['arpu'] = (channel_summary['decoded_revenue'] / 
                               channel_summary['installs'])
    
    return channel_summary


def compare_skan_vs_traditional(skan_data, traditional_data):
    """
    对比 SKAN 数据与传统归因数据
    """
    
    comparison = pd.merge(
        skan_data[['media_source', 'installs', 'decoded_revenue']].rename(
            columns={'installs': 'skan_installs', 'decoded_revenue': 'skan_revenue'}
        ),
        traditional_data[['media_source', 'installs', 'revenue']].rename(
            columns={'installs': 'trad_installs', 'revenue': 'trad_revenue'}
        ),
        on='media_source',
        how='outer'
    )
    
    # 计算差异
    comparison['install_diff'] = (
        (comparison['skan_installs'] - comparison['trad_installs']) / 
        comparison['trad_installs']
    )
    comparison['revenue_diff'] = (
        (comparison['skan_revenue'] - comparison['trad_revenue']) / 
        comparison['trad_revenue']
    )
    
    return comparison
```

---

## 反作弊数据分析

### 作弊类型识别

| 作弊类型 | 表现特征 | 检测方法 |
|---------|---------|---------|
| **设备农场** | 大量新设备、相同 IP 段 | 设备 ID 聚类、IP 分析 |
| **点击注入** | 点击到安装时间极短 | CTIT 分布分析 |
| **点击欺诈** | 异常高点击量、低 CVR | 点击量异常检测 |
| **SDK 欺骗** | 请求模式异常 | 请求签名验证 |
| **安装劫持** | 最后一跳异常 | 归因路径分析 |

### 反作弊监控代码

```python
import pandas as pd
import numpy as np

def analyze_fraud_data(installs_df):
    """
    分析作弊数据
    
    installs_df: DataFrame with columns:
        - appsflyer_id
        - media_source
        - install_time
        - click_time
        - is_blocked
        - blocked_reason
        - blocked_sub_reason
    """
    
    # 计算各渠道作弊率
    fraud_by_channel = installs_df.groupby('media_source').agg({
        'appsflyer_id': 'count',
        'is_blocked': 'sum'
    }).rename(columns={
        'appsflyer_id': 'total_installs',
        'is_blocked': 'blocked_installs'
    })
    
    fraud_by_channel['fraud_rate'] = (
        fraud_by_channel['blocked_installs'] / 
        fraud_by_channel['total_installs']
    )
    
    # 作弊类型分布
    fraud_types = installs_df[installs_df['is_blocked'] == True].groupby(
        'blocked_reason'
    ).size().sort_values(ascending=False)
    
    return fraud_by_channel, fraud_types


def analyze_ctit(installs_df):
    """
    分析 Click-to-Install Time (CTIT) 分布
    检测点击注入等异常
    """
    
    df = installs_df.copy()
    df['ctit_seconds'] = (
        pd.to_datetime(df['install_time']) - 
        pd.to_datetime(df['click_time'])
    ).dt.total_seconds()
    
    # 按渠道统计 CTIT 分布
    ctit_stats = df.groupby('media_source').agg({
        'ctit_seconds': ['mean', 'median', 'std', 'min']
    })
    
    # 标记异常（CTIT < 10秒可能是点击注入）
    ctit_stats['suspicious_ratio'] = df.groupby('media_source').apply(
        lambda x: (x['ctit_seconds'] < 10).sum() / len(x)
    )
    
    return ctit_stats
```

---

## 数据仓库集成方案

### 推荐架构

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      数据仓库集成架构                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────┐                                                          │
│   │  AppsFlyer  │                                                          │
│   │  Data Locker│ ──────▶ S3/GCS Bucket ──────▶ ┌────────────────┐         │
│   └─────────────┘         (Parquet)              │  数据仓库       │         │
│                                                  │  (Snowflake/   │         │
│   ┌─────────────┐                               │   BigQuery)    │         │
│   │  业务数据库  │ ──────▶ ETL Pipeline ────────▶│                │         │
│   │  (订单/用户) │                               └───────┬────────┘         │
│   └─────────────┘                                       │                  │
│                                                         │                  │
│                                                         ▼                  │
│                                                  ┌────────────────┐         │
│                                                  │  BI 报表系统   │         │
│                                                  │  (Tableau/     │         │
│                                                  │   Looker)      │         │
│                                                  └────────────────┘         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 核心表设计

```sql
-- 安装明细表
CREATE TABLE fact_installs (
    install_id VARCHAR(64) PRIMARY KEY,
    appsflyer_id VARCHAR(64),
    customer_user_id VARCHAR(64),
    install_time TIMESTAMP,
    media_source VARCHAR(128),
    campaign VARCHAR(256),
    adset VARCHAR(256),
    ad VARCHAR(256),
    country_code VARCHAR(8),
    platform VARCHAR(16),
    device_type VARCHAR(64),
    cost DECIMAL(10,4),
    is_organic BOOLEAN,
    is_blocked BOOLEAN,
    blocked_reason VARCHAR(64)
);

-- 事件明细表
CREATE TABLE fact_events (
    event_id VARCHAR(64) PRIMARY KEY,
    appsflyer_id VARCHAR(64),
    customer_user_id VARCHAR(64),
    event_time TIMESTAMP,
    event_name VARCHAR(128),
    event_value JSON,
    event_revenue DECIMAL(10,4),
    event_currency VARCHAR(8),
    install_id VARCHAR(64) REFERENCES fact_installs(install_id)
);

-- 日度汇总表
CREATE TABLE agg_daily_performance (
    date DATE,
    media_source VARCHAR(128),
    campaign VARCHAR(256),
    country_code VARCHAR(8),
    platform VARCHAR(16),
    impressions INTEGER,
    clicks INTEGER,
    installs INTEGER,
    cost DECIMAL(12,4),
    d0_revenue DECIMAL(12,4),
    d1_revenue DECIMAL(12,4),
    d7_revenue DECIMAL(12,4),
    d30_revenue DECIMAL(12,4),
    d1_retention_rate DECIMAL(5,4),
    d7_retention_rate DECIMAL(5,4),
    PRIMARY KEY (date, media_source, campaign, country_code, platform)
);
```

### ETL 示例

```python
import pandas as pd
from datetime import datetime, timedelta

def load_data_locker_to_warehouse(bucket_path, date, data_type='installs'):
    """
    从 Data Locker 加载数据到数据仓库
    """
    
    # 读取 Parquet 文件
    file_path = f"{bucket_path}/{data_type}/dt={date}/*.parquet"
    df = pd.read_parquet(file_path)
    
    # 数据清洗
    df = clean_appsflyer_data(df)
    
    # 写入数据仓库
    df.to_sql(
        f'fact_{data_type}',
        engine,
        if_exists='append',
        index=False
    )
    
    return len(df)


def build_daily_aggregation(date):
    """
    构建日度汇总数据
    """
    
    query = f"""
    INSERT INTO agg_daily_performance
    SELECT 
        DATE(install_time) as date,
        media_source,
        campaign,
        country_code,
        platform,
        COUNT(DISTINCT install_id) as installs,
        SUM(cost) as cost,
        SUM(CASE WHEN DATEDIFF(day, install_time, event_time) <= 0 
            THEN event_revenue ELSE 0 END) as d0_revenue,
        SUM(CASE WHEN DATEDIFF(day, install_time, event_time) <= 1 
            THEN event_revenue ELSE 0 END) as d1_revenue,
        SUM(CASE WHEN DATEDIFF(day, install_time, event_time) <= 7 
            THEN event_revenue ELSE 0 END) as d7_revenue
    FROM fact_installs i
    LEFT JOIN fact_events e ON i.appsflyer_id = e.appsflyer_id
    WHERE DATE(install_time) = '{date}'
    GROUP BY 1, 2, 3, 4, 5
    """
    
    execute_query(query)
```

---

## 📚 参考资源

- [AppsFlyer Help Center](https://support.appsflyer.com/)
- [AppsFlyer API Documentation](https://dev.appsflyer.com/)
- [SKAdNetwork Documentation](https://support.appsflyer.com/hc/en-us/articles/360011451778)
- [Protect360 Documentation](https://support.appsflyer.com/hc/en-us/articles/360000429997)

---

## ⚠️ 注意事项

- 所有 API 调用必须使用 **V2 Token**（JWT Bearer 认证），V1 Token 已于 2023年9月停用
- 数据分析代码中的 API 调用请参考 `APPSFLYER_API_CAPABILITIES.md` 中的最新示例

---

*文档版本：v1.1 | 最后更新：2025-01*

