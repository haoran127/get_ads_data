# Google Ads 周报模板

> **使用说明**：本模板用于生成规范的 Google Ads 周报，确保时间术语准确、数据一致。

---

## 🗄️ 数据架构设计

### 数据流向

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│  Google Ads API │ ──▶  │    数据库       │ ──▶  │   周报生成      │
│  (每日增量拉取)  │      │  (持久化存储)   │      │  (读取DB)       │
└─────────────────┘      └─────────────────┘      └─────────────────┘
        ↓                        ↓                        ↓
   只拉增量/变化数据        存储全年+历史数据         支持任意时间范围查询
```

### 核心原则

| 原则 | 说明 |
|------|------|
| **一次拉取，永久存储** | API 数据拉取成功后立即入库，不重复拉取已有数据 |
| **增量更新** | 每日只拉取前一天的数据（或变化的数据） |
| **历史可追溯** | 数据库保留全年（52周+）甚至多年历史数据 |
| **报表从DB读取** | 周报生成时查询数据库，不直接调用 API |

### 推荐表结构

```sql
-- 周度汇总表（用于周报）
CREATE TABLE ads_weekly_summary (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    
    -- 时间维度
    year INT NOT NULL,                    -- 年份：2025
    week_number INT NOT NULL,             -- 周数：1-52
    week_start DATE NOT NULL,             -- 周一日期：2025-12-08
    week_end DATE NOT NULL,               -- 周日日期：2025-12-14
    
    -- 层级维度
    account_id VARCHAR(50) NOT NULL,      -- 账号ID
    account_name VARCHAR(100),            -- 账号名称
    campaign_id VARCHAR(50),              -- Campaign ID（可为空=账号汇总）
    campaign_name VARCHAR(200),           -- Campaign 名称
    
    -- 核心指标
    cost DECIMAL(15,2) NOT NULL,          -- 花费
    impressions BIGINT NOT NULL,          -- 展示
    clicks BIGINT NOT NULL,               -- 点击
    conversions BIGINT NOT NULL,          -- 转化（安装）
    conversion_value DECIMAL(15,2),       -- 转化价值
    
    -- 计算指标（冗余存储，提升查询性能）
    cpi DECIMAL(10,4),                    -- Cost Per Install
    ctr DECIMAL(8,4),                     -- Click Through Rate
    cvr DECIMAL(8,4),                     -- Conversion Rate
    roas DECIMAL(10,4),                   -- Return On Ad Spend
    
    -- 元数据
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- 索引
    UNIQUE KEY uk_week_account_campaign (year, week_number, account_id, campaign_id),
    INDEX idx_week (year, week_number),
    INDEX idx_account (account_id),
    INDEX idx_campaign (campaign_id)
);

-- 日度明细表（用于更细粒度分析）
CREATE TABLE ads_daily_detail (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    date DATE NOT NULL,
    account_id VARCHAR(50) NOT NULL,
    campaign_id VARCHAR(50) NOT NULL,
    -- ... 同上指标字段 ...
    
    UNIQUE KEY uk_date_campaign (date, account_id, campaign_id),
    INDEX idx_date (date)
);
```

### 数据同步流程

```python
# 每日定时任务（建议凌晨执行）
def daily_sync_job():
    """
    每日增量同步 Google Ads 数据
    """
    yesterday = date.today() - timedelta(days=1)
    
    # 1. 检查是否已同步
    if is_date_synced(yesterday):
        log.info(f"{yesterday} 数据已存在，跳过")
        return
    
    # 2. 从 API 拉取昨日数据
    data = fetch_from_google_ads_api(
        date_range=(yesterday, yesterday)
    )
    
    # 3. 写入日度明细表
    insert_daily_detail(data)
    
    # 4. 如果是周日，汇总本周数据到周度表
    if yesterday.weekday() == 6:  # Sunday
        week_start = yesterday - timedelta(days=6)
        aggregate_weekly_summary(week_start, yesterday)
    
    log.info(f"{yesterday} 数据同步完成")
```

---

## 📋 模板结构

```
┌─────────────────────────────────────────────────────────────┐
│  1. 报告头部（时间定义 + 核心指标卡片）                        │
├─────────────────────────────────────────────────────────────┤
│  2. 历史周趋势（近4-8周，含上周vs前周对比）                    │
├─────────────────────────────────────────────────────────────┤
│  3. Campaign TOP 排行                                        │
├─────────────────────────────────────────────────────────────┤
│  4. 变化最大的 Campaign                                      │
├─────────────────────────────────────────────────────────────┤
│  5. 月度累计                                                 │
├─────────────────────────────────────────────────────────────┤
│  6. 洞察与行动建议                                           │
└─────────────────────────────────────────────────────────────┘
```

> 💡 **设计原则**：历史周趋势表已包含上周与前周的对比数据，无需单独设置"上周 vs 前周"模块，避免信息冗余。

---

## 🕐 时间术语定义（关键！）

假设**报告生成日期**为 `2025-12-15（周一）`：

| 术语 | 定义 | 日期范围 |
|------|------|---------|
| **上周** | 报告生成日的上一个完整周（周一~周日） | 2025/12/08 - 12/14 |
| **前周** | 上周的前一周 | 2025/12/01 - 12/07 |
| **本周** | 报告生成日所在周（通常数据不完整，慎用） | 2025/12/15 - 12/21 |

> ⚠️ **注意**：周报统计的是**已结束的完整周**，所以主体数据应该是"上周"，对比的是"前周"。"本周"仅在需要展示实时进度时使用。

---

## 📝 完整模板示例

```markdown
# 📊 Google Ads 周报

**报告生成时间**：2025-12-15 09:00（周一）  
**统计周期**：2025/12/08（周一）- 12/14（周日）  
**数据来源**：Google Ads API  

---

## 📅 历史周趋势

> **数据来源**：从数据库 `ads_weekly_summary` 表读取，支持全年 52 周历史数据。  
> **展示策略**：周报默认显示近 8 周，可按需扩展查看全年数据。

### 近 8 周趋势（周报默认视图）

| # | 周 | 日期范围 | 花费 | 转化 | CPI | CTR | CVR | 花费环比 |
|---|---|---------|------|------|-----|-----|-----|---------|
| 1 | **上周** | 12/08-12/14 | **$65,568** | **15,191** | **$4.32** | 3.43% | 12.66% | 🔴 -10.4% |
| 2 | 前周 | 12/01-12/07 | $73,205 | 13,787 | $5.31 | 3.59% | 11.99% | +8.2% |
| 3 | W49 | 11/24-11/30 | $67,650 | 12,500 | $5.41 | 3.28% | 11.75% | -3.1% |
| 4 | W48 | 11/17-11/23 | $69,800 | 13,200 | $5.29 | 3.35% | 12.02% | +5.5% |
| 5 | W47 | 11/10-11/16 | $66,150 | 12,800 | $5.17 | 3.22% | 11.88% | +2.3% |
| 6 | W46 | 11/03-11/09 | $64,670 | 12,100 | $5.34 | 3.18% | 11.52% | -1.8% |
| 7 | W45 | 10/27-11/02 | $65,850 | 11,900 | $5.53 | 3.15% | 11.35% | +4.2% |
| 8 | W44 | 10/20-10/26 | $63,200 | 11,500 | $5.50 | 3.10% | 11.20% | - |

> **趋势图例**：🟢 正向变化 | 🔴 负向变化 | 第一行加粗为本期核心数据

### 全年趋势查询（从数据库读取）

```sql
-- 查询全年52周数据
SELECT 
    week_number,
    week_start,
    week_end,
    SUM(cost) as total_cost,
    SUM(conversions) as total_conversions,
    SUM(cost) / NULLIF(SUM(conversions), 0) as cpi,
    SUM(clicks) / NULLIF(SUM(impressions), 0) * 100 as ctr,
    SUM(conversions) / NULLIF(SUM(clicks), 0) * 100 as cvr,
    -- 环比计算
    LAG(SUM(cost)) OVER (ORDER BY week_number) as prev_week_cost,
    (SUM(cost) - LAG(SUM(cost)) OVER (ORDER BY week_number)) 
        / NULLIF(LAG(SUM(cost)) OVER (ORDER BY week_number), 0) * 100 as cost_wow
FROM ads_weekly_summary
WHERE year = 2025
GROUP BY week_number, week_start, week_end
ORDER BY week_number DESC;
```

### 全年趋势可视化（示例）

```
2025 年度花费趋势（52周）
$80k |                                    ▄▄
$70k |              ▄▄▄▄    ▄▄▄▄▄▄▄▄▄▄████████▄▄
$60k |    ▄▄▄▄▄▄████████████████████████████████████▄▄
$50k |████████████████████████████████████████████████
$40k |████████████████████████████████████████████████
     └─W1──W5──W10─W15─W20─W25─W30─W35─W40─W45─W50─W52
       Jan  Feb  Mar  Apr  May  Jun  Jul  Aug  Sep  Oct  Nov  Dec
```

### 同比分析（YoY）

数据库支持多年数据后，可进行同比分析：

```sql
-- 本周 vs 去年同期
SELECT 
    a.week_number,
    a.cost as this_year_cost,
    b.cost as last_year_cost,
    (a.cost - b.cost) / NULLIF(b.cost, 0) * 100 as yoy_change
FROM ads_weekly_summary a
LEFT JOIN ads_weekly_summary b 
    ON a.week_number = b.week_number 
    AND a.year = b.year + 1
WHERE a.year = 2025;
```

**📊 趋势图**：
```
花费趋势（近8周）
$75k |        ██
$70k |  ██    ██  ██
$65k |  ██ ██ ██  ██ ██ ██ ██
$60k |  ██ ██ ██  ██ ██ ██ ██ ██
     └─W44─W45─W46─W47─W48─W49─前周─上周─
```

---

## 🏆 Campaign 花费 TOP 10

| # | 账号 | Campaign | 花费 | 转化 | CPI | 占比 | vs前周 |
|---|------|----------|------|------|-----|------|--------|
| 1 | SHY | ROE_AD_day14_3.0_WW_roas | $18,602 | 4,392 | $4.24 | 28.4% | 🔴 -32% |
| 2 | TQ | LS_And_WW_20250821_3.0 | $14,900 | 5,369 | $2.78 | 22.7% | 🟡 -2% |
| 3 | SHY | ROE_AD_day7_3.0_US_roas | $13,873 | 519 | $26.73 | 21.2% | 🔴 -38% |
| 4 | SHY | ROE_AD_day7_3.0_WW_GGM | $10,230 | 3,466 | $2.95 | 15.6% | 🟢 +12% |
| 5 | TQ | LS_And_WW_20251205_3.0 | $7,963 | 1,445 | $5.51 | 12.1% | 🟢 +956% |
| | | **其他** | $0 | 0 | - | 0% | - |
| | | **合计** | **$65,568** | **15,191** | **$4.32** | **100%** | - |

---

## 📈 变化最大的 Campaign

### 🚀 花费增长 TOP 5

| # | 账号 | Campaign | 前周 | 上周 | 变化 | 原因分析 |
|---|------|----------|------|------|------|---------|
| 1 | TQ | LS_And_WW_20251205_3.0 | $754 | $7,963 | 🟢 +956% | 新Campaign放量 |
| 2 | SHY | ROE_AD_day7_3.0_WW_GGM | $9,134 | $10,230 | 🟢 +12% | 预算上调 |

### 📉 花费下降 TOP 5

| # | 账号 | Campaign | 前周 | 上周 | 变化 | 原因分析 |
|---|------|----------|------|------|------|---------|
| 1 | SHY | ROE_AD_day7_3.0_US_roas | $22,210 | $13,873 | 🔴 -38% | CPI过高主动收量 |
| 2 | SHY | ROE_AD_day14_3.0_WW_roas | $27,187 | $18,602 | 🔴 -32% | 学习期波动 |

---

## 📆 月度累计（MTD）

| 月份 | 花费 | 转化 | CPI | CTR | CVR | 完成进度 |
|------|------|------|-----|-----|-----|---------|
| 2025-12（截至12/14） | $138,773 | 28,978 | $4.79 | 3.35% | 12.32% | 45%（预估月底$308k） |
| 2025-11（完整月） | $275,120 | 52,300 | $5.26 | 3.28% | 11.89% | 100% |
| 2025-10（完整月） | $258,650 | 47,800 | $5.41 | 3.15% | 11.52% | 100% |

---

## 💡 洞察与行动建议

### ✅ 正向信号
- **CPI 大幅下降 -18.7%**：投放效率持续改善，单位获客成本降低
- **转化数增长 +10.2%**：在花费减少的情况下转化反增，ROI 提升明显
- **LS_And_WW_20251205 表现优异**：新 Campaign CPI $5.51，有放量空间

### ⚠️ 关注事项
- **ROE_AD_day7_3.0_US_roas CPI=$26.73**：严重偏高，需评估是否继续投放
- **总花费环比下降 -10.4%**：需确认是主动优化还是被动受限

### 📋 本周行动计划

| 优先级 | 行动项 | 负责人 | 截止日期 |
|--------|--------|--------|---------|
| P0 | 评估 US_roas Campaign 是否暂停或调整策略 | @UA | 12/16 |
| P1 | LS_And_WW_20251205 预算提升测试（+30%） | @UA | 12/17 |
| P1 | 分析 WW_GGM 成功因素，考虑复制策略 | @UA | 12/18 |
| P2 | 补充新素材 2-3 组，覆盖高潜力 Campaign | @Creative | 12/19 |

---

## 📎 附录

### A. 数据口径说明
- **花费**：Google Ads 后台消耗，已扣除无效点击
- **转化**：App 安装数（first_open 事件）
- **CPI**：Cost Per Install = 花费 / 转化数
- **统计时区**：UTC+8

### B. Campaign 命名规则
```
{Game}_{Platform}_{Window}_{Version}_{Geo}_{Strategy}
示例：ROE_AD_day14_3.0_WW_roas
```

### C. 趋势判定标准
| 变化幅度 | 标记 |
|---------|------|
| > +5% | 🟢 正向 |
| -5% ~ +5% | 🟡 持平 |
| < -5% | 🔴 负向 |
```

---

## 🔧 模板使用指南

### 1. 时间参数配置

```python
# 伪代码示例
from datetime import datetime, timedelta

def get_report_periods(report_date):
    """
    根据报告生成日期计算各时间段
    report_date: 报告生成日（通常是周一）
    """
    # 上周：报告日往前推到上一个周日，再往前6天到周一
    last_sunday = report_date - timedelta(days=report_date.weekday() + 1)
    last_monday = last_sunday - timedelta(days=6)
    
    # 前周
    prev_sunday = last_monday - timedelta(days=1)
    prev_monday = prev_sunday - timedelta(days=6)
    
    return {
        "上周": (last_monday, last_sunday),      # 统计主体
        "前周": (prev_monday, prev_sunday),      # 对比基准
        "本周": (report_date, None),             # 进行中（可选）
    }

# 示例：2025-12-15（周一）生成报告
periods = get_report_periods(datetime(2025, 12, 15))
# 输出：
# 上周: 2025-12-08 ~ 2025-12-14
# 前周: 2025-12-01 ~ 2025-12-07
```

### 2. 数据一致性检查

```python
def validate_report_data(summary_spend, campaign_spend_total, trend_spend):
    """
    确保各模块数据一致
    """
    assert summary_spend == campaign_spend_total, "汇总花费与Campaign合计不一致"
    assert summary_spend == trend_spend, "汇总花费与趋势表上周数据不一致"
    return True
```

### 3. 自动生成 Checklist

报告生成后自动检查：

- [ ] 时间术语是否准确（上周/前周定义）
- [ ] 汇总数据与明细合计是否一致
- [ ] 历史趋势是否包含足够周数（建议 ≥4 周）
- [ ] 环比计算是否正确
- [ ] 行动建议是否可执行且有负责人

---

## 📊 可视化建议

### 推荐图表

| 模块 | 图表类型 | 说明 |
|------|---------|------|
| 核心指标 | KPI 卡片 | 大数字 + 环比箭头 |
| 历史趋势 | 折线图 / 面积图 | X轴=周，Y轴=花费/转化 |
| Campaign TOP | 横向柱状图 | 按花费排序 |
| 花费分布 | 饼图 / 环形图 | 各 Campaign 占比 |
| 变化最大 | 瀑布图 | 增减贡献拆解 |

---

## 📌 最佳实践

1. **固定生成时间**：建议每周一上午固定时间生成，确保数据完整
2. **保留历史版本**：周报存档便于回溯对比
3. **自动化校验**：生成后自动检查数据一致性
4. **行动闭环**：下周报告回顾上周行动项完成情况
5. **异常高亮**：对超出阈值的指标自动标红/标绿

---

**模板版本**：v1.0  
**最后更新**：2025-12-15

