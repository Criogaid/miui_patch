## 安全中心
APK位置： `/system/priv-app/SecurityCenter/SecurityCenter.apk`

apktool命令： `apktool d -r *.apk`

对 `IS_INTERNATIONAL_BUILD:Z` 统一处理的方法：将 `winter` 文件夹拷贝到 `smali/com` 目录下，将 `Lmiui/os/Build;->IS_INTERNATIONAL_BUILD:Z` 修改为 `Lcom/winter/mysu;->TRUE:Z`


### 移除安全扫描、游戏加速、应用锁、网络助手中的广告

反编译 res，在 `values/public.xml` 分别找到 `display_antivirus_Ads` 、`display_gamebooster_Ads` 、`display_gamebooster_xunyou` 、`overlay_config_datausage_purchase_enabled` （均为 bool 型）对应的 `id`，再在 `smali` 中查找其对应的布尔方法，return false
```
# 同时在路径 com/miui/networkassistant 中搜索以下方法，return false
.method public isNATrafficPurchaseAvailable()Z
.method public isPurchaseTipsEnable()Z
.method public static isDataUsagePurchaseEnabled
```

### 移除网络助手的『流量购买』标签
代码位置： `com/miui/networkassistant/ui/NetworkAssistantActivity.smali`
```
.method private checkTrafficPurchaseAvaliable
# 或者 .method private checkTrafficPurchaseEnable
# 搜索 Lcom/miui/networkassistant/utils/DeviceUtil;->IS_INTERNATIONAL_BUILD:Z 将其改成 Lcom/winter/mysu;->TRUE:Z
```

### 恢复应用权限监控&USB安装管理设置入口
代码位置： `com/miui/permcenter/MainAcitivty.smali`
```
.method protected onCreate
# 在该方法的开头可以找到诸如以下的代码：
const v1, 0x7f0e02cc  # handle_item_root 对应的id

const/16 v3, 0x17

invoke-super {p0, p1}, Lcom/miui/common/c/a;->onCreate(Landroid/os/Bundle;)V

const v0, 0x7f03011c  # pm_activity_main 对应的id

invoke-virtual {p0, v0}, Lcom/miui/permcenter/MainAcitivty;->setContentView(I)V

invoke-static {}, Lcom/miui/permcenter/d;->cz()Z

move-result v0

if-nez v0, :cond_0

invoke-static {p0}, Lcom/miui/permcenter/install/i;->getInstance(Landroid/content/Context;)Lcom/miui/permcenter/install/i;

move-result-object v0

invoke-virtual {v0}, Lcom/miui/permcenter/install/i;->aB()Z

move-result v0

# 找到该方法中调用的布尔函数()Z，例如此处的：
Lcom/miui/permcenter/d;->cz()Z
Lcom/miui/permcenter/install/i;->aB()Z
# 在 com/miui/permcenter 路径查找对应方法，return ture

# 如果你使用的是开发版，希望移除自带的ROOT权限管理
# 可将该方法中的 Lmiui/os/Build;->IS_STABLE_VERSION:Z 修改为 Lcom/winter/mysu;->TRUE:Z
```

### 移除安全体检（立即优化）完成页的资讯推荐
代码路径： `com/miui/securityscan`
```
# 在该路径中查找：key_sc_setting_news_recommend ，此代码会在两个方法出现，对布尔方法 return false
# 等同于：反编译res，在 values/public.xml 找到 preference_key_information_setting_close 对应的id，再在 smali 中查找其对应的布尔方法，return false
```

### 移除安全中心首页底部的推荐广告
注：可能导致安全中心主界面云控方案下发失效
代码位置：com/miui/securityscan/MainActivity.smali
```
# 搜索 Lmiui/os/Build;->IS_INTERNATIONAL_BUILD:Z 将其改成 Lcom/winter/mysu;->TRUE:Z
```

修改后：
```
.method private Gq()Z
    .locals 2

    sget-boolean v0, Lcom/winter/mysu;->TRUE:Z

    if-nez v0, :cond_0

    const-string/jumbo v0, "zh_CN" # 关键词："zh_CN"

    invoke-static {}, Ljava/util/Locale;->getDefault()Ljava/util/Locale;

    move-result-object v1

    invoke-virtual {v1}, Ljava/util/Locale;->toString()Ljava/lang/String;

    move-result-object v1

    invoke-virtual {v0, v1}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z

    move-result v0

    :goto_0
    return v0

    :cond_0
    const/4 v0, 0x0

    goto :goto_0
.end method
```

### 移除安全体检不必要的扫描检测项
MIUI开发组闲的蛋疼，在安全中心加入一堆野鸡的扫描检测，可通过下面的手段去掉。

代码路径： `com/miui/securityscan/model/manualitem` 、`com/miui/securityscan/model/system`
```
.method public scan()V
# 修改该方法的函数体，强制设置相关检测项状态为安全，参考示例：
.method public scan()V
    .locals 1

    sget-object v0, Lcom/miui/securityscan/model/AbsModel$State;->SAFE:Lcom/miui/securityscan/model/AbsModel$State;

    invoke-virtual {p0, v0}, Lcom/miui/securityscan/model/manualitem/AppUpdateModel;->setSafe(Lcom/miui/securityscan/model/AbsModel$State;)V

    return-void
.end method
# 其中 Lcom/miui/securityscan/model/manualitem/AppUpdateModel 须根据方法所在位置动态调整
# 如果方法在 com/miui/securityscan/model/system 路径下，同时对方法所在类中的 .method public getDesc()Ljava/lang/String; 方法作以下修改：
.method public getDesc()Ljava/lang/String;
    .locals 1

    const/4 v0, 0x0

    return-object v0
.end method

# 建议执行上述修改的相关检测项：
.
├── manualitem
│   ├── AppUpdateModel.smali  # 建议更新应用
│   ├── AutoUpdateModel.smali  # 系统智能更新
│   ├── CloudServiceModel.smali  # 小米云服务
│   ├── CloudSpaceModel.smali  # 小米云空间
│   ├── ConsumePowerRankModel.smali  # 应用耗电排行
│   ├── DataPurchaseModel.smali  # 流量购买
│   ├── DefaultAppModel.smali  # 默认应用
│   ├── FlowRankModel.smali  # 今日流量排行
│   ├── MifiInsuranceModel.smali  # 小米意外保
│   ├── MiRoamingModel.smali  # 小米漫游
│   ├── MiuiUpdaterModel.smali  # 建议更新系统版本
│   ├── PermissionRootModel.smali  # ROOT权限
│   ├── RealnameVerifyModel.smali  # 实名认证
└── system
    ├── AutoDownloadModel.smali  # 系统更新自动下载
    ├── DevModeModel.smali  # 开发者选项
    └── UsbModel.smali  # USB调试
```

### 移除安全中心设置的信息流设置项

代码位置： `com/miui/securityscan/ui/settings/SettingsActivity.smali`
```
.method protected onCreate
# 在方法结束 return-void 前增加以下代码：
const-string/jumbo v0, "category_information_setting"

invoke-virtual {p0, v0}, Lcom/miui/securityscan/ui/settings/SettingsActivity;->findPreference(Ljava/lang/CharSequence;)Landroid/preference/Preference;

move-result-object v0

check-cast v0, Landroid/preference/PreferenceCategory;

invoke-virtual {p0}, Lcom/miui/securityscan/ui/settings/SettingsActivity;->getPreferenceScreen()Landroid/preference/PreferenceScreen;

move-result-object v1

invoke-virtual {v1, v0}, Landroid/preference/PreferenceScreen;->removePreference(Landroid/preference/Preference;)Z
```

### 移除安全扫描的系统更新检测
代码路径： `com/miui/antivirus`
```
# 搜索代码 key_check_item_update，此代码会在两个方法出现，对其中的布尔方法 return false
# 移除相关设置项，代码位置： Lcom/miui/antivirus/activity/SettingsActivity.smali

.method protected onCreate
# 在该方法中搜索 preference_key_check_item_update 对应的 id（在 res/values/public.xml 可以找到）定位相关项
# 可以发现诸如以下的代码：
const v0, 0x7f0705ac # preference_key_check_item_update 对应的 id

invoke-virtual {p0, v0}, Lcom/miui/antivirus/activity/SettingsActivity;->getString(I)Ljava/lang/String;

move-result-object v0

invoke-virtual {p0, v0}, Lcom/miui/antivirus/activity/SettingsActivity;->findPreference(Ljava/lang/CharSequence;)Landroid/preference/Preference;

move-result-object v0

check-cast v0, Landroid/preference/CheckBoxPreference;

iput-object v0, p0, Lcom/miui/antivirus/activity/SettingsActivity;->avc:Landroid/preference/CheckBoxPreference;

# 在方法结束 return-void 前增加以下代码
# auQ 对应 preference_key_category_check_item, avc 对应 preference_key_check_item_update
# 在 res/values/public.xml 找到 preference_key_category_check_item 对应的 id, 然后在该方法中定位 preference_key_category_check_item
iget-object v0, p0, Lcom/miui/antivirus/activity/SettingsActivity;->auQ:Landroid/preference/PreferenceCategory;

iget-object v1, p0, Lcom/miui/antivirus/activity/SettingsActivity;->avc:Landroid/preference/CheckBoxPreference;

invoke-virtual {v0, v1}, Landroid/preference/PreferenceCategory;->removePreference(Landroid/preference/Preference;)Z
```

### 移除应用管理的资源推荐，关闭应用升级提醒
代码路径/位置： `com/miui/appmanager/model` 、`com/miui/appmanager/AppManagerMainActivity.smali`
```
# 搜索 Lmiui/os/Build;->IS_INTERNATIONAL_BUILD:Z 将其改成 Lcom/winter/mysu;->TRUE:Z
# ps：在 AppManagerMainActivity.smali 中有个存在关键代码 com.google.gms 的方法跳过不用修改
```
代码路径： `com/miui/appmanager`
```
# 搜索
am_update_app_notify 
am_ads_enable
# 对应的布尔方法，return false
```
