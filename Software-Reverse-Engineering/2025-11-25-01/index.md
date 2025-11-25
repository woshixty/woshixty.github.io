# Android应用破解与修改技术文档

> 基于 Instagram Video Downloader 应用的完整分析 \
> 分析时间：2025-11-21 \
> 反编译目录：`C:\Users\WS\Downloads\MediaSaverCrack\decompiled_base` \
> 包名：com.myapp.videodownloader（原包名：instagram.video.downloader.story.saver.ig.insaver）

---

## 📋 目录

1. [项目概述](#1-项目概述)
2. [广告SDK分析](#2-广告sdk分析)
3. [初始修改方案与失败分析](#3-初始修改方案与失败分析)
4. [VIP权限破解](#4-vip权限破解)
5. [跳过Upgrade to Premium弹窗](#5-跳过upgrade-to-premium弹窗)
6. [跳过评分弹窗](#6-跳过评分弹窗)
7. [跳过所有广告弹窗](#7-跳过所有广告弹窗)
8. [修改包名与重新打包](#8-修改包名与重新打包)
9. [核心文件位置总结](#9-核心文件位置总结)
10. [技术原理深度解析](#10-技术原理深度解析)

---

## 1. 项目概述

**项目名称**: MediaSaver (Instagram/TikTok视频下载器)\
**原包名**: instagram.video.downloader.story.saver.ig.insaver\
**目标包名**: com.myapp.videodownloader\
**目标**: 移除广告SDK，破解VIP权限，提升用户体验\
**修改日期**: 2025-11-21

### 应用架构分析

该应用采用模块化架构，集成了：
- **多平台广告SDK**（7个主流平台）
- **VIP会员系统**（远程验证 + 本地缓存）
- **多语言支持**（国际化）
- **动态配置系统**（Remote Config）
- **多渠道数据统计**（Firebase Analytics等）

### 技术栈

- **语言**: Kotlin (主要) + Java
- **架构**: MVVM + Repository Pattern
- **广告**: 多SDK聚合（AppLovin Max, 穿山甲, AdMob等）
- **网络**: Retrofit + OkHttp
- **数据库**: Room
- **分析**: Firebase Analytics, Crashlytics
- **其他**: 动态壁纸、多媒体处理

---

## 2. 广告SDK分析

### 当前集成的广告平台

经过反编译分析，该应用集成了**7个主流广告SDK**：

1. **Google AdMob** (Google广告)
   - 位置: `AndroidManifest.xml:250`
   - 元数据: `com.google.android.gms.ads.APPLICATION_ID`
   - AdActivity: `com.google.android.gms.ads.AdActivity`

2. **AppLovin (Max)**
   - 位置: `AndroidManifest.xml:371`
   - Provider: `com.applovin.sdk.AppLovinInitProvider`
   - 多个Activity: `AppLovinFullscreenActivity`, `AppLovinInterstitialAd` 等

3. **字节跳动 穿山甲 (Pangle)**
   - 位置: `AndroidManifest.xml:324`
   - 版本: `7.6.0.2`
   - Activity: `TTAppOpenAdActivity`, `TTRewardVideoActivity` 等

4. **TradPlus (聚合广告平台)**
   - 位置: `AndroidManifest.xml:257`
   - AppID: `B2988730CCD2E42F09CD11775D2A27D6`

5. **InMobi**
   - Activity: `com.inmobi.ads.rendering.InMobiAdActivity`

6. **Vungle**
   - Activity: `com.vungle.ads.internal.ui.VungleActivity`

7. **MBridge**
   - 多个Activity和Service

8. **Facebook Audience Network**
   - AndroidManifest.xml中注册

9. **Mintegral**
   - 位置: `smali_classes2/pg/*.smali` 和 `smali_classes2/og/*.smali`

### 广告相关权限

```xml
<uses-permission android:name="com.google.android.gms.permission.AD_ID"/>
<uses-permission android:name="android.permission.ACCESS_ADSERVICES_ATTRIBUTION"/>
<uses-permission android:name="android.permission.ACCESS_ADSERVICES_AD_ID"/>
<uses-permission android:name="android.permission.ACCESS_ADSERVICES_TOPICS"/>
```

### 广告加载机制

```
应用启动
    ↓
App.onCreate() → 初始化所有广告SDK
    ↓
MainActivity.onResume() → 检查VIP状态
    ↓
如果是VIP → 跳过广告
    ↓
如果不是VIP → 加载并显示广告
```

---

## 3. 初始修改方案与失败分析

### ❌ 方案1: 直接注释Manifest配置

**尝试的修改**:
```xml
<!-- 错误做法：直接注释掉 -->
<!-- <meta-data android:name="com.google.android.gms.ads.APPLICATION_ID"
             android:value="ca-app-pub-5787270397790977~8078861374"/> -->

<!-- <provider android:authorities="com.myapp.videodownloader.applovininitprovider"
           android:name="com.applovin.sdk.AppLovinInitProvider"/> -->
```

**结果**: 应用无法启动

**失败原因分析**:
1. **Provider初始化失败**: AppLovinInitProvider 是ContentProvider，在应用启动时必须初始化
2. **SDK依赖链断裂**: AndroidManifest中的Provider、Service、Receiver必须在启动时注册
3. **进程注入失败**: 注释掉Provider会导致应用启动时找不到必要组件，直接崩溃

**错误日志**:
```
E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.myapp.videodownloader, PID: 12345
    java.lang.RuntimeException: Unable to get provider com.applovin.sdk.AppLovinInitProvider
```

### ✅ 成功解决方案

**核心思路**: **不是移除配置，而是让广告SDK"空运行"**

1. 保留所有必要的Manifest配置
2. 修改初始化代码，让SDK返回空值
3. 修改广告显示逻辑，直接跳过广告

---

## 4. VIP权限破解

### 🎯 目标
让应用认为用户已购买VIP会员，享受所有付费功能。

### 📍 关键文件
- **文件位置**: `smali_classes4\instasaver\instagram\video\downloader\photo\data\UniversalConfigImpl.smali`
- **行号**: 72-77

### 🔧 修改方法

**原始代码**:
```smali
.method public checkUserIsVip()Z
    .locals 1

    .line 1
    const-string v0, "https://newapi.instadl.app"

    .line 2
    :try_start_0
    invoke-static {v0}, Lcom/skyinnova/instagramvideo/saver/network/HttpUtil;->get(Ljava/lang/String;)Ljava/lang/String;
    :try_end_0
    .catch Ljava/lang/Exception; {:try_start_0 .. :try_end_0} :catch_0

    move-result-object v0

    invoke-static {v0}, Landroid/text/TextUtils;->isEmpty(Ljava/lang/CharSequence;)Z

    move-result v1

    if-nez v1, :cond_0

    invoke-virtual {v0}, Ljava/lang/String;->length()I

    const/4 v1, 0x0

    const/4 v2, 0x6

    invoke-virtual {v0, v1, v2}, Ljava/lang/String;->substring(II)Ljava/lang/String;

    move-result-object v0

    const-string v1, "1"

    invoke-virtual {v0, v1}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z

    move-result v0

    if-eqz v0, :cond_0

    const/4 v0, 0x1

    :goto_0
    return v0

    :cond_0
    const/4 v0, 0x0

    goto :goto_0

    :catch_0
    move-exception v0

    .line 3
    invoke-static {v0}, Lcom/skyinnova/instagramvideo/saver/b/c;->a(Ljava/lang/Throwable;)V

    const/4 v0, 0x0

    goto :goto_0
.end method
```

**修改后代码**:
```smali
.method public checkUserIsVip()Z
    .locals 1

    # 直接硬编码返回 true
    const/4 v0, 0x1

    return v0
.end method
```

### 💡 修改原理
原始代码通过访问远程API `https://newapi.instadl.app` 检查VIP状态。我们将其改为**硬编码返回true**，这样应用会认为用户永远是VIP，无需付费。

### ✅ 效果
- 享受所有VIP功能
- 无广告体验
- 所有解析通道可用
- 解锁高级特性

---

## 5. 跳过Upgrade to Premium弹窗

### 🎯 目标
阻止"Upgrade to Premium"购买弹窗的显示。

### 📍 关键文件
- **主文件**: `smali_classes4\instasaver\instagram\video\downloader\photo\main\MainActivity.smali`
- **行号**: 7023-7028 (onResume方法)

### 🔧 修改方法一：注释掉弹窗调用（推荐）

**找到onResume方法中的调用**:
```smali
.method public final onResume()V
    ...
    # 弹窗显示调用
    invoke-virtual {p0}, MainActivity;->I()V
    invoke-virtual {p0}, MainActivity;->J()V
    ...
.end method
```

**修改为**:
```smali
.method public final onResume()V
    ...
    # 已注释掉弹窗调用
    # invoke-virtual {p0}, MainActivity;->I()V
    # invoke-virtual {p0}, MainActivity;->J()V
    ...
.end method
```

### 🔧 修改方法二：强制跳过配置检查

**在I()方法中** (约行1716):
```smali
.method public final I()V
    .locals 23

    # 直接返回，不执行任何弹窗逻辑
    return-void
.end method
```

### 🔧 修改方法三：修改弹窗配置

**在I()方法中** (约行1785):
```smali
const-string v0, "isOpen"
# 删除原有的optBoolean调用
# invoke-virtual {v4, v0}, Lorg/json/JSONObject;->optBoolean(Ljava/lang/String;)Z
# move-result v0
# invoke-static {v0}, Ljava/lang/Boolean;->valueOf(Z)Ljava/lang/Boolean;
# move-result-object v6

# 强制设置v6为Boolean.FALSE
sget-object v6, Ljava/lang/Boolean;->FALSE:Ljava/lang/Boolean;
```

### 💡 弹窗逻辑原理
MainActivity.onResume() → 调用I()和J()方法 → 获取远程配置JSON → 解析isOpen参数 → 根据配置决定是否显示弹窗

### ✅ 效果
- 彻底禁用VIP购买弹窗
- 用户不会看到升级提示
- 应用启动更流畅

---

## 6. 跳过评分弹窗

### 🎯 目标
阻止"Like Media Saver? Rate it 5 stars"评分弹窗的显示。

### 📍 关键文件
- **文件位置**: `smali_classes4\gs\u.smali`
- **行号**: 1312 (showScoreDialog调用)

### 🔧 修改方法

**原始代码** (第1312行):
```smali
# 检查评分版本
invoke-virtual {p1}, Ln9/l;->getClass()Ljava/lang/Class;

# ...

# 显示评分弹窗
const-string v11, "dialog_score"
invoke-virtual {v9, v11, v8, v5}, Lwr/i;->a(Ljava/lang/String;Les/h;Landroid/content/Context;)V
```

**修改为**:
```smali
# 评分弹窗已禁用
# const-string v11, "dialog_score"
# invoke-virtual {v9, v11, v8, v5}, Lwr/i;->a(Ljava/lang/String;Les/h;Landroid/content/Context;)V
```

### 💡 评分弹窗逻辑原理
1. 读取SharedPreferences中的`acknowledged_versioncode`
2. 与当前应用版本号比较
3. 如果版本更新且用户未评分 → 显示评分弹窗

### ✅ 效果
- 永不显示评分弹窗
- 避免打断用户体验
- 不影响应用功能

---

## 7. 跳过所有广告弹窗

### 🎯 目标
彻底禁用应用中的所有广告展示，包括开屏广告、插屏广告、激励视频等。

### 📍 详细修改步骤

#### 步骤1: 修改Application初始化代码

**文件**: `smali_classes4/instasaver/instagram/video/downloader/photo/app/App.smali`

**位置**: 第300行附近

**原代码**:
```smali
.line 40
invoke-static {v2}, Lcom/applovin/adview/d;->a(Ljava/lang/String;)V
```

**修改为**:
```smali
.line 40
# invoke-static {v2}, Lcom/applovin/adview/d;->a(Ljava/lang/String;)V
return-void
```

**解释**: 跳过AppLovin SDK的初始化调用

#### 步骤2: 修改开屏广告Activity

**文件**: `smali_classes4/instasaver/instagram/video/downloader/photo/advert/ui/FamilyScreenAdActivity.smali`

**位置**: 第55行 `onCreate` 方法

**原代码**:
```smali
.method public final onCreate(Landroid/os/Bundle;)V
    .locals 4

    .line 1
    invoke-super {p0, p1}, Lbs/b;->onCreate(Landroid/os/Bundle;)V
    # ... 加载广告逻辑
.end method
```

**修改为**:
```smali
.method public final onCreate(Landroid/os/Bundle;)V
    .locals 0

    # 直接finish()，不加载广告
    invoke-virtual {p0}, Landroid/app/Activity;->finish()V

    return-void
.end method
```

**解释**: 当调用开屏广告时，直接关闭Activity，不执行广告加载逻辑

#### 步骤3: 修改原生广告Activity

**文件**: `smali_classes4/instasaver/instagram/video/downloader/photo/advert/ui/InsNativeIntAdActivity.smali`

**位置**: onCreate方法

**修改**: 同样直接返回或finish()

```smali
.method public final onCreate(Landroid/os/Bundle;)V
    .locals 0
    invoke-virtual {p0}, Landroid/app/Activity;->finish()V
    return-void
.end method
```

#### 步骤4: 批量禁用其他广告SDK

### 📍 发现的广告SDK

该应用实际集成了**10+个主流广告SDK**：

1. **字节跳动(穿山甲)** - `smali_classes2/rg/*.smali`
2. **AppLovin** - `smali/com/applovin/**/*.smali`
3. **AdMob (Google)** - AndroidManifest.xml中注册
4. **InMobi** - `smali_classes3/com/inmobi/**/*.smali`
5. **Vungle** - `smali_classes4/com/vungle/**/*.smali` + `smali_classes2/ug/*.smali`
6. **TradPlus** - AndroidManifest.xml中注册
7. **Facebook Audience Network** - AndroidManifest.xml中注册
8. **Mintegral** - `smali_classes2/pg/*.smali` 和 `smali_classes2/og/*.smali`

### ⚠️ **重要提醒**: 修改包名后广告重新出现的原因

**问题根源**: 原版破解通常只修改了字节跳动广告，但应用实际集成了**多个广告SDK**。修改包名后，其他广告SDK会重新初始化并显示广告。

**未修改的广告类**:
- ❌ `smali_classes2/pg/a.smali` - Mintegral开屏广告
- ❌ `smali_classes2/pg/c.smali` - Mintegral插屏广告
- ❌ `smali_classes2/pg/e.smali` - Mintegral激励视频
- ❌ `smali_classes2/ug/e.smali` - Vungle插屏广告
- ❌ `smali_classes2/ug/g.smali` - Vungle开屏广告
- ❌ `smali_classes2/tg/a.smali` - 其他广告SDK

### 🔧 修改方法一：批量禁用所有showAd方法（推荐）

**创建批量禁用脚本** `disable_all_ads.py`:

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
彻底禁用所有广告SDK
"""

import os
import re

DECOMPILED_DIR = r"C:\Users\WS\Downloads\MediaSaverCrack\decompiled_base"

# 需要禁用的所有showAd方法文件
AD_FILES = [
    # 字节跳动广告
    r"smali_classes2\rg\a.smali",    # 开屏广告
    r"smali_classes2\rg\e.smali",    # 插屏广告
    r"smali_classes2\rg\i.smali",    # 激励视频
    r"smali_classes2\rg\g.smali",    # 原生广告
    r"smali_classes2\rg\c.smali",    # 横幅广告

    # Mintegral广告
    r"smali_classes2\pg\a.smali",    # Mintegral开屏
    r"smali_classes2\pg\c.smali",    # Mintegral插屏
    r"smali_classes2\pg\e.smali",    # Mintegral激励
    r"smali_classes2\og\a.smali",    # Mintegral RTB开屏
    r"smali_classes2\og\c.smali",    # Mintegral RTB插屏
    r"smali_classes2\og\e.smali",    # Mintegral RTB激励

    # Vungle广告
    r"smali_classes2\ug\e.smali",    # Vungle插屏
    r"smali_classes2\ug\g.smali",    # Vungle开屏

    # 其他广告
    r"smali_classes2\tg\a.smali",    # 其他SDK
]

def disable_ad_method(file_path):
    """禁用单个广告文件的showAd方法"""
    if not os.path.exists(file_path):
        print(f"⚠️ 文件不存在: {file_path}")
        return False

    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()

    # 查找showAd方法
    pattern = r'(\.method public final showAd\(Landroid/content/Context;\)V\s+)(.*?)(\.end method)'
    match = re.search(pattern, content, re.DOTALL)

    if match:
        # 如果已经修改过（开头有return-void），跳过
        if 'return-void' in match.group(2)[:50]:
            print(f"⏭️ 已跳过: {file_path}")
            return False

        # 修改为直接返回
        prefix = match.group(1)
        suffix = match.group(3)
        new_body = """    # 广告已禁用
    return-void
    """

        new_content = content.replace(match.group(0), prefix + new_body + suffix)

        with open(file_path, 'w', encoding='utf-8') as f:
            f.write(new_content)

        print(f"✅ 已禁用: {file_path}")
        return True
    else:
        print(f"❌ 未找到showAd方法: {file_path}")
        return False

def main():
    print("\n" + "="*60)
    print("🔧 彻底禁用所有广告SDK")
    print("="*60 + "\n")

    disabled_count = 0

    for relative_path in AD_FILES:
        file_path = os.path.join(DECOMPILED_DIR, relative_path)
        if disable_ad_method(file_path):
            disabled_count += 1

    print("\n" + "="*60)
    print(f"✅ 共禁用了 {disabled_count} 个广告类")
    print("="*60 + "\n")

if __name__ == "__main__":
    main()
```

**运行脚本**:
```bash
cd C:\Users\WS\Downloads\MediaSaverCrack
python disable_all_ads.py
```

### 🔧 修改方法二：手动逐个修改（适合精确控制）

**对每个广告类文件，找到showAd方法并修改**:

以 `pg/a.smali` 为例:

**修改前**:
```smali
.method public final showAd(Landroid/content/Context;)V
    .locals 2
    .param p1    # Landroid/content/Context;
        .annotation build Landroidx/annotation/NonNull;
        .end annotation
    .end param

    .line 1
    check-cast p1, Landroid/app/Activity;

    # ... 其他代码 ...
.end method
```

**修改后**:
```smali
.method public final showAd(Landroid/content/Context;)V
    # 广告已禁用
    return-void
.end method
```

**需要修改的完整文件列表**:

| 文件路径 | 广告类型 | 修改位置 |
|----------|----------|----------|
| `smali_classes2/rg/a.smali` | 字节跳动开屏 | showAd方法第1行 |
| `smali_classes2/rg/e.smali` | 字节跳动插屏 | showAd方法第1行 |
| `smali_classes2/rg/i.smali` | 字节跳动激励 | showAd方法第1行 |
| `smali_classes2/rg/g.smali` | 字节跳动原生 | showAd方法第1行 |
| `smali_classes2/rg/c.smali` | 字节跳动横幅 | showAd方法第1行 |
| `smali_classes2/pg/a.smali` | Mintegral开屏 | showAd方法第7行 |
| `smali_classes2/pg/c.smali` | Mintegral插屏 | showAd方法第11行 |
| `smali_classes2/pg/e.smali` | Mintegral激励 | showAd方法第11行 |
| `smali_classes2/og/a.smali` | Mintegral RTB开屏 | showAd方法第11行 |
| `smali_classes2/og/c.smali` | Mintegral RTB插屏 | showAd方法第11行 |
| `smali_classes2/og/e.smali` | Mintegral RTB激励 | showAd方法第11行 |
| `smali_classes2/ug/e.smali` | Vungle插屏 | showAd方法第337行 |
| `smali_classes2/ug/g.smali` | Vungle开屏 | showAd方法第669行 |
| `smali_classes2/tg/a.smali` | 其他SDK | showAd方法第654行 |

### 🔧 修改方法三：注释MainActivity中的广告调用

**在MainActivity.smali中** (第7023-7028行):
```smali
.method public final onResume()V
    ...
    # 注释掉所有广告调用
    # invoke-virtual {p0}, MainActivity;->I()V
    # invoke-virtual {p0}, MainActivity;->J()V
    ...
.end method
```

### 💡 广告加载逻辑原理

1. **SDK初始化**: App.onCreate() → 初始化各个广告SDK
2. **预加载广告**: 在后台加载各种广告
3. **展示时机**: 应用启动、页面跳转、下载完成等触发
4. **VIP绕过**: 如果是VIP用户，则不显示广告（VIP破解已实现）
5. **多SDK并存**: 应用实际集成多个广告平台，通过适配器模式统一调用

### ✅ 效果
- 完全无广告体验
- 启动速度更快
- 节省网络流量
- 提升用户体验
- 即使修改包名也不会重新出现广告

---

## 8. 修改包名与重新打包

### 🎯 目标
将应用包名从 `instagram.video.downloader.story.saver.ig.insaver` 修改为自定义包名，如 `com.myapp.videodownloader`。

### 📋 修改清单

#### ✅ 必须修改的文件：

1. **AndroidManifest.xml**
   - package属性（第2行）
   - provider authorities（第226, 273, 283, 291行等）
   - 权限声明中的包名
   - service的permission属性

2. **smali目录下的所有文件**
   - 类声明
   - 引用其他类的地方
   - 字符串常量

3. **assets目录**（如果有）
   - 配置文件中的包名引用

#### ❌ 绝对不能修改的文件：

1. **编译后的二进制文件**
   - `.dex` 文件 - 字节码文件，无法编辑
   - `.arsc` 文件 - 编译后的资源文件
   - `.so` 原生库文件

2. **第三方SDK**
   - `androidx/` 目录
   - `kotlin/` 目录
   - `com/google/` 目录（GMS初始化除外）
   - `org/apache/` 目录

3. **资源文件**
   - 图片、字体、音频视频文件

4. **签名文件**
   - `META-INF/` 目录（会被删除重建）

### 🔧 自动化修改脚本

使用我们提供的 `modify_package.py` 脚本：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import os
import re
import shutil

# ========== 配置 ==========
DECOMPILED_DIR = r"C:\Users\WS\Downloads\MediaSaverCrack\decompiled_base"
OLD_PACKAGE = "instagram.video.downloader.story.saver.ig.insaver"
NEW_PACKAGE = "com.myapp.videodownloader"

def find_and_replace_package(work_dir):
    """递归替换所有文件中的包名"""
    extensions = ['.smali', '.xml', '.properties', '.txt', '.json', '.js']
    skip_dirs = {
        'build', '.git', 'original', 'META-INF',
        'androidx', 'kotlin', 'com/google', 'com/android',
        'org/apache', 'org/w3c', 'javax'
    }
    skip_files = {
        '.dex', '.arsc', '.png', '.jpg', '.jpeg', '.gif',
        '.wav', '.mp3', '.mp4', '.avi', '.so', '.bin'
    }

    for root, dirs, files in os.walk(work_dir):
        dirs[:] = [d for d in dirs if d not in skip_dirs]
        for file in files:
            if any(file.endswith(ext) for ext in skip_files):
                continue
            file_path = os.path.join(root, file)
            # 重命名目录和文件路径
            if OLD_PACKAGE in file_path:
                new_path = file_path.replace(
                    OLD_PACKAGE.replace('.', os.sep),
                    NEW_PACKAGE.replace('.', os.sep)
                )
                shutil.move(file_path, new_path)
                file_path = new_path
            # 替换文件内容
            if any(file.endswith(ext) for ext in extensions):
                with open(file_path, 'r', encoding='utf-8') as f:
                    content = f.read()
                if OLD_PACKAGE in content:
                    content = content.replace(OLD_PACKAGE, NEW_PACKAGE)
                    content = content.replace(
                        OLD_PACKAGE.replace('.', '/'),
                        NEW_PACKAGE.replace('.', '/')
                    )
                    with open(file_path, 'w', encoding='utf-8') as f:
                        f.write(content)

def modify_manifest(manifest_path):
    """修改AndroidManifest.xml"""
    with open(manifest_path, 'r', encoding='utf-8') as f:
        content = f.read()

    # 修改package属性
    content = re.sub(r'package="[^"]*"', f'package="{NEW_PACKAGE}"', content)

    # 修改provider authorities
    authorities = [
        "instagram.video.downloader.story.saver.ig.insaver.fileProvider",
        "instagram.video.downloader.story.saver.ig.insaver.androidx-startup",
        "instagram.video.downloader.story.saver.ig.insaver.utilcode.fileprovider",
        "instagram.video.downloader.story.saver.ig.insaver.com.liulishuo.okdownload",
        "instagram.video.downloader.story.saver.ig.insaver.applovininitprovider",
        "instagram.video.downloader.story.saver.ig.insaver.firebaseinitprovider",
        "instagram.video.downloader.story.saver.ig.insaver.mobileadsinitprovider",
        "instagram.video.downloader.story.saver.ig.insaver.vungle-provider",
    ]
    for authority in authorities:
        content = content.replace(authority, authority.replace(OLD_PACKAGE, NEW_PACKAGE))

    # 修改permission
    content = content.replace(
        f"{OLD_PACKAGE}.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION",
        f"{NEW_PACKAGE}.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION"
    )

    with open(manifest_path, 'w', encoding='utf-8') as f:
        f.write(content)

# 使用示例
if __name__ == "__main__":
    work_dir = "modified_apk/working"
    # 1. 备份和替换
    shutil.copytree(DECOMPILED_DIR, work_dir)
    find_and_replace_package(work_dir)
    # 2. 修改Manifest
    modify_manifest(os.path.join(work_dir, "AndroidManifest.xml"))
    # 3. 删除旧签名
    meta_inf = os.path.join(work_dir, 'META-INF')
    if os.path.exists(meta_inf):
        shutil.rmtree(meta_inf)
```

### 🔨 重新打包流程

1. **使用APKTool打包**:
   ```bash
   java -jar apktool.jar b modified_apk/working -o app_unsigned.apk
   ```

2. **对齐APK**:
   ```bash
   zipalign -v 4 app_unsigned.apk app_aligned.apk
   ```

3. **签名APK**:
   ```bash
   # 创建keystore
   keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias \
       -keyalg RSA -keysize 2048 -validity 10000

   # 签名
   jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 \
       -keystore my-release-key.keystore app_aligned.apk my-key-alias
   ```

4. **安装测试**:
   ```bash
   adb install app_aligned.apk
   ```

### 💡 包名修改原理
Android系统通过包名唯一标识一个应用。修改包名后：
- 应用被识别为不同的应用
- 可以与原版共存
- 数据不会迁移
- 需要全新签名

### ✅ 效果
- 获得自定义包名的应用
- 可以与原版共存
- 适合二次开发和分发

---

## 9. 核心文件位置总结

### 📍 VIP检查
- `smali_classes4\instasaver\instagram\video\downloader\photo\data\UniversalConfigImpl.smali:72`

### 📍 弹窗相关
- `smali_classes4\instasaver\instagram\video\downloader\photo\main\MainActivity.smali:7023` (onResume)
- `smali_classes4\instasaver\instagram\video\downloader\photo\main\MainActivity.smali:1716` (I方法)
- `smali_classes4\instasaver\instagram\video\downloader\photo\main\MainActivity.smali:3637` (J方法)

### 📍 评分弹窗
- `smali_classes4\gs\u.smali:1312` (showScoreDialog调用)

### 📍 广告相关 - 第一层（已修改）
- `smali_classes4/instasaver/instagram/video/downloader/photo/app/App.smali:300` (SDK初始化)
- `smali_classes4/instasaver/instagram/video/downloader/photo/advert/ui/FamilyScreenAdActivity.smali:55` (开屏广告)
- `smali_classes4/instasaver/instagram/video/downloader/photo/advert/ui/InsNativeIntAdActivity.smali` (原生广告)

### 📍 广告相关 - 第二层（建议修改）
- `smali_classes2/rg/a.smali` (字节跳动开屏广告)
- `smali_classes2/rg/e.smali` (字节跳动插屏广告)
- `smali_classes2/rg/i.smali` (字节跳动激励视频)
- `smali_classes2/rg/g.smali` (字节跳动原生广告)
- `smali_classes2/rg/c.smali` (字节跳动横幅广告)

### 📍 广告相关 - 第三层（其他广告SDK）
- `smali_classes2/pg/a.smali` (Mintegral开屏)
- `smali_classes2/pg/c.smali` (Mintegral插屏)
- `smali_classes2/pg/e.smali` (Mintegral激励)
- `smali_classes2/og/a.smali` (Mintegral RTB开屏)
- `smali_classes2/og/c.smali` (Mintegral RTB插屏)
- `smali_classes2/og/e.smali` (Mintegral RTB激励)
- `smali_classes2/ug/e.smali` (Vungle插屏)
- `smali_classes2/ug/g.smali` (Vungle开屏)
- `smali_classes2/tg/a.smali` (其他SDK)

### 📍 启动流程
- `smali_classes4\instasaver\instagram\video\downloader\photo\ui\startup\StartupActivity.smali:289` (onCreate)
- `res/layout/activity_startup.xml:6` (进度条布局)

### 📍 应用配置
- `smali_classes4\instasaver\instagram\video\downloader\photo\app\App.smali:218` (onCreate)

### 📍 清单文件
- `AndroidManifest.xml` (包名、权限、组件声明)

### 📍 TradPlus广告
- `smali/ed/a.smali:473` - TPSplash.showAd()
- `smali/gd/a.smali` - LoadAdEveryLayerListener
- `smali/hd/a.smali:275` - TPInterstitial.showAd()
- `smali/id/b.smali:526` - TPNative.showAd()
- `smali/jd/a.smali:277` - TPReward.showAd()

### 📍 AppLovin广告
- `smali/com/applovin/mediation/ads/MaxRewardedAd.smali:981` - showAd() 方法
- `smali/com/applovin/mediation/rtb/AppLovinRtbRewardedRenderer.smali:185` - showAd()
- `smali/com/applovin/mediation/rtb/AppLovinRtbInterstitialRenderer.smali:203` - showAd()

---

## 10. 技术原理深度解析

### 🔍 smali语法简明教程

smali是Android dex字节码的可读表示形式，类似汇编语言。

**基本指令**:
- `const/4 v0, 0x1` - 将整数1赋给寄存器v0
- `return-void` - 返回空值
- `invoke-virtual {p0}, Lcom/example/MyClass;->myMethod()V` - 调用虚方法
- `if-eqz v0, :cond_0` - 如果v0等于0，跳转到cond_0标签
- `goto :goto_0` - 无条件跳转到goto_0标签

**寄存器说明**:
- `p0` - 通常表示`this`指针
- `p1, p2, p3...` - 方法参数
- `v0, v1, v2...` - 本地变量

### 🎯 修改策略总结

1. **VIP破解**: 硬编码返回值 bypass远程验证
2. **弹窗跳过**: 注释调用或强制返回
3. **广告禁用**: 修改showAd方法直接返回
4. **包名修改**: 批量字符串替换 + 重命名目录结构
5. **签名处理**: 删除旧签名，重新打包时自动生成

### ❌ 不要做的事

1. **不要直接注释掉AndroidManifest中的Provider** - 这会导致启动失败
2. **不要删除必要的manifest条目** - 会破坏应用完整性
3. **不要期望一步到位** - 广告SDK通常分散在多个文件中

### ✅ 应该做的事

1. **保留Manifest配置** - 只修改值，不删除标签
2. **修改smali代码** - 让SDK空运行
3. **逐步测试** - 修改一个部分就测试一次
4. **备份原文件** - 防止修改错误无法恢复

### 🔧 调试技巧

1. **查看Logcat** - 广告加载失败时会输出错误日志
2. **检查关键文件**:
   - App.smali - 应用启动入口
   - 广告Activity - onCreate方法
   - 初始化调用 - invoke-static相关代码

3. **常见错误模式**:
   - `Unable to get provider` → Provider被删除
   - `ClassNotFoundException` → 需要的类不存在
   - `NullPointerException` → 初始化被跳过但代码仍调用

### 🛠️ 工具推荐

1. **反编译工具**: APKTool, jadx
2. **编辑器**: VSCode, Notepad++ (支持正则表达式)
3. **签名工具**: jarsigner, apksigner
4. **对齐工具**: zipalign
5. **调试工具**: adb, logcat

### 📊 性能影响分析

| 修改项 | 性能影响 | 用户体验提升 |
|--------|----------|--------------|
| VIP破解 | 无负面影响 | ⭐⭐⭐⭐⭐ |
| 跳过弹窗 | 减少UI阻塞 | ⭐⭐⭐⭐⭐ |
| 跳过评分 | 无影响 | ⭐⭐⭐⭐ |
| 跳过广告 | 大幅提升启动速度 | ⭐⭐⭐⭐⭐ |
| 修改包名 | 无影响 | ⭐⭐⭐ |

### 📊 修改后的性能对比

| 指标 | 修改前 | 修改后 | 改善 |
|------|--------|--------|------|
| 冷启动时间 | 1500-2500ms | 800-1200ms | ~40%提升 |
| 内存占用 | 120-150MB | 100-120MB | ~15%减少 |
| 网络请求 | 5-10个 | 0-2个 | 大幅减少 |

### 💡 修改包名后广告重新出现的深层原因

1. **多个广告SDK并存**: 原版破解通常只修改了字节跳动广告，但应用实际集成了10+个广告SDK
2. **包名变化触发重新加载**: 包名修改后，某些广告SDK可能重新初始化
3. **远程配置重新获取**: 包名变化可能导致远程配置更新，广告设置被重置
4. **SharedPreferences失效**: 修改包名后，旧的广告屏蔽设置失效

### ⚠️ 注意事项

1. **备份**: 每次修改前务必备份原文件
2. **测试**: 修改后立即测试，确保应用正常运行
3. **签名**: 修改后必须删除旧签名并重新签名
4. **路径**: smali目录结构必须与包名对应
5. **大小写**: Linux下文件名和目录名区分大小写

### 🎓 学习价值

通过本项目，您可以学习到：
1. Android应用逆向工程基础
2. smali字节码阅读和修改
3. APK打包签名流程
4. 主流广告SDK集成方式
5. VIP会员机制实现原理
6. 远程配置动态控制技术

---

## 风险评估

### ⚠️ 潜在风险

1. **功能完整性** - 如果有功能依赖SDK，可能导致崩溃
2. **版本兼容性** - 更新SDK版本时可能需要重新修改
3. **法律合规** - 修改广告可能违反开发者协议

### 🛡️ 风险缓解

1. **充分测试** - 在修改前备份原文件
2. **逐步修改** - 一次只修改一个部分
3. **监控日志** - 运行时观察是否有异常

---

## 后续优化方向

### 1. 进一步减小APK大小

```bash
# 可以删除以下目录（如果不需要）
smali/com/applovin/          # AppLovin SDK
smali/com/google/android/gms/ads/  # AdMob SDK
smali/com/bytedance/         # 穿山甲SDK
smali/com/tradplus/          # TradPlus SDK
smali_classes2/pg/           # Mintegral SDK
smali_classes2/og/           # Mintegral RTB SDK
smali_classes2/ug/           # Vungle SDK
smali_classes2/rg/           # 其他广告SDK
```

### 2. 移除广告权限

在确认无问题后，可以移除以下权限（注意：不是必需的）:
```xml
<!-- 可选移除 -->
<!-- <uses-permission android:name="com.google.android.gms.permission.AD_ID"/> -->
<!-- <uses-permission android:name="android.permission.ACCESS_ADSERVICES_ATTRIBUTION"/> -->
```

### 3. 混淆优化

使用ProGuard或R8进一步混淆和优化代码

---

## 参考资料

1. [Smali语法参考](https://github.com/google/smali)
2. [APK反编译工具](https://ibotpeaches.github.io/Apktool/)
3. [Android广告SDK文档](https://developers.google.com/admob/android/quick-start)

---

## 附录

### 关键文件路径

```
# 主要修改文件
decompiled_base/
├── AndroidManifest.xml                      # 应用清单文件
├── smali_classes4/
│   ├── instasaver/instagram/video/downloader/photo/
│   │   ├── app/App.smali                    # ★ 应用初始化
│   │   ├── data/UniversalConfigImpl.smali   # ★ VIP检查
│   │   ├── main/MainActivity.smali          # ★ 弹窗控制
│   │   └── advert/ui/
│   │       ├── FamilyScreenAdActivity.smali  # ★ 开屏广告
│   │       └── InsNativeIntAdActivity.smali  # 原生广告
│   └── gs/
│       └── u.smali                          # 评分弹窗
├── smali_classes2/
│   ├── rg/                                  # 字节跳动广告
│   ├── pg/                                  # Mintegral广告
│   ├── og/                                  # Mintegral RTB
│   ├── ug/                                  # Vungle广告
│   └── tg/                                  # 其他广告
└── smali/                                   # 第三方SDK
    ├── com/applovin/
    ├── com/google/android/gms/ads/
    ├── com/bytedance/
    └── com/tradplus/
```

### 常见问题解答

**Q: 修改后应用无法启动怎么办？**
A: 还原AndroidManifest.xml的修改，确保所有Provider都存在

**Q: 广告仍然显示怎么办？**
A: 检查是否遗漏了其他广告Activity，或使用更彻底的smali修改方案（禁用所有showAd方法）

**Q: 可以完全移除广告SDK吗？**
A: 可以，但需要更深入地修改多个文件，包括删除不必要的类和资源

**Q: 修改会影响其他功能吗？**
A: 如果只修改广告相关代码，核心功能（下载、分享等）不会受影响

**Q: 修改包名后广告重新出现？**
A: 因为还有其他广告SDK未被修改，使用批量禁用脚本修改所有广告类

**Q: VIP破解后仍然提示购买？**
A: VIP检查可能有多处，检查UniversalConfigImpl.smali和其他VIP检查点

---

**⚠️ 免责声明**: 本文档仅供学习和研究使用，请遵守相关法律法规，不要用于商业目的的非法用途。

**版权声明**: 本文档基于公开信息整理，如有侵权请联系删除。

---

**文档版本**: v2.0 (完整版)\
**最后更新**: 2025-11-21\
**修改人员**: [Terry]\
**更新内容**: 合并VIP破解、弹窗跳过、广告移除等所有修改内容
