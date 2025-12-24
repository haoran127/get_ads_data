# 🔧 AppsFlyer SDK 集成与实施指南

> 从零到一完成 AppsFlyer 集成的完整实施指南

---

## 📋 目录

1. [集成概述](#集成概述)
2. [SDK 集成](#sdk-集成)
3. [事件追踪设计](#事件追踪设计)
4. [深度链接配置](#深度链接配置)
5. [广告平台集成](#广告平台集成)
6. [S2S 事件回传](#s2s-事件回传)
7. [SKAdNetwork 配置](#skadnetwork-配置)
8. [验证与测试](#验证与测试)

---

## 集成概述

### 集成架构

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      AppsFlyer 集成架构                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────┐                     ┌─────────────────┐               │
│   │   移动端 App    │                     │   后端服务器     │               │
│   │                 │                     │                 │               │
│   │  ┌───────────┐  │                     │  ┌───────────┐  │               │
│   │  │AppsFlyer  │  │                     │  │ S2S API   │  │               │
│   │  │   SDK     │  │                     │  │ 集成      │  │               │
│   │  └─────┬─────┘  │                     │  └─────┬─────┘  │               │
│   └────────┼────────┘                     └────────┼────────┘               │
│            │                                       │                        │
│            │  安装/事件/会话                         │  付费/订阅/后端事件     │
│            │                                       │                        │
│            ▼                                       ▼                        │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │                        AppsFlyer                                 │      │
│   ├─────────────────────────────────────────────────────────────────┤      │
│   │  归因引擎  │  数据处理  │  报表系统  │  Postback 发送             │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│                               │                                             │
│            ┌──────────────────┼──────────────────┐                         │
│            ▼                  ▼                  ▼                         │
│     ┌────────────┐     ┌────────────┐     ┌────────────┐                   │
│     │ 广告平台    │     │ 数据仓库    │     │  BI 系统   │                   │
│     │ (回传优化)  │     │ (数据分析)  │     │ (报表展示) │                   │
│     └────────────┘     └────────────┘     └────────────┘                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 集成清单

| 阶段 | 任务 | 负责人 | 重要程度 |
|-----|------|-------|---------|
| **SDK 集成** | iOS/Android SDK 集成 | 客户端开发 | 🔴 必须 |
| **事件设计** | 定义核心事件和参数 | 产品/数据 | 🔴 必须 |
| **事件实现** | 客户端上报事件代码 | 客户端开发 | 🔴 必须 |
| **深度链接** | OneLink 配置与实现 | 客户端开发 | 🟡 建议 |
| **广告集成** | 配置广告平台连接 | 投放运营 | 🔴 必须 |
| **S2S 回传** | 后端付费事件上报 | 后端开发 | 🟡 建议 |
| **SKAN 配置** | iOS SKAN CV 配置 | 产品/数据 | 🔴 必须（iOS） |
| **验证测试** | 端到端测试验证 | QA | 🔴 必须 |

---

## SDK 集成

### iOS SDK 集成

#### 1. 安装 SDK

**CocoaPods**:
```ruby
# Podfile
pod 'AppsFlyerFramework', '~> 6.14'  # 最新版本 6.14.6（2024年7月）
```

**Swift Package Manager**:
```
https://github.com/AppsFlyerSDK/AppsFlyerFramework
```

#### 2. 初始化配置

```swift
// AppDelegate.swift
import AppsFlyerLib

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication, 
                    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // 基础配置
        AppsFlyerLib.shared().appsFlyerDevKey = "YOUR_DEV_KEY"
        AppsFlyerLib.shared().appleAppID = "YOUR_APP_ID"
        
        // 可选配置
        AppsFlyerLib.shared().isDebug = true  // 开发时启用，上线关闭
        AppsFlyerLib.shared().customerUserID = "user_123"  // 设置客户用户ID
        
        // 设置委托
        AppsFlyerLib.shared().delegate = self
        
        return true
    }
    
    func applicationDidBecomeActive(_ application: UIApplication) {
        // 启动 SDK（每次 App 进入前台时调用）
        AppsFlyerLib.shared().start()
    }
}

// 实现委托方法
extension AppDelegate: AppsFlyerLibDelegate {
    
    func onConversionDataSuccess(_ conversionInfo: [AnyHashable : Any]) {
        // 归因数据回调
        print("Conversion Data: \(conversionInfo)")
        
        // 处理 Deep Link 数据
        if let isFirstLaunch = conversionInfo["is_first_launch"] as? Bool,
           isFirstLaunch == true {
            // 首次安装的归因数据
            if let mediaSource = conversionInfo["media_source"] as? String {
                print("Media Source: \(mediaSource)")
            }
        }
    }
    
    func onConversionDataFail(_ error: Error) {
        print("Conversion Data Error: \(error)")
    }
}
```

#### 3. Privacy Manifest 配置（iOS 2024年4月起必需）

自 2024 年 4 月 7 日起，Apple 要求所有使用特定 API 的应用必须提供 Privacy Manifest。AppsFlyer SDK 6.13.0+ 已内置支持。

在 `Info.plist` 中确保包含以下配置（SDK 会自动处理大部分内容）：
- NSPrivacyTracking
- NSPrivacyTrackingDomains
- NSPrivacyCollectedDataTypes

#### 4. ATT 配置（iOS 14.5+）

```swift
import AppTrackingTransparency

func requestATTPermission() {
    if #available(iOS 14.5, *) {
        ATTrackingManager.requestTrackingAuthorization { status in
            switch status {
            case .authorized:
                // 用户允许追踪
                print("ATT Authorized")
            case .denied:
                // 用户拒绝追踪
                print("ATT Denied")
            case .notDetermined:
                print("ATT Not Determined")
            case .restricted:
                print("ATT Restricted")
            @unknown default:
                break
            }
            
            // 无论用户选择如何，都启动 AppsFlyer
            AppsFlyerLib.shared().start()
        }
    } else {
        AppsFlyerLib.shared().start()
    }
}
```

### Android SDK 集成

#### 1. 添加依赖

```gradle
// build.gradle (app)
dependencies {
    implementation 'com.appsflyer:af-android-sdk:6.17.+'  // 最新版本 6.17.1（2025年7月）
    implementation 'com.android.installreferrer:installreferrer:2.2'
    // 可选：Google Play Integrity API 支持（6.17.0+）
    implementation 'com.google.android.play:integrity:1.3.0'
}
```

#### 2. 初始化配置

```kotlin
// Application.kt
import com.appsflyer.AppsFlyerLib
import com.appsflyer.AppsFlyerConversionListener

class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        val conversionListener = object : AppsFlyerConversionListener {
            override fun onConversionDataSuccess(conversionData: MutableMap<String, Any>?) {
                conversionData?.let { data ->
                    val mediaSource = data["media_source"]
                    val campaign = data["campaign"]
                    val isFirstLaunch = data["is_first_launch"] as? Boolean ?: false
                    
                    Log.d("AppsFlyer", "Media Source: $mediaSource, Campaign: $campaign")
                    
                    if (isFirstLaunch) {
                        // 处理首次安装归因
                    }
                }
            }
            
            override fun onConversionDataFail(error: String?) {
                Log.e("AppsFlyer", "Conversion Error: $error")
            }
            
            override fun onAppOpenAttribution(attributionData: MutableMap<String, String>?) {
                // Deep Link 归因
                attributionData?.let { data ->
                    Log.d("AppsFlyer", "Deep Link Data: $data")
                }
            }
            
            override fun onAttributionFailure(error: String?) {
                Log.e("AppsFlyer", "Attribution Error: $error")
            }
        }
        
        AppsFlyerLib.getInstance().apply {
            init("YOUR_DEV_KEY", conversionListener, this@MyApplication)
            setCustomerUserId("user_123")  // 可选
            setDebugLog(true)  // 开发时启用
            start(this@MyApplication)
        }
    }
}
```

#### 3. 权限配置

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="com.google.android.gms.permission.AD_ID" />
```

---

## 事件追踪设计

### 事件设计原则

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        事件设计最佳实践                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ✅ 应该追踪：                                                               │
│     • 核心转化事件（注册、付费、订阅）                                         │
│     • 漏斗关键节点（教程完成、首次XX）                                         │
│     • 收入相关事件（首充、复购、订阅续费）                                      │
│     • 用户质量指标（活跃、留存、完成度）                                        │
│                                                                             │
│  ❌ 不应追踪：                                                                │
│     • 高频低价值事件（每秒触发的事件）                                         │
│     • 敏感隐私数据（密码、身份证号等）                                         │
│     • 重复或冗余事件                                                          │
│                                                                             │
│  💡 事件命名规范：                                                            │
│     • 使用 AppsFlyer 预定义事件名（如 af_purchase）                            │
│     • 自定义事件使用小写下划线命名（如 tutorial_complete）                      │
│     • 事件名称清晰表达含义                                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 推荐事件清单

#### 游戏类 App

| 事件名 | 触发时机 | 关键参数 | 用途 |
|-------|---------|---------|------|
| `af_complete_registration` | 完成注册 | `af_registration_method` | 注册转化 |
| `af_tutorial_completion` | 完成新手教程 | `af_success`, `af_content` | 用户质量 |
| `af_level_achieved` | 升级 | `af_level`, `af_score` | 用户参与 |
| `af_purchase` | 付费 | `af_revenue`, `af_currency`, `af_content_id` | 收入归因 |
| `af_subscribe` | 订阅 | `af_revenue`, `af_currency` | 订阅转化 |
| `first_purchase` | 首充 | `af_revenue`, `af_currency` | 付费转化 |
| `retention_d1` | D1 留存 | 无 | 留存分析 |

#### 电商类 App

| 事件名 | 触发时机 | 关键参数 | 用途 |
|-------|---------|---------|------|
| `af_complete_registration` | 完成注册 | `af_registration_method` | 注册转化 |
| `af_search` | 搜索商品 | `af_search_string` | 用户意图 |
| `af_content_view` | 浏览商品 | `af_content_id`, `af_content_type`, `af_price` | 商品兴趣 |
| `af_add_to_cart` | 加入购物车 | `af_content_id`, `af_quantity`, `af_price` | 购物意向 |
| `af_initiated_checkout` | 发起结算 | `af_price`, `af_quantity` | 结算漏斗 |
| `af_purchase` | 完成购买 | `af_revenue`, `af_currency`, `af_order_id` | 收入归因 |

### 事件上报代码

#### iOS

```swift
import AppsFlyerLib

class EventTracker {
    
    // 注册事件
    static func trackRegistration(method: String) {
        AppsFlyerLib.shared().logEvent(
            AFEventCompleteRegistration,
            withValues: [
                AFEventParamRegistrationMethod: method
            ]
        )
    }
    
    // 付费事件
    static func trackPurchase(revenue: Double, currency: String, 
                              orderId: String, productId: String) {
        AppsFlyerLib.shared().logEvent(
            AFEventPurchase,
            withValues: [
                AFEventParamRevenue: revenue,
                AFEventParamCurrency: currency,
                AFEventParamOrderId: orderId,
                AFEventParamContentId: productId
            ]
        )
    }
    
    // 自定义事件
    static func trackLevelUp(level: Int, score: Int) {
        AppsFlyerLib.shared().logEvent(
            AFEventLevelAchieved,
            withValues: [
                AFEventParamLevel: level,
                AFEventParamScore: score
            ]
        )
    }
    
    // 自定义事件（非预定义）
    static func trackTutorialComplete(tutorialName: String) {
        AppsFlyerLib.shared().logEvent(
            "tutorial_complete",
            withValues: [
                "tutorial_name": tutorialName,
                "completion_time": Date().timeIntervalSince1970
            ]
        )
    }
}
```

#### Android

```kotlin
import com.appsflyer.AppsFlyerLib
import com.appsflyer.AFInAppEventType
import com.appsflyer.AFInAppEventParameterName

object EventTracker {
    
    // 注册事件
    fun trackRegistration(context: Context, method: String) {
        val eventValues = mapOf(
            AFInAppEventParameterName.REGISTRATION_METHOD to method
        )
        AppsFlyerLib.getInstance().logEvent(
            context,
            AFInAppEventType.COMPLETE_REGISTRATION,
            eventValues
        )
    }
    
    // 付费事件
    fun trackPurchase(context: Context, revenue: Double, currency: String,
                      orderId: String, productId: String) {
        val eventValues = mapOf(
            AFInAppEventParameterName.REVENUE to revenue,
            AFInAppEventParameterName.CURRENCY to currency,
            AFInAppEventParameterName.ORDER_ID to orderId,
            AFInAppEventParameterName.CONTENT_ID to productId
        )
        AppsFlyerLib.getInstance().logEvent(
            context,
            AFInAppEventType.PURCHASE,
            eventValues
        )
    }
    
    // 升级事件
    fun trackLevelUp(context: Context, level: Int, score: Int) {
        val eventValues = mapOf(
            AFInAppEventParameterName.LEVEL to level,
            AFInAppEventParameterName.SCORE to score
        )
        AppsFlyerLib.getInstance().logEvent(
            context,
            AFInAppEventType.LEVEL_ACHIEVED,
            eventValues
        )
    }
}
```

---

## 深度链接配置

### OneLink 概述

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       OneLink 深度链接流程                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   用户点击 OneLink                                                           │
│        │                                                                    │
│        ▼                                                                    │
│   ┌─────────────────┐                                                       │
│   │ 检测 App 是否    │                                                       │
│   │ 已安装          │                                                       │
│   └────────┬────────┘                                                       │
│            │                                                                │
│     ┌──────┴──────┐                                                         │
│     │             │                                                         │
│     ▼             ▼                                                         │
│  已安装         未安装                                                       │
│     │             │                                                         │
│     │             ▼                                                         │
│     │      ┌─────────────┐                                                  │
│     │      │ 跳转应用商店 │                                                  │
│     │      │ 下载安装     │                                                  │
│     │      └──────┬──────┘                                                  │
│     │             │                                                         │
│     │             ▼                                                         │
│     │      ┌─────────────┐                                                  │
│     │      │ 延迟深度链接 │                                                  │
│     │      │ (首次打开时) │                                                  │
│     │      └──────┬──────┘                                                  │
│     │             │                                                         │
│     ▼             ▼                                                         │
│   ┌─────────────────────┐                                                   │
│   │ 打开 App 并跳转到   │                                                   │
│   │ 指定页面/内容       │                                                   │
│   └─────────────────────┘                                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### iOS Universal Links 配置

#### 1. 配置 Associated Domains

```
// Xcode → Signing & Capabilities → Associated Domains
applinks:yourapp.onelink.me
```

#### 2. 处理 Universal Link

```swift
// SceneDelegate.swift (iOS 13+)
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    AppsFlyerLib.shared().continue(userActivity, restorationHandler: nil)
}

// AppDelegate.swift (iOS 12 及以下)
func application(_ application: UIApplication, 
                continue userActivity: NSUserActivity,
                restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    AppsFlyerLib.shared().continue(userActivity, restorationHandler: nil)
    return true
}
```

### Android App Links 配置

#### 1. AndroidManifest.xml

```xml
<activity android:name=".MainActivity">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https" 
              android:host="yourapp.onelink.me" />
    </intent-filter>
</activity>
```

#### 2. 处理 Deep Link

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        handleDeepLink(intent)
    }
    
    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        intent?.let { handleDeepLink(it) }
    }
    
    private fun handleDeepLink(intent: Intent) {
        val data = intent.data
        data?.let { uri ->
            // 解析 Deep Link 参数
            val productId = uri.getQueryParameter("product_id")
            val campaignId = uri.getQueryParameter("campaign_id")
            
            // 跳转到对应页面
            productId?.let { navigateToProduct(it) }
        }
    }
}
```

---

## 广告平台集成

### 自归因网络（SRN）配置

主要的 SRN 包括：Meta、Google、TikTok、Apple Search Ads 等。

#### Meta（Facebook/Instagram）配置

1. **在 AppsFlyer 中配置**：
   - 进入 Configuration → Integrated Partners → Facebook
   - 开启 Partner Integration
   - 配置 Facebook App ID
   - 设置事件映射

2. **事件映射示例**：

| AppsFlyer 事件 | Facebook 事件 | 用途 |
|---------------|--------------|------|
| `af_complete_registration` | `CompleteRegistration` | 注册优化 |
| `af_purchase` | `Purchase` | 付费优化 |
| `af_add_to_cart` | `AddToCart` | 购物车优化 |
| `first_purchase` | `FirstPurchase` | 首充优化 |

#### Google Ads 配置

1. **Link ID 获取**：在 Google Ads 中生成 Link ID
2. **在 AppsFlyer 配置**：填入 Link ID 完成关联
3. **Firebase 集成**（可选）：用于 Google 的 iOS 归因

### 非自归因网络配置

对于 Unity Ads、ironSource 等：

1. 获取 Partner 的 Click URL 和 Postback URL
2. 在广告平台配置 Tracking URL（包含 AppsFlyer 宏）
3. 在 AppsFlyer 配置 Postback 回传

---

## S2S 事件回传

### 架构设计

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       S2S 事件回传架构                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   客户端                    后端服务                     AppsFlyer           │
│                                                                             │
│   ┌─────────┐              ┌─────────┐               ┌─────────┐           │
│   │ 用户    │  订单请求    │ 订单    │  S2S API      │         │           │
│   │ 付费    │───────────▶ │ 服务    │─────────────▶ │ 归因    │           │
│   └─────────┘              └─────────┘               │ & 统计  │           │
│                                                      └────┬────┘           │
│                                                           │                 │
│                                                           ▼                 │
│                                                    ┌─────────────┐          │
│                                                    │ 广告平台     │          │
│                                                    │ (回传优化)   │          │
│                                                    └─────────────┘          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 后端实现

```python
import requests
from datetime import datetime
import json

class AppsFlyerS2S:
    def __init__(self, dev_key: str, app_id: str):
        self.dev_key = dev_key
        self.app_id = app_id
        self.base_url = f"https://api2.appsflyer.com/inappevent/{app_id}"
    
    def send_event(self, appsflyer_id: str, event_name: str, 
                   event_value: dict = None, revenue: float = None,
                   currency: str = "USD", customer_user_id: str = None,
                   event_time: datetime = None):
        """
        发送 S2S 事件到 AppsFlyer
        
        Args:
            appsflyer_id: AppsFlyer 设备ID（必须从 SDK 获取并存储）
            event_name: 事件名称
            event_value: 事件附加参数
            revenue: 收入金额
            currency: 币种
            customer_user_id: 客户用户ID
            event_time: 事件时间（默认当前时间）
        """
        headers = {
            "authentication": self.dev_key,
            "Content-Type": "application/json"
        }
        
        payload = {
            "appsflyer_id": appsflyer_id,
            "eventName": event_name,
            "af_events_api": "true"
        }
        
        # 处理事件值
        if event_value:
            if revenue is not None:
                event_value["af_revenue"] = revenue
                event_value["af_currency"] = currency
            payload["eventValue"] = json.dumps(event_value)
            payload["eventCurrency"] = currency
        elif revenue is not None:
            payload["eventValue"] = json.dumps({
                "af_revenue": revenue,
                "af_currency": currency
            })
            payload["eventCurrency"] = currency
        
        # 客户用户ID
        if customer_user_id:
            payload["customer_user_id"] = customer_user_id
        
        # 事件时间
        if event_time:
            payload["eventTime"] = event_time.strftime("%Y-%m-%d %H:%M:%S.000")
        
        response = requests.post(self.base_url, headers=headers, json=payload)
        return response.status_code == 200, response.text


# 使用示例
s2s = AppsFlyerS2S(
    dev_key="YOUR_DEV_KEY",
    app_id="com.example.app"
)

# 发送付费事件
success, message = s2s.send_event(
    appsflyer_id="1234567890123-1234567890123456789",
    event_name="af_purchase",
    event_value={
        "af_order_id": "order_12345",
        "af_content_id": "product_abc"
    },
    revenue=99.99,
    currency="USD",
    customer_user_id="user_123"
)
```

### 客户端获取 AppsFlyer ID

**iOS**:
```swift
let appsFlyerId = AppsFlyerLib.shared().getAppsFlyerUID()
// 将此 ID 发送到后端并存储
```

**Android**:
```kotlin
val appsFlyerId = AppsFlyerLib.getInstance().getAppsFlyerUID(context)
// 将此 ID 发送到后端并存储
```

---

## SKAdNetwork 配置

### Conversion Value 策略设计

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Conversion Value 设计要点                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  SKAN 限制：                                                                 │
│  • CV 范围：0-63（6 bit，SKAN 4.0）                                          │
│  • 更新窗口：首次安装后 24 小时内（可延长至 35 天，需用户活跃）                  │
│  • 隐私阈值：安装量太少时数据会被隐藏                                          │
│                                                                             │
│  设计原则：                                                                   │
│  1. 优先编码高价值信号（付费 > 注册 > 活跃）                                   │
│  2. 考虑事件发生时间（24小时内能触发的事件）                                   │
│  3. 预留部分 CV 用于测试和调整                                                │
│  4. 与投放团队对齐 CV 解读规则                                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 游戏类 CV 配置示例

| CV 范围 | 含义 | 解码规则 |
|--------|------|---------|
| 0 | 仅安装 | revenue = 0 |
| 1-5 | 完成教程，收入 $0-$5 | revenue = (CV-1) * 1 |
| 6-20 | 完成首充，收入 $1-$50 | revenue = (CV-5) * 3.33 |
| 21-45 | 多次付费，收入 $50-$200 | revenue = 50 + (CV-21) * 6.25 |
| 46-63 | 高价值用户，收入 $200+ | revenue = 200 + (CV-46) * 20 |

### 在 AppsFlyer 配置 SKAN

1. 进入 App Settings → SKAdNetwork
2. 选择 Measurement Mode（Revenue / Events / Custom）
3. 配置 CV 映射规则
4. 设置 Lock Window 时间
5. 启用 SKAN 数据收集

---

## 验证与测试

### SDK 集成验证

#### 使用 AppsFlyer Debug 日志

**iOS**:
```swift
AppsFlyerLib.shared().isDebug = true
```

**Android**:
```kotlin
AppsFlyerLib.getInstance().setDebugLog(true)
```

#### 关键检查点

| 检查项 | 验证方法 | 预期结果 |
|-------|---------|---------|
| SDK 初始化 | 查看 Debug 日志 | 无报错，显示 Dev Key |
| 安装上报 | AppsFlyer Dashboard | 测试设备安装可见 |
| 事件上报 | Dashboard → Events | 事件名称和参数正确 |
| 收入归因 | Dashboard → Revenue | 收入数值正确 |
| 归因数据 | Debug 日志 Conversion Data | 包含 media_source |

### 测试设备注册

1. 获取设备 ID（IDFA/GAID）
2. 在 AppsFlyer Dashboard 注册测试设备
3. 使用测试链接安装 App
4. 验证归因和事件数据

### 端到端测试清单

- [ ] SDK 正确初始化
- [ ] 安装被正确追踪
- [ ] 核心事件正确上报
- [ ] 事件参数完整
- [ ] 收入数据准确
- [ ] Deep Link 正确跳转
- [ ] S2S 事件正确关联
- [ ] SKAN CV 正确更新

---

## 🔗 参考资源

- [AppsFlyer SDK Integration Guide](https://support.appsflyer.com/hc/en-us/articles/207032126)
- [OneLink Deep Linking](https://support.appsflyer.com/hc/en-us/articles/208874366)
- [S2S Events API](https://support.appsflyer.com/hc/en-us/articles/207034486)
- [SKAdNetwork Integration](https://support.appsflyer.com/hc/en-us/articles/360011451778)

---

## ⚠️ 重要变更记录

| 日期 | 变更内容 |
|-----|---------|
| 2023-09-18 | API V2 Token 发布，旧版 V1 Token 弃用 |
| 2024-04-07 | iOS 应用必须包含 Privacy Manifest |
| 2024-07-24 | iOS SDK 6.14.6 发布（修复 getConversionData 问题） |
| 2025-07-28 | Android SDK 6.17.1 发布（新增 Google Play Integrity API） |

---

*文档版本：v1.1 | 最后更新：2025-01（SDK 版本已更新至最新）*

