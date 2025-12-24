# 📤 AppsFlyer 回传数据结构与分析指南

> 基于真实回传数据的解析与应用指南

---

## 📋 目录

1. [回传数据概述](#回传数据概述)
2. [数据结构解析](#数据结构解析)
3. [关键字段说明](#关键字段说明)
4. [数据分析应用](#数据分析应用)
5. [数据表设计](#数据表设计)
6. [分析 SQL 示例](#分析-sql-示例)
7. [注意事项](#注意事项)

---

## 回传数据概述

### 什么是回传数据？

回传数据（Postback Data）是 AppsFlyer 通过 Push API 实时推送到广告主服务器的事件数据，包含：
- 用户行为事件（安装、注册、付费等）
- 完整的归因信息（媒体来源、Campaign、广告组等）
- 设备和用户标识

### 数据流向

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         回传数据流向                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   用户行为              AppsFlyer                  广告主服务器              │
│                                                                             │
│   ┌─────────┐          ┌─────────┐              ┌─────────────┐            │
│   │ 安装    │          │         │   Postback   │             │            │
│   │ 注册    │ ───────▶ │  归因   │ ───────────▶ │   Kafka     │            │
│   │ 付费    │   SDK    │  处理   │   (实时)     │   数据湖    │            │
│   │ 事件    │          │         │              │             │            │
│   └─────────┘          └─────────┘              └──────┬──────┘            │
│                                                        │                    │
│                                                        ▼                    │
│                                                 ┌─────────────┐            │
│                                                 │  数据仓库    │            │
│                                                 │  BI 分析    │            │
│                                                 └─────────────┘            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 数据结构解析

### 完整数据示例

```json
{
  "channel": "market_global",
  "event_type": "OStep2",
  "kafka_partition": "3",
  "kafka_offset": "59596",
  "platform": "Android",
  "country": "",
  "server_time": "1766303307273",
  "device_id": "emula_c6efa72d1e1b4950aae00533e7f26f22FB88",
  "kafka_topic": "ls_uba_events",
  "kafka_timestamp": "1766303307",
  "event_data": "{\"af_customer_user_id\":\"412487996001444\"}",
  "user_id": "412487996001444",
  "app_version": "25.1203.2",
  "appsflyer_data": "{...归因数据...}",
  "kafka_vpcId": "vpc-rj9vp1h8c2et2w95txshe",
  "ip": "37.188.180.216",
  "project_id": "ls",
  "timestamp": "1766296105267",
  "key": "412487996001444"
}
```

### appsflyer_data 字段解析

```json
{
  "ad_event_id": "CltDandLQ0FqdzhyVzJCaEFnRWl3QW9STzVyT0pkVlA3U2w5...",
  "af_ad": "",
  "af_ad_id": "",
  "af_ad_type": "ClickToDownload",
  "af_adset": "20240723图文组",
  "af_adset_id": "168145036514",
  "af_c_id": "21421952087",
  "af_channel": "ACI_Search",
  "af_click_lookback": "30d",
  "af_reengagement_window": "30d",
  "af_siteid": "GoogleSearch",
  "af_status": "Non-organic",
  "af_viewthrough_lookback": "1d",
  "campaign": "TQ|LS_And_T2T3_20240628_3.0_bid5.5%_数值跑酷",
  "campaign_id": "21421952087",
  "click-timestamp": "1724790698289",
  "click_time": "2024-08-27 20:31:38.289",
  "cost_cents_USD": "0",
  "external_account_id": 1754904690,
  "install_time": "2024-08-27 20:35:03.715",
  "is_first_launch": false,
  "iscache": true,
  "lat": "0",
  "match_type": "srn",
  "media_source": "googleadwords_int",
  "network": "Search",
  "orig_cost": "0.0",
  "referrer_gclid": "CjwKCAjw8rW2BhAgEiwAoRO5rOJdVP7Sl9njO3OMx_SR_09Fi4nk6FTLmVUJ21ettwpZpEjR8PcZoxoCg00QAvD_BwE",
  "retargeting_conversion_type": "none"
}
```

---

## 关键字段说明

### 事件层字段

| 字段 | 类型 | 说明 | 示例 |
|-----|------|------|------|
| `event_type` | String | 事件类型（漏斗步骤） | `OStep2`, `FirstPay`, `Install` |
| `user_id` | String | 用户唯一标识 | `412487996001444` |
| `device_id` | String | 设备唯一标识 | `emula_c6efa72d...` |
| `platform` | String | 平台 | `Android`, `iOS` |
| `country` | String | 国家代码 | `US`, `CN`（可能为空） |
| `timestamp` | String | 事件时间戳（毫秒） | `1766296105267` |
| `server_time` | String | 服务器接收时间（毫秒） | `1766303307273` |
| `ip` | String | 用户 IP 地址 | `37.188.180.216` |
| `app_version` | String | App 版本号 | `25.1203.2` |
| `channel` | String | 业务渠道 | `market_global` |
| `event_data` | JSON String | 事件附加数据 | `{"af_customer_user_id":"..."}` |

### AppsFlyer 归因字段

| 字段 | 类型 | 说明 | 分析用途 |
|-----|------|------|---------|
| **`media_source`** | String | 媒体来源 | 渠道归因、ROAS 分析 |
| **`campaign`** | String | 广告系列名称 | Campaign 效果分析 |
| **`campaign_id`** | String | 广告系列 ID | 与广告平台数据关联 |
| **`af_adset`** | String | 广告组名称 | 广告组效果分析 |
| **`af_adset_id`** | String | 广告组 ID | 与广告平台数据关联 |
| **`af_ad`** | String | 广告名称 | 素材效果分析 |
| **`af_ad_id`** | String | 广告 ID | 素材关联 |
| **`af_channel`** | String | 渠道类型 | `ACI_Search`, `ACI_Display` |
| **`network`** | String | 网络类型 | `Search`, `Display`, `Video` |
| **`af_siteid`** | String | 投放位置 | `GoogleSearch`, `YouTube` |
| **`af_status`** | String | 归因状态 | `Non-organic`（付费）, `Organic`（自然） |
| **`click_time`** | String | 点击时间 | CTIT 分析、反作弊 |
| **`install_time`** | String | 安装时间 | 用户获取时间、Cohort 分析 |
| **`match_type`** | String | 匹配类型 | `srn`（自归因网络）, `id_matching` |
| **`referrer_gclid`** | String | Google Click ID | Google Ads 增强转化回传 |
| **`external_account_id`** | Number | 广告账户 ID | 账户维度分析 |
| **`cost_cents_USD`** | String | 花费（美分） | 通常为 0，需从广告平台获取 |
| **`af_ad_type`** | String | 广告类型 | `ClickToDownload`, `Video` |
| **`is_first_launch`** | Boolean | 是否首次启动 | 区分新用户/老用户 |
| **`retargeting_conversion_type`** | String | 再营销类型 | `none`, `re-engagement` |

### 归因窗口字段

| 字段 | 说明 | 默认值 |
|-----|------|-------|
| `af_click_lookback` | 点击归因窗口 | `30d` |
| `af_viewthrough_lookback` | 展示归因窗口 | `1d` |
| `af_reengagement_window` | 再营销归因窗口 | `30d` |

---

## 数据分析应用

### 可支持的分析类型

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         分析能力矩阵                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   分析类型              数据来源                      可行性                 │
│                                                                             │
│   ✅ 渠道归因分析        appsflyer_data.media_source   完全支持             │
│                         appsflyer_data.campaign                             │
│                                                                             │
│   ✅ 漏斗转化分析        event_type (OStep1/OStep2)    完全支持             │
│                                                                             │
│   ✅ Campaign 效果       campaign + campaign_id        完全支持             │
│                                                                             │
│   ✅ 广告组效果          af_adset + af_adset_id        完全支持             │
│                                                                             │
│   ✅ 用户质量分析        关联留存/转化数据              完全支持             │
│                                                                             │
│   ✅ CTIT 分析           click_time → install_time     完全支持             │
│      (反作弊)                                                               │
│                                                                             │
│   ⚠️ ROAS 分析           Revenue ✅ / Cost ❌          需补充花费数据       │
│                         Cost 需从广告平台 API 获取                          │
│                                                                             │
│   ⚠️ 地区分析            country 字段可能为空          需从 IP 反查         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 数据关联方式

```
回传数据关联策略：

┌──────────────┐      user_id      ┌──────────────┐
│  回传数据     │ ◀──────────────▶ │  留存数据     │
│  (Postback)  │                   │  (Retention) │
└──────┬───────┘                   └──────────────┘
       │
       │ user_id
       │
       ▼
┌──────────────┐      user_id      ┌──────────────┐
│  用户归因表   │ ◀──────────────▶ │  转化数据     │
│  (首次归因)   │                   │  (Revenue)   │
└──────┬───────┘                   └──────────────┘
       │
       │ campaign_id
       │
       ▼
┌──────────────┐
│  广告花费     │  ← 从 Google Ads API 获取
│  (Cost)      │
└──────────────┘
```

---

## 数据表设计

### 实际生产表：af_raw_data（Doris/StarRocks）

这是从 AppsFlyer Pull API / Push API 拉取的原始数据表：

```sql
-- bi_ods.af_raw_data (Doris/StarRocks)
CREATE TABLE `af_raw_data` (
  -- ========== 主键字段 ==========
  `App_ID` varchar(255) NULL DEFAULT "",                    -- 应用ID
  `AppsFlyer_ID` varchar(255) NOT NULL,                     -- AppsFlyer 设备ID
  `Event_Name` varchar(255) NULL DEFAULT "",                -- 事件名称
  `Event_Time` datetime NOT NULL,                           -- 事件时间
  
  -- ========== 应用信息 ==========
  `App_Name` varchar(255) NULL DEFAULT "",                  -- 应用名称
  `Bundle_ID` varchar(255) NULL DEFAULT "",                 -- Bundle ID
  `Project` varchar(255) NULL DEFAULT "",                   -- 项目名称
  
  -- ========== 归因触点信息 ==========
  `Attributed_Touch_Type` varchar(32) NULL DEFAULT "",      -- 归因触点类型 (click/impression)
  `Attributed_Touch_Time` datetime NULL,                    -- 归因触点时间
  `Install_Time` datetime NULL,                             -- 安装时间
  
  -- ========== 事件数据 ==========
  `Event_Value` varchar(1000) NULL DEFAULT "",              -- 事件值（JSON格式）
  `Event_Revenue` decimal(20,2) NULL,                       -- 事件收入（本币）
  `Event_Revenue_Currency` varchar(32) NULL DEFAULT "",     -- 收入币种
  `Event_Revenue_USD` decimal(20,2) NULL,                   -- 事件收入（USD）
  `Event_Source` varchar(32) NULL DEFAULT "",               -- 事件来源
  `Is_Receipt_Validated` varchar(32) NULL DEFAULT "",       -- 收据是否验证
  
  -- ========== 归因层级（核心） ==========
  `Partner` varchar(50) NULL DEFAULT "",                    -- 合作伙伴
  `Media_Source` varchar(50) NULL DEFAULT "",               -- 媒体来源 ⭐
  `Channel` varchar(255) NULL DEFAULT "",                   -- 渠道
  `Keywords` varchar(255) NULL DEFAULT "",                  -- 关键词
  `Campaign` varchar(255) NULL DEFAULT "",                  -- 广告系列名称 ⭐
  `Campaign_ID` varchar(32) NULL DEFAULT "",                -- 广告系列ID ⭐
  `Adset` varchar(255) NULL DEFAULT "",                     -- 广告组名称 ⭐
  `Adset_ID` varchar(32) NULL DEFAULT "",                   -- 广告组ID ⭐
  `Ad` varchar(255) NULL DEFAULT "",                        -- 广告名称
  `Ad_ID` varchar(255) NULL DEFAULT "",                     -- 广告ID
  `Ad_Type` varchar(255) NULL DEFAULT "",                   -- 广告类型
  `Site_ID` varchar(255) NULL DEFAULT "",                   -- 站点ID
  `Sub_Site_ID` varchar(255) NULL DEFAULT "",               -- 子站点ID
  
  -- ========== 自定义参数 ==========
  `Sub_Param_1` varchar(255) NULL DEFAULT "",               -- 自定义参数1
  `Sub_Param_2` varchar(255) NULL DEFAULT "",               -- 自定义参数2
  `Sub_Param_3` varchar(255) NULL DEFAULT "",               -- 自定义参数3
  `Sub_Param_4` varchar(255) NULL DEFAULT "",               -- 自定义参数4
  `Sub_Param_5` varchar(255) NULL DEFAULT "",               -- 自定义参数5
  
  -- ========== 花费信息 ==========
  `Cost_Model` varchar(255) NULL DEFAULT "",                -- 计费模式
  `Cost_Value` varchar(255) NULL DEFAULT "",                -- 花费值
  `Cost_Currency` varchar(255) NULL DEFAULT "",             -- 花费币种
  
  -- ========== 多触点归因（助攻渠道） ==========
  `Contributor_1_Partner` varchar(255) NULL DEFAULT "",     -- 助攻1-合作伙伴
  `Contributor_1_Media_Source` varchar(255) NULL DEFAULT "",-- 助攻1-媒体来源
  `Contributor_1_Campaign` varchar(255) NULL DEFAULT "",    -- 助攻1-广告系列
  `Contributor_1_Touch_Type` varchar(255) NULL DEFAULT "",  -- 助攻1-触点类型
  `Contributor_1_Touch_Time` varchar(255) NULL DEFAULT "",  -- 助攻1-触点时间
  `Contributor_2_Partner` varchar(255) NULL DEFAULT "",     -- 助攻2-合作伙伴
  `Contributor_2_Media_Source` varchar(255) NULL DEFAULT "",-- 助攻2-媒体来源
  `Contributor_2_Campaign` varchar(255) NULL DEFAULT "",    -- 助攻2-广告系列
  `Contributor_2_Touch_Type` varchar(255) NULL DEFAULT "",  -- 助攻2-触点类型
  `Contributor_2_Touch_Time` varchar(255) NULL DEFAULT "",  -- 助攻2-触点时间
  `Contributor_3_Partner` varchar(255) NULL DEFAULT "",     -- 助攻3-合作伙伴
  `Contributor_3_Media_Source` varchar(255) NULL DEFAULT "",-- 助攻3-媒体来源
  `Contributor_3_Campaign` varchar(255) NULL DEFAULT "",    -- 助攻3-广告系列
  `Contributor_3_Touch_Type` varchar(255) NULL DEFAULT "",  -- 助攻3-触点类型
  `Contributor_3_Touch_Time` varchar(255) NULL DEFAULT "",  -- 助攻3-触点时间
  
  -- ========== 地理信息 ==========
  `Region` varchar(100) NULL DEFAULT "",                    -- 地区
  `Country_Code` varchar(8) NULL DEFAULT "",                -- 国家代码 ⭐
  `State` varchar(255) NULL DEFAULT "",                     -- 州/省
  `City` varchar(255) NULL DEFAULT "",                      -- 城市
  `Postal_Code` varchar(255) NULL DEFAULT "",               -- 邮编
  `DMA` varchar(255) NULL DEFAULT "",                       -- 指定市场区域
  `IP` varchar(255) NULL DEFAULT "",                        -- IP地址
  
  -- ========== 网络信息 ==========
  `WIFI` varchar(255) NULL DEFAULT "",                      -- 是否WIFI
  `Operator` varchar(255) NULL DEFAULT "",                  -- 运营商
  `Carrier` varchar(255) NULL DEFAULT "",                   -- 运营商
  `Language` varchar(255) NULL DEFAULT "",                  -- 语言
  
  -- ========== 设备标识 ==========
  `Advertising_ID` varchar(255) NULL DEFAULT "",            -- 广告ID (GAID)
  `IDFA` varchar(255) NULL DEFAULT "",                      -- iOS IDFA
  `Android_ID` varchar(255) NULL DEFAULT "",                -- Android ID
  `Customer_User_ID` varchar(255) NULL DEFAULT "",          -- 客户用户ID ⭐
  `IMEI` varchar(255) NULL DEFAULT "",                      -- IMEI
  `IDFV` varchar(255) NULL DEFAULT "",                      -- iOS IDFV
  
  -- ========== 设备信息 ==========
  `Platform` varchar(255) NULL DEFAULT "",                  -- 平台 ⭐
  `Device_Type` varchar(255) NULL DEFAULT "",               -- 设备类型
  `OS_Version` varchar(255) NULL DEFAULT "",                -- 系统版本
  `App_Version` varchar(255) NULL DEFAULT "",               -- App版本
  `SDK_Version` varchar(255) NULL DEFAULT "",               -- SDK版本
  
  -- ========== 再营销信息 ==========
  `Is_Retargeting` varchar(255) NULL DEFAULT "",            -- 是否再营销
  `Retargeting_Conversion_Type` varchar(255) NULL DEFAULT "",-- 再营销转化类型
  `Attribution_Lookback` varchar(255) NULL DEFAULT "",      -- 归因回溯窗口
  `Reengagement_Window` varchar(255) NULL DEFAULT "",       -- 再激活窗口
  `Is_Primary_Attribution` varchar(255) NULL DEFAULT "",    -- 是否主归因
  
  -- ========== 请求信息 ==========
  `User_Agent` varchar(512) NULL DEFAULT "",                -- UA
  `HTTP_Referrer` varchar(4096) NULL DEFAULT "",            -- HTTP Referrer
  `Original_URL` text NULL DEFAULT ""                       -- 原始URL
  
) ENGINE=OLAP
AGGREGATE KEY(`App_ID`, `AppsFlyer_ID`, `Event_Name`, `Event_Time`)
PARTITION BY RANGE(`Event_Time`) (...)
DISTRIBUTED BY HASH(`App_ID`, `AppsFlyer_ID`, `Event_Time`) BUCKETS 16;
```

### 字段分类说明

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         af_raw_data 字段分类                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   🔑 主键/标识字段                                                           │
│   ├── App_ID             应用标识                                           │
│   ├── AppsFlyer_ID       设备级唯一标识（AppsFlyer生成）                      │
│   ├── Customer_User_ID   业务用户ID（你们自己的user_id）                      │
│   └── Event_Time         事件时间                                           │
│                                                                             │
│   📊 事件字段                                                                │
│   ├── Event_Name         事件名称（install/af_purchase/自定义事件）           │
│   ├── Event_Value        事件值（JSON格式的附加数据）                         │
│   ├── Event_Revenue      事件收入（本币）                                    │
│   └── Event_Revenue_USD  事件收入（美元）⭐ 用于ROAS计算                      │
│                                                                             │
│   🎯 归因层级（核心）                                                        │
│   ├── Media_Source       媒体来源（googleadwords_int/Facebook Ads等）        │
│   ├── Campaign           广告系列名称                                        │
│   ├── Campaign_ID        广告系列ID（用于关联花费）                           │
│   ├── Adset / Adset_ID   广告组                                             │
│   ├── Ad / Ad_ID         广告/素材                                          │
│   └── Keywords           关键词（搜索广告）                                   │
│                                                                             │
│   🌍 地理字段                                                                │
│   ├── Country_Code       国家代码（US/CN/JP等）                              │
│   ├── Region / State     地区/州省                                          │
│   └── City               城市                                               │
│                                                                             │
│   📱 设备字段                                                                │
│   ├── Platform           平台（android/ios）                                 │
│   ├── Device_Type        设备类型                                           │
│   ├── IDFA / GAID        设备广告ID                                         │
│   └── OS_Version         系统版本                                           │
│                                                                             │
│   🔄 多触点归因                                                              │
│   ├── Contributor_1_*    第1助攻渠道                                        │
│   ├── Contributor_2_*    第2助攻渠道                                        │
│   └── Contributor_3_*    第3助攻渠道                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 推荐的衍生表：渠道日度汇总表

```sql
-- bi_dws.dws_channel_daily_stats
CREATE TABLE `dws_channel_daily_stats` (
  `stat_date` date NOT NULL COMMENT '统计日期',
  `app_id` varchar(255) NOT NULL COMMENT '应用ID',
  `media_source` varchar(50) NOT NULL COMMENT '媒体来源',
  `campaign` varchar(255) NULL DEFAULT "" COMMENT '广告系列',
  `campaign_id` varchar(32) NULL DEFAULT "" COMMENT '广告系列ID',
  `adset` varchar(255) NULL DEFAULT "" COMMENT '广告组',
  `adset_id` varchar(32) NULL DEFAULT "" COMMENT '广告组ID',
  `country_code` varchar(8) NULL DEFAULT "" COMMENT '国家',
  `platform` varchar(32) NULL DEFAULT "" COMMENT '平台',
  
  -- 规模指标
  `installs` bigint NULL DEFAULT 0 COMMENT '安装数',
  `events` bigint NULL DEFAULT 0 COMMENT '事件数',
  `unique_users` bigint NULL DEFAULT 0 COMMENT '活跃用户数',
  
  -- 收入指标（来自 af_raw_data.Event_Revenue_USD）
  `revenue_usd` decimal(20,2) NULL DEFAULT 0 COMMENT '总收入USD',
  `payers` bigint NULL DEFAULT 0 COMMENT '付费用户数',
  `purchases` bigint NULL DEFAULT 0 COMMENT '付费次数',
  
  -- 花费指标（需从广告平台API获取）
  `cost_usd` decimal(20,2) NULL DEFAULT 0 COMMENT '花费USD',
  
  -- 计算指标
  `cpi` decimal(10,4) NULL COMMENT 'CPI',
  `roas` decimal(10,4) NULL COMMENT 'ROAS',
  `arpu` decimal(10,4) NULL COMMENT 'ARPU',
  `pay_rate` decimal(10,4) NULL COMMENT '付费率',
  
  -- 留存指标（关联留存表计算）
  `d1_retention` decimal(10,4) NULL COMMENT 'D1留存率',
  `d7_retention` decimal(10,4) NULL COMMENT 'D7留存率',
  `d30_retention` decimal(10,4) NULL COMMENT 'D30留存率'
  
) ENGINE=OLAP
AGGREGATE KEY(`stat_date`, `app_id`, `media_source`, `campaign`, `campaign_id`, 
              `adset`, `adset_id`, `country_code`, `platform`)
PARTITION BY RANGE(`stat_date`) (...)
DISTRIBUTED BY HASH(`stat_date`, `app_id`, `media_source`) BUCKETS 8;
```

### 用户首次归因表（用户级去重）

```sql
-- bi_dwd.dwd_user_first_attribution
-- 每个用户只保留首次归因信息
CREATE TABLE `dwd_user_first_attribution` (
  `customer_user_id` varchar(255) NOT NULL COMMENT '业务用户ID',
  `appsflyer_id` varchar(255) NOT NULL COMMENT 'AppsFlyer ID',
  `app_id` varchar(255) NOT NULL COMMENT '应用ID',
  
  -- 首次归因信息
  `first_media_source` varchar(50) NULL COMMENT '首次媒体来源',
  `first_campaign` varchar(255) NULL COMMENT '首次广告系列',
  `first_campaign_id` varchar(32) NULL COMMENT '首次广告系列ID',
  `first_adset` varchar(255) NULL COMMENT '首次广告组',
  `first_adset_id` varchar(32) NULL COMMENT '首次广告组ID',
  `first_ad` varchar(255) NULL COMMENT '首次广告',
  `first_keywords` varchar(255) NULL COMMENT '首次关键词',
  
  -- 时间信息
  `install_time` datetime NULL COMMENT '安装时间',
  `first_touch_time` datetime NULL COMMENT '首次触点时间',
  `first_event_time` datetime NULL COMMENT '首次事件时间',
  
  -- 设备与地理
  `platform` varchar(32) NULL COMMENT '平台',
  `country_code` varchar(8) NULL COMMENT '国家',
  `device_type` varchar(255) NULL COMMENT '设备类型',
  
  -- 是否付费流量
  `is_organic` tinyint NULL COMMENT '是否自然量 (Media_Source为空或organic)',
  `is_retargeting` tinyint NULL COMMENT '是否再营销'
  
) ENGINE=OLAP
UNIQUE KEY(`customer_user_id`, `appsflyer_id`, `app_id`)
DISTRIBUTED BY HASH(`customer_user_id`) BUCKETS 16;
```

---

## 分析 SQL 示例（基于 af_raw_data 表）

### 1. 渠道归因分析

```sql
-- 按 Campaign 统计每日新增安装
SELECT 
    DATE(Install_Time) AS install_date,
    Media_Source,
    Campaign,
    Campaign_ID,
    Country_Code,
    Platform,
    COUNT(DISTINCT AppsFlyer_ID) AS installs,
    COUNT(DISTINCT Customer_User_ID) AS users,
    COUNT(DISTINCT CASE WHEN Media_Source != '' AND Media_Source != 'organic' 
                        THEN AppsFlyer_ID END) AS paid_installs,
    COUNT(DISTINCT CASE WHEN Media_Source = '' OR Media_Source = 'organic' 
                        THEN AppsFlyer_ID END) AS organic_installs
FROM bi_ods.af_raw_data
WHERE Event_Name = 'install'
  AND Install_Time >= '2024-08-01'
GROUP BY DATE(Install_Time), Media_Source, Campaign, Campaign_ID, Country_Code, Platform
ORDER BY install_date DESC, installs DESC;
```

### 2. 漏斗转化分析

```sql
-- 按 Campaign 分析用户漏斗转化（基于 af_raw_data）
WITH user_first_attr AS (
    -- 获取用户首次归因信息
    SELECT 
        Customer_User_ID,
        AppsFlyer_ID,
        Media_Source,
        Campaign,
        Campaign_ID,
        Install_Time,
        ROW_NUMBER() OVER(PARTITION BY Customer_User_ID ORDER BY Install_Time) AS rn
    FROM bi_ods.af_raw_data
    WHERE Event_Name = 'install'
      AND Install_Time >= '2024-08-01'
),
user_events AS (
    -- 统计用户完成的事件
    SELECT 
        Customer_User_ID,
        MAX(CASE WHEN Event_Name = 'install' THEN 1 ELSE 0 END) AS has_install,
        MAX(CASE WHEN Event_Name = 'af_complete_registration' THEN 1 ELSE 0 END) AS has_register,
        MAX(CASE WHEN Event_Name = 'af_tutorial_completion' THEN 1 ELSE 0 END) AS has_tutorial,
        MAX(CASE WHEN Event_Name = 'af_purchase' THEN 1 ELSE 0 END) AS has_purchase,
        SUM(CASE WHEN Event_Name = 'af_purchase' THEN Event_Revenue_USD ELSE 0 END) AS total_revenue
    FROM bi_ods.af_raw_data
    WHERE Event_Time >= '2024-08-01'
    GROUP BY Customer_User_ID
)
SELECT 
    ua.Media_Source,
    ua.Campaign,
    COUNT(DISTINCT ua.Customer_User_ID) AS total_users,
    SUM(ue.has_register) AS registered_users,
    SUM(ue.has_tutorial) AS tutorial_users,
    SUM(ue.has_purchase) AS payers,
    SUM(ue.total_revenue) AS total_revenue,
    ROUND(SUM(ue.has_register) / COUNT(*) * 100, 2) AS register_rate,
    ROUND(SUM(ue.has_purchase) / COUNT(*) * 100, 2) AS pay_rate,
    ROUND(SUM(ue.total_revenue) / COUNT(*), 2) AS arpu
FROM user_first_attr ua
LEFT JOIN user_events ue ON ua.Customer_User_ID = ue.Customer_User_ID
WHERE ua.rn = 1
GROUP BY ua.Media_Source, ua.Campaign
HAVING COUNT(*) >= 100
ORDER BY total_users DESC;
```

### 3. CTIT 分析（反作弊）

```sql
-- Click-to-Install Time 分析（检测点击注入等作弊）
SELECT 
    Media_Source,
    Campaign,
    COUNT(*) AS installs,
    AVG(TIMESTAMPDIFF(SECOND, Attributed_Touch_Time, Install_Time)) AS avg_ctit_seconds,
    MIN(TIMESTAMPDIFF(SECOND, Attributed_Touch_Time, Install_Time)) AS min_ctit_seconds,
    PERCENTILE_APPROX(TIMESTAMPDIFF(SECOND, Attributed_Touch_Time, Install_Time), 0.5) AS median_ctit,
    MAX(TIMESTAMPDIFF(SECOND, Attributed_Touch_Time, Install_Time)) AS max_ctit_seconds,
    -- 可疑安装（CTIT < 10秒，可能是点击注入）
    SUM(CASE WHEN TIMESTAMPDIFF(SECOND, Attributed_Touch_Time, Install_Time) < 10 THEN 1 ELSE 0 END) AS suspicious_installs,
    ROUND(SUM(CASE WHEN TIMESTAMPDIFF(SECOND, Attributed_Touch_Time, Install_Time) < 10 THEN 1 ELSE 0 END) 
          / COUNT(*) * 100, 2) AS suspicious_rate
FROM bi_ods.af_raw_data
WHERE Event_Name = 'install'
  AND Attributed_Touch_Time IS NOT NULL 
  AND Install_Time IS NOT NULL
  AND Media_Source != '' 
  AND Media_Source != 'organic'
  AND Install_Time >= '2024-08-01'
GROUP BY Media_Source, Campaign
HAVING COUNT(*) > 100
ORDER BY suspicious_rate DESC;
```

### 4. D7 ROAS 分析

```sql
-- 按 Campaign 计算 D7 ROAS（基于 af_raw_data）
WITH user_install AS (
    -- 用户安装信息
    SELECT 
        Customer_User_ID,
        AppsFlyer_ID,
        Media_Source,
        Campaign,
        Campaign_ID,
        Adset,
        Country_Code,
        Platform,
        MIN(Install_Time) AS install_time
    FROM bi_ods.af_raw_data
    WHERE Event_Name = 'install'
      AND Install_Time >= '2024-08-01'
      AND Media_Source != '' AND Media_Source != 'organic'
    GROUP BY Customer_User_ID, AppsFlyer_ID, Media_Source, Campaign, Campaign_ID, 
             Adset, Country_Code, Platform
),
user_d7_revenue AS (
    -- 用户 D7 内收入
    SELECT 
        ui.Customer_User_ID,
        ui.Campaign_ID,
        SUM(CASE 
            WHEN DATEDIFF(r.Event_Time, ui.install_time) <= 7 
            THEN r.Event_Revenue_USD ELSE 0 
        END) AS d7_revenue
    FROM user_install ui
    LEFT JOIN bi_ods.af_raw_data r 
        ON ui.Customer_User_ID = r.Customer_User_ID
        AND r.Event_Name = 'af_purchase'
    GROUP BY ui.Customer_User_ID, ui.Campaign_ID
),
campaign_stats AS (
    SELECT 
        ui.Media_Source,
        ui.Campaign,
        ui.Campaign_ID,
        ui.Country_Code,
        COUNT(DISTINCT ui.Customer_User_ID) AS installs,
        COUNT(DISTINCT CASE WHEN ur.d7_revenue > 0 THEN ui.Customer_User_ID END) AS payers,
        SUM(COALESCE(ur.d7_revenue, 0)) AS d7_revenue
    FROM user_install ui
    LEFT JOIN user_d7_revenue ur ON ui.Customer_User_ID = ur.Customer_User_ID
    GROUP BY ui.Media_Source, ui.Campaign, ui.Campaign_ID, ui.Country_Code
)
SELECT 
    cs.Media_Source,
    cs.Campaign,
    cs.Country_Code,
    cs.installs,
    cs.payers,
    cs.d7_revenue,
    gc.cost AS spend,
    -- 计算指标
    ROUND(cs.d7_revenue / NULLIF(gc.cost, 0), 4) AS d7_roas,
    ROUND(gc.cost / NULLIF(cs.installs, 0), 2) AS cpi,
    ROUND(cs.payers / NULLIF(cs.installs, 0) * 100, 2) AS pay_rate,
    ROUND(cs.d7_revenue / NULLIF(cs.installs, 0), 2) AS d7_arpu
FROM campaign_stats cs
LEFT JOIN (
    -- 花费数据（需从 Google Ads API 拉取到本地表）
    SELECT campaign_id, SUM(cost_usd) AS cost
    FROM bi_ods.google_ads_cost
    WHERE date >= '2024-08-01'
    GROUP BY campaign_id
) gc ON cs.Campaign_ID = gc.campaign_id
ORDER BY d7_roas DESC NULLS LAST;
```

### 5. 收入分析（直接使用 Event_Revenue_USD）

```sql
-- 按渠道统计收入分布
SELECT 
    DATE(Event_Time) AS event_date,
    Media_Source,
    Campaign,
    Country_Code,
    Platform,
    COUNT(DISTINCT Customer_User_ID) AS payers,
    COUNT(*) AS purchases,
    SUM(Event_Revenue_USD) AS total_revenue_usd,
    AVG(Event_Revenue_USD) AS avg_order_value,
    MAX(Event_Revenue_USD) AS max_order_value
FROM bi_ods.af_raw_data
WHERE Event_Name = 'af_purchase'
  AND Event_Revenue_USD > 0
  AND Event_Time >= '2024-08-01'
GROUP BY DATE(Event_Time), Media_Source, Campaign, Country_Code, Platform
ORDER BY event_date DESC, total_revenue_usd DESC;
```

### 6. 多触点归因分析（使用 Contributor 字段）

```sql
-- 分析助攻渠道贡献
SELECT 
    Media_Source AS last_touch_source,
    Contributor_1_Media_Source AS assist_1_source,
    Contributor_2_Media_Source AS assist_2_source,
    COUNT(DISTINCT AppsFlyer_ID) AS installs,
    SUM(CASE WHEN Contributor_1_Media_Source != '' THEN 1 ELSE 0 END) AS has_assist_1,
    SUM(CASE WHEN Contributor_2_Media_Source != '' THEN 1 ELSE 0 END) AS has_assist_2
FROM bi_ods.af_raw_data
WHERE Event_Name = 'install'
  AND Install_Time >= '2024-08-01'
  AND Media_Source != ''
GROUP BY Media_Source, Contributor_1_Media_Source, Contributor_2_Media_Source
HAVING COUNT(*) > 50
ORDER BY installs DESC
LIMIT 100;
```

---

## 注意事项

### 数据处理注意点

| 问题 | 说明 | 解决方案 |
|-----|------|---------|
| **花费数据缺失** | `af_raw_data` 表的 `Cost_Value` 通常为空 | 必须从 Google Ads / Meta API 单独拉取花费数据 |
| **Event_Value 解析** | `Event_Value` 是 JSON 字符串格式 | 使用 `JSON_EXTRACT` 或 `GET_JSON_STRING` 解析 |
| **用户ID关联** | `AppsFlyer_ID` vs `Customer_User_ID` | 优先用 `Customer_User_ID`（业务ID），缺失时用 `AppsFlyer_ID` |
| **重复事件** | 同一事件可能多次入库 | 使用主键去重：`App_ID + AppsFlyer_ID + Event_Name + Event_Time` |
| **时区问题** | `Event_Time` 可能是 UTC 时间 | 根据实际情况转换到业务时区 |
| **收入币种** | `Event_Revenue` vs `Event_Revenue_USD` | 分析时统一使用 `Event_Revenue_USD` |

### 字段映射关系

```
af_raw_data 字段        →  Google Ads 字段   →  用途
───────────────────────────────────────────────────────
Campaign_ID             →  campaign.id        →  关联花费数据
Adset_ID                →  ad_group.id        →  广告组关联
Ad_ID                   →  ad.id              →  素材关联
Keywords                →  keyword.text       →  关键词分析
Media_Source            →  -                  →  识别 googleadwords_int
Customer_User_ID        →  user_id            →  离线转化回传
```

### af_raw_data 核心字段速查

| 分析场景 | 使用字段 |
|---------|---------|
| **渠道归因** | `Media_Source`, `Campaign`, `Campaign_ID`, `Adset`, `Adset_ID` |
| **用户识别** | `Customer_User_ID`（优先）, `AppsFlyer_ID` |
| **收入分析** | `Event_Revenue_USD`, `Event_Name = 'af_purchase'` |
| **时间分析** | `Install_Time`（安装）, `Event_Time`（事件） |
| **地区分析** | `Country_Code`, `Region`, `City` |
| **设备分析** | `Platform`, `Device_Type`, `OS_Version` |
| **再营销** | `Is_Retargeting`, `Retargeting_Conversion_Type` |
| **多触点** | `Contributor_1/2/3_*` 系列字段 |

### 数据质量检查 SQL

```sql
-- af_raw_data 数据质量检查
SELECT 
    DATE(Event_Time) AS event_date,
    App_ID,
    COUNT(*) AS total_events,
    COUNT(DISTINCT AppsFlyer_ID) AS unique_devices,
    COUNT(DISTINCT Customer_User_ID) AS unique_users,
    
    -- 字段完整性检查
    SUM(CASE WHEN Media_Source = '' OR Media_Source IS NULL THEN 1 ELSE 0 END) AS missing_media_source,
    SUM(CASE WHEN Campaign = '' OR Campaign IS NULL THEN 1 ELSE 0 END) AS missing_campaign,
    SUM(CASE WHEN Country_Code = '' OR Country_Code IS NULL THEN 1 ELSE 0 END) AS missing_country,
    SUM(CASE WHEN Customer_User_ID = '' OR Customer_User_ID IS NULL THEN 1 ELSE 0 END) AS missing_user_id,
    
    -- 收入数据检查
    SUM(CASE WHEN Event_Name = 'af_purchase' THEN 1 ELSE 0 END) AS purchase_events,
    SUM(CASE WHEN Event_Name = 'af_purchase' AND (Event_Revenue_USD IS NULL OR Event_Revenue_USD = 0) 
        THEN 1 ELSE 0 END) AS purchase_without_revenue,
    SUM(Event_Revenue_USD) AS total_revenue_usd,
    
    -- 事件分布
    COUNT(DISTINCT Event_Name) AS event_types
FROM bi_ods.af_raw_data
WHERE Event_Time >= '2024-08-01'
GROUP BY DATE(Event_Time), App_ID
ORDER BY event_date DESC;
```

### 常用事件名称对照

| Event_Name | 含义 | 用途 |
|------------|------|------|
| `install` | 安装 | 用户获取统计 |
| `af_complete_registration` | 完成注册 | 注册转化 |
| `af_tutorial_completion` | 完成教程 | 用户质量 |
| `af_purchase` | 付费 | 收入归因 |
| `af_subscribe` | 订阅 | 订阅转化 |
| `af_add_to_cart` | 加入购物车 | 电商漏斗 |
| `af_level_achieved` | 升级 | 游戏参与度 |
| `session_start` | 会话开始 | 活跃分析 |
| 自定义事件名 | 业务自定义 | 根据业务需求 |

---

## 📚 相关文档

- [APPSFLYER_API_CAPABILITIES.md](./APPSFLYER_API_CAPABILITIES.md) - API 能力详解
- [APPSFLYER_DATA_ANALYSIS_GUIDE.md](./APPSFLYER_DATA_ANALYSIS_GUIDE.md) - 数据分析指南
- [APPSFLYER_INTEGRATION_GUIDE.md](./APPSFLYER_INTEGRATION_GUIDE.md) - 集成指南

---

*文档版本：v1.0 | 最后更新：2025-01*

