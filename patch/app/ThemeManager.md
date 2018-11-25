## 主题
APK位置： `/system/app/ThemeManager/ThemeManager.apk`

apktool命令： `apktool d -r *.apk`

### 删除首页banners广告位
代码路径： `com/android/thememanager`
```
.method public getAdMarker()I
# return false

.method private static isSupported
# return false
```
代码位置： `com/android/thememanager/view/AdBannerView.smali`
```
.method protected init()V
# return null
```
代码位置： `com/android/thememanager/v9/adapter/V9ElementAdapter.smali`
```
.method public onCreateViewHolder
# 在 res/values/public.xml 中找到 element_ad_banner 对应的id
# 在该方法中搜索找到的id，可以发现诸如以下的代码，删除相关代码段
# 删除前：
:pswitch_1
new-instance v0, Lcom/android/thememanager/v9/d/f;

iget-object v2, p0, Lcom/android/thememanager/v9/a/d;->at:Lcom/android/thememanager/activity/m;

const v3, 0x7f03000d  # element_ad_banner 对应的id

invoke-virtual {v1, v3, p1, v6}, Landroid/view/LayoutInflater;->inflate(ILandroid/view/ViewGroup;Z)Landroid/view/View;

move-result-object v3

iget-object v4, p0, Lcom/android/thememanager/v9/a/d;->az:Lcom/android/thememanager/v9/b/a;

invoke-direct {v0, v2, v3, v4}, Lcom/android/thememanager/v9/d/f;-><init>(Landroid/app/Fragment;Landroid/view/View;Lcom/android/thememanager/v9/b/a;)V

goto :goto_0

# 删除后：
:pswitch_1
goto :goto_0
```
