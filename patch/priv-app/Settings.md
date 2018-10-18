## 设置
APK位置： `/system/priv-app/Settings/Settings.apk`

apktool命令： `apktool d *.apk`

### 优化恢复默认应用时浏览器显示包名 com.android.browser 的问题
代码位置： `com/android/settings/applications/PreferredListSettings$1.smali`
```
.method public onClick
# 搜索 com.android.browser 可以找到诸如以下的代码片段：
const-string/jumbo v4, "com.android.browser"

iget-object v5, p0, Lcom/android/settings/applications/PreferredListSettings$1;->sQ:Lcom/android/settings/applications/PreferredListSettings;

invoke-virtual {v5}, Lcom/android/settings/applications/PreferredListSettings;->getUserId()I

move-result v5

invoke-virtual {v0, v4, v5}, Landroid/content/pm/PackageManager;->setDefaultBrowserPackageNameAsUser(Ljava/lang/String;I)Z

# 将其修改为：
iget-object v5, p0, Lcom/android/settings/applications/PreferredListSettings$1;->sQ:Lcom/android/settings/applications/PreferredListSettings;

invoke-virtual {v5}, Lcom/android/settings/applications/PreferredListSettings;->getUserId()I

move-result v5

# 注意此处 setDefaultBrowserPackageNameAsUser 变成 getDefaultBrowserPackageNameAsUser
invoke-virtual {v0, v5}, Landroid/content/pm/PackageManager;->getDefaultBrowserPackageNameAsUser(I)Ljava/lang/String;
```

### 移除『我的设备』，恢复为『关于手机』
代码路径： `com/android/settings/device`
```
# 搜索代码 Lmiui/os/Build;->IS_GLOBAL_BUILD:Z ，该语句会在多个方法出现，定位代码结构与示例方法类似的布尔方法
# 将 Lmiui/os/Build;->IS_GLOBAL_BUILD:Z 修改为 Lcom/winter/mysu;->TRUE:Z
# 参考示例：
.method public static Ib(Landroid/content/Context;)Z
    .locals 1

    .prologue
    .line 231
    sget-boolean v0, Lmiui/os/Build;->IS_GLOBAL_BUILD:Z

    const/4 v0, 0x1

    if-eqz v0, :cond_0

    .line 232
    const/4 v0, 0x0

    return v0

    .line 234
    :cond_0
    invoke-static {}, Lcom/android/settings/device/a;->In()Z

    move-result v0

    return v0
.end method
```

### 移除『关于手机』的系统更新菜单
代码位置： `com/android/settings/device/MiuiDeviceInfoSettings.smali`
```
.method public onCreate(Landroid/os/Bundle;)V
# 搜索代码 miui_update 定位相关项，将诸如以下代码：
const-string/jumbo v0, "miui_update"

invoke-virtual {p0, v0}, Lcom/android/settings/device/MiuiDeviceInfoSettings;->findPreference(Ljava/lang/CharSequence;)Landroid/preference/Preference;

move-result-object v0

check-cast v0, Lcom/android/settings/widget/CustomValuePreference;

iput-object v0, p0, Lcom/android/settings/device/MiuiDeviceInfoSettings;->aog:Lcom/android/settings/widget/CustomValuePreference;

iget-boolean v0, p0, Lcom/android/settings/device/MiuiDeviceInfoSettings;->aoa:Z

if-eqz v0, :cond_0

invoke-virtual {p0}, Lcom/android/settings/device/MiuiDeviceInfoSettings;->getActivity()Landroid/app/Activity;

move-result-object v0

invoke-static {v0}, Lcom/android/settings/device/a;->abV(Landroid/content/Context;)Ljava/lang/String;

move-result-object v0

invoke-static {v0}, Landroid/text/TextUtils;->isEmpty(Ljava/lang/CharSequence;)Z

move-result v0

if-nez v0, :cond_0

iget-object v0, p0, Lcom/android/settings/device/MiuiDeviceInfoSettings;->aog:Lcom/android/settings/widget/CustomValuePreference;

const/4 v3, 0x2

invoke-virtual {v0, v3}, Lcom/android/settings/widget/CustomValuePreference;->ajN(I)V

:cond_0
const-string/jumbo v0, "model_number"

# 修改为：
const-string/jumbo v0, "miui_update"

invoke-virtual {p0, v0}, Lcom/android/settings/device/MiuiDeviceInfoSettings;->findPreference(Ljava/lang/CharSequence;)Landroid/preference/Preference;

move-result-object v0

check-cast v0, Lcom/android/settings/widget/CustomValuePreference;

iput-object v0, p0, Lcom/android/settings/device/MiuiDeviceInfoSettings;->aog:Lcom/android/settings/widget/CustomValuePreference;

invoke-virtual {p0}, Lcom/android/settings/device/MiuiDeviceInfoSettings;->getPreferenceScreen()Landroid/preference/PreferenceScreen;

move-result-object v3

# v0 对应上面的 move-result-object v0
invoke-virtual {v3, v0}, Landroid/preference/PreferenceScreen;->removePreference(Landroid/preference/Preference;)Z

const-string/jumbo v0, "model_number"

# MIUI8
.method public onCreateOptionsMenu
# return null
```

### 移除音效设置中的均衡器设置项
代码位置： `com/android/settings/HeadsetSettings.smali`
```
# 搜索 audio_effect_optimize 、equalizer 定位相应方法，可以找到诸如以下的代码：
const-string/jumbo v0, "audio_effect_optimize"

invoke-virtual {p0, v0}, Lcom/android/settings/HeadsetSettings;->findPreference(Ljava/lang/CharSequence;)Landroid/preference/Preference;

move-result-object v0

check-cast v0, Landroid/preference/PreferenceCategory;

iput-object v0, p0, Lcom/android/settings/HeadsetSettings;->bdb:Landroid/preference/PreferenceCategory;

# 省略的代码

const-string/jumbo v0, "equalizer"

invoke-virtual {p0, v0}, Lcom/android/settings/HeadsetSettings;->findPreference(Ljava/lang/CharSequence;)Landroid/preference/Preference;

move-result-object v0

iput-object v0, p0, Lcom/android/settings/HeadsetSettings;->bdc:Landroid/preference/Preference;

在方法结束 return-void 前增加以下代码：
# ->bdb 对应 audio_effect_optimize ，->bdc 对应 equalizer
iget-object v0, p0, Lcom/android/settings/HeadsetSettings;->bdb:Landroid/preference/PreferenceCategory;

iget-object v1, p0, Lcom/android/settings/HeadsetSettings;->bdc:Landroid/preference/Preference;

invoke-virtual {v0, v1}, Landroid/preference/PreferenceCategory;->removePreference(Landroid/preference/Preference;)Z
```

### 恢复『更多应用』原来的展示样式
代码位置： `com/android/settings/applications/ManageApplicationsActivity.smali`
```
.method protected onCreate
# 搜索 Lmiui/os/Build;->IS_TABLET:Z 将其改成 Lcom/winter/mysu;->TRUE:Z
```
代码位置： `com/android/settings/MiuiSettings.smali`
```
.method public onBuildStartFragmentIntent
# 安卓N及以上，将以下代码：
const-string/jumbo v1, "com.miui.securitycenter"

const-string/jumbo v2, "com.miui.appmanager.AppManagerMainActivity"

invoke-virtual {v0, v1, v2}, Landroid/content/Intent;->setClassName(Ljava/lang/String;Ljava/lang/String;)Landroid/content/Intent;

invoke-virtual {p0}, Lcom/android/settings/MiuiSettings;->isInMultiWindowMode()Z

move-result v1

if-eqz v1, :cond_2

const/high16 v1, 0x10000000

invoke-virtual {v0, v1}, Landroid/content/Intent;->addFlags(I)Landroid/content/Intent;
# 修改为：
const-class v1, Lcom/android/settings/applications/ManageApplicationsActivity;

invoke-virtual {v0, p0, v1}, Landroid/content/Intent;->setClass(Landroid/content/Context;Ljava/lang/Class;)Landroid/content/Intent;

# 安卓M及以下，将以下代码：
const-string v1, "com.miui.securitycenter"

const-string v2, "com.miui.appmanager.AppManagerMainActivity"

invoke-virtual {v0, v1, v2}, Landroid/content/Intent;->setClassName(Ljava/lang/String;Ljava/lang/String;)Landroid/content/Intent;
# 修改为：
const-class v1, Lcom/android/settings/applications/ManageApplicationsActivity;

invoke-virtual {v0, p0, v1}, Landroid/content/Intent;->setClass(Landroid/content/Context;Ljava/lang/Class;)Landroid/content/Intent;
```
代码位置： `com/android/settings/SettingsFragment.smali`
```
# 搜索代码 "com.miui.securitycenter"
# 安卓N及以上，将以下代码：
const-string/jumbo v0, "com.miui.securitycenter"

const-string/jumbo v1, "com.miui.appmanager.AppManagerMainActivity"

invoke-virtual {v9, v0, v1}, Landroid/content/Intent;->setClassName(Ljava/lang/String;Ljava/lang/String;)Landroid/content/Intent;

invoke-virtual {p0}, Lcom/android/settings/SettingsFragment;->getActivity()Landroid/app/Activity;

move-result-object v0

invoke-virtual {v0}, Landroid/app/Activity;->isInMultiWindowMode()Z

move-result v0

if-eqz v0, :cond_1

const/high16 v0, 0x10000000

invoke-virtual {v9, v0}, Landroid/content/Intent;->addFlags(I)Landroid/content/Intent;
# 修改为：
const-class v0, Lcom/android/settings/applications/ManageApplicationsActivity;

invoke-virtual {v9, v2, v0}, Landroid/content/Intent;->setClass(Landroid/content/Context;Ljava/lang/Class;)Landroid/content/Intent;

# 安卓M及以下，同上
```

### 移除系统与安全中的广告设置项
代码位置： `com/android/settings/SecuritySettings.smali`
```
# 搜索 ad_control_settings 定位相关方法
# 将该方法中的 Lmiui/os/Build;->IS_INTERNATIONAL_BUILD:Z 修改为 Lcom/winter/mysu;->TRUE:Z
```
代码位置： `res/xml/security_settings_debug_log.xml`
```
# 为 <PreferenceCategory android:title="@string/user_experience_program"> 增加 android:key 属性
android:key="security_settings_access_control"
```
**注意**：上述修改成功的前提是 `ad_control_settings` 所在方法必须在当前 `Preferences` 加载之后调用。

然而部分 MIUI 版本并非如此，你需要要将该方法调用移动到当前 `Preferences` 的 `addPreferencesFromResource` 方法之后。

移动前：
```
const v0, 0x7f0800a6  # security_settings_debug_log 对应的id

invoke-virtual {p0, v0}, Lcom/android/settings/SecuritySettings;->addPreferencesFromResource(I)V

const-string/jumbo v0, "user_experience_program"

invoke-virtual {v5, v0}, Landroid/preference/PreferenceScreen;->findPreference(Ljava/lang/CharSequence;)Landroid/preference/Preference;

move-result-object v0

check-cast v0, Landroid/preference/CheckBoxPreference;

iput-object v0, p0, Lcom/android/settings/SecuritySettings;->boy:Landroid/preference/CheckBoxPreference;

const-string/jumbo v0, "upload_debug_log"

invoke-virtual {v5, v0}, Landroid/preference/PreferenceScreen;->findPreference(Ljava/lang/CharSequence;)Landroid/preference/Preference;

move-result-object v0

check-cast v0, Landroid/preference/CheckBoxPreference;

iput-object v0, p0, Lcom/android/settings/SecuritySettings;->box:Landroid/preference/CheckBoxPreference;

invoke-virtual {p0}, Lcom/android/settings/SecuritySettings;->getActivity()Landroid/app/Activity;

move-result-object v0
```
移动后：
```
const v0, 0x7f0800a6

invoke-virtual {p0, v0}, Lcom/android/settings/SecuritySettings;->addPreferencesFromResource(I)V

const-string/jumbo v0, "user_experience_program"

invoke-virtual {v5, v0}, Landroid/preference/PreferenceScreen;->findPreference(Ljava/lang/CharSequence;)Landroid/preference/Preference;

move-result-object v0

check-cast v0, Landroid/preference/CheckBoxPreference;

iput-object v0, p0, Lcom/android/settings/SecuritySettings;->boy:Landroid/preference/CheckBoxPreference;

const-string/jumbo v0, "upload_debug_log"

invoke-virtual {v5, v0}, Landroid/preference/PreferenceScreen;->findPreference(Ljava/lang/CharSequence;)Landroid/preference/Preference;

move-result-object v0

check-cast v0, Landroid/preference/CheckBoxPreference;

iput-object v0, p0, Lcom/android/settings/SecuritySettings;->box:Landroid/preference/CheckBoxPreference;

# 调用 ad_control_settings 所在的方法
invoke-direct {p0}, Lcom/android/settings/SecuritySettings;->bcq()V

invoke-virtual {p0}, Lcom/android/settings/SecuritySettings;->getActivity()Landroid/app/Activity;

move-result-object v0
```

### 关于手机显示贡献者信息
代码位置： `res/values/bools.xml`
```
# 将代码 has_translation_contributors 的键值改为 true

# 修改贡献者信息
# 代码位置：
res/raw/translation_contributors
res/raw-bo-rCN/translation_contributors
res/raw-ug-rCN/translation_contributors
# 同时修改这3个文件中的文本即可
```
