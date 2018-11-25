## services.jar
JAR位置： `/system/framework/services.jar`

### 删除MIUI系统保护机制
代码位置： `com/miui/server/SecurityManagerService.smali`
```
.method private checkSystemSelfProtection(Z)V
# return null
```
