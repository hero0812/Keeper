## 接入说明

时间| 版本 |  修订内容|    作者
---|---|---|---
2019.03.13|1.7.1|1. 接入权限SDK mainland版本，完善PushSDK接入说明文档|张鸿喜
2019.01.24|1.7.0|1. 接入权限SDK v2.2.0版本，修改内部权限申请逻辑|张鸿喜
2018.09.05|1.6.3|1. 处理序列化问题</br>2. 处理部分小米设备无法获取到RegId导致无法订阅的问题|潘星海
2018.07.24|1.6.2|将mqtt的资源文件放入assets中，防止unity忽略资源|潘星海
2018.07.03|1.6.1|1. 处理重复链接问题</br>2. 优化接入过程|潘星海
2017.10.10|1.6.0|1. 增加设置点击通知后是否打开应用的接口</br>2. 重构保活模块 |   徐宁
2017.09.01|1.5.0|   增加点击Notification后跳转逻辑|   徐宁
2017.08.29|1.4.1|   修改keep|  徐宁
2016.03.23|1.2.0|  添加小米pushSDK所需配置，通用SDK初始化接口 | 刘学栋
2015.11.18|-| 增加自定义额外参数说明，详见2.6 |  何志英
2015.09.28|-|  1. 增加对ormlite第三方包的混淆keep</br>2. 组件配置中标示出包名替换 | 何志英
2015.09.07|-| 增加接口与配置说明 |  何志英
2015.09.06|-|   创建文档  |  张智会



## SDK配置
### SDK引用

#### Android studio 环境
   可参照zip包中Lib4Studio的配置，具体步骤为：
1. 拷贝libs下所有的.jar包及.aar包到自己的项目
2. 在自己项目app目录下的build.gradle中添加对libs下的.jar包及.aar包的依赖

#### Eclipse开发环境
##### 方法一：以.jar形式引用
    将Lib4Eclipse中提供的res、libs、assets目录下的资源文件，拷贝到自己项目里对应位置

    <注> 由于新版本权限SDK剔除了xml资源文件的使用，如不需要用到，请手动在您的项目中找到并删除旧版权限SDK相关的xml文件：
        1）/layout/activity_permission_support.xml
        2) /value/permissionsupport_2_2_0_values.xml
           /values-zh-rCN/permissionsupport_2_2_0_values-zh-rCN.xml 等

##### 方法二：以Lib工程形式引用
1. 导入Lib4Eclipse到eclipse工作空间；
2. 配置Lib4Eclipse，该工程作为Lib工程来使用

可选择默认设置，如有问题，可按如下步骤处理：

- 将该工程标记为library，工程 -> Properties –> Android
- 添加工程libs下jar包
- 引用library工程

待接入工程 -> Properties –> Android：让第三方待接入工程引用此Lib工程

### 配置AndroidManifest.xml
#### 配置权限
将如下权限拷贝到第三方工程AndroidManifest.xml文件中。
```XML
<!-- PushSDK所用的权限 -->
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<!--开机启动监听-->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<uses-permission android:name="android.permission.VIBRATE" />
<!--小米Push使用的权限，必须有-->
<uses-permission android:name="android.permission.BROADCAST_PACKAGE_REMOVED" />
<uses-permission android:name="android.permission.GET_TASKS" />

<!--1.2.0版本新加入，这里{$packageName}改成app的包名-->
<uses-permission android:name="{$packageName}.permission.MIPUSH_RECEIVE" />
<!--1.2.0版本新加入，这里{$packageName}改成app的包名-->
<permission
  android:name="{$packageName}.permission.MIPUSH_RECEIVE"
  android:protectionLevel="signature" />
```
可以参考Demo的AndroidManifest.xml文件。

### 配置组件
将如下内容copy至接入工程的AndroidManifest.xml文件中。并替换对应字段
```XML
<!-- 采用jar包形式引入的，需要手动在AndroidManifest.xml中声明此组件 -->
    <activity
        android:name="com.wanmei.permission.newapi.PermissionSupportActivity"
        android:configChanges="keyboardHidden|orientation|screenSize|locale|layoutDirection|fontScale"
        android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen" />
    
<!-- 推送的长连接服务 -->
<service
    android:name="com.wanmei.push.service.PushService"
    android:exported="true"
    android:persistent="true"
    android:label="PerfectPush"
    android:process="com.perfect.push.service">
    <intent-filter>
        <action android:name="com.wanmei.sdk.push.intent.action.PERFECT_PUSH" />
    </intent-filter>
</service>

<service
    android:name="com.wanmei.push.service.KeepPushAliveJobSchedulerService"
    android:persistent="true"
    android:exported="false"
    android:permission="android.permission.BIND_JOB_SERVICE"
    android:process="com.perfect.push.job_scheduler_service"/>

<!-- mqtt推送服务 收到通知的广播接收者-->
<receiver android:name="com.wanmei.push.receiver.PushArrivedReceiver">
     <intent-filter>
          <action android:name="{$packageName}.intent.action.PERFECT_PUSH_ARRIVED" />
     </intent-filter>
<intent-filter>
    <action android:name="android.intent.action.BOOT_COMPLETED" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
<intent-filter>
    <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    <category android:name="android.intent.category.DEFAULT"/>
</intent-filter>

</receiver>

<!-- 显示通知被点击后，处理的广播接收者 -->
<receiver android:name="com.wanmei.push.receiver.PushNotifyReceiver">
    <intent-filter>
         <action android:name="{$packageName}.intent.action.PERFECT_PUSH_CLICKED" />
</intent-filter>
<intent-filter>
    <action android:name="android.intent.action.BOOT_COMPLETED" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
<intent-filter>
    <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    <category android:name="android.intent.category.DEFAULT"/>
</intent-filter>
</receiver>

<!--  ************点击通知跳转自定义业务的广播接收者************  -->
<receiver android:name="com.wanmei.push.receiver.PushClickReceiver">
    <intent-filter>
        <action android:name="{$packageName}.intent.action.PUSH_CLICKED_BROADCAST_INTENT_ACTION_FLAG_WITH_LISTENER" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</receiver>

<!-- 当前PushService状态的广播监听 -->
<receiver android:name="com.wanmei.push.receiver.PushStatusReceiver">
    <intent-filter>
        <action android:name="android.intent.action.PERFECT_PUSH_STATUS_START_SUCCESS" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.PERFECT_PUSH_STATUS_START_FAIL" />
    </intent-filter>
    <intent-filter>
        <action
         android:name="android.intent.action.PERFECT_PUSH_STATUS_REFRESH" />
    </intent-filter>
</receiver>

<!-- 系统相关状态变化的广播监听 -->
<receiver android:name="com.wanmei.push.receiver.SystemStatusReceiver"
android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.PACKAGE_REMOVED" />
    <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="package" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SCREEN_ON"/>
    <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.USER_PRESENT"/>
    <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.ACTION_POWER_CONNECTED"/>
    <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED"/>
    <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</receiver>

<!-- 1.2.0版本新加入小米Push的相关 -->
<!-- Mi Push Sdk Service -->
<service
    android:name="com.xiaomi.push.service.XMPushService"
    android:enabled="true"
    android:process=":pushservice" />
<service
    android:name="com.xiaomi.push.service.XMJobService"
    android:enabled="true"
    android:exported="false"
    android:permission="android.permission.BIND_JOB_SERVICE"
    android:process=":pushservice"/>
<!-- Mi Push Sdk Message Handler -->
<service
    android:name="com.xiaomi.mipush.sdk.PushMessageHandler"
    android:enabled="true"
    android:exported="true" />

<!-- Mi Push Sdk Message Handler Service-->
<service
    android:name="com.xiaomi.mipush.sdk.MessageHandleService"
    android:enabled="true" />

<!-- Mi Push Sdk Receiver -->
<receiver
    android:name="com.xiaomi.push.service.receivers.NetworkStatusReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</receiver>

<!-- Mi Push Sdk Receiver -->
<receiver
    android:name="com.xiaomi.push.service.receivers.PingReceiver"
    android:exported="false"
    android:process=":pushservice">
    <intent-filter>
        <action android:name="com.xiaomi.push.PING_TIMER" />
    </intent-filter>
</receiver>

<!-- Mi Push Sdk Receiver -->
<receiver
    android:name="com.wanmei.push.receiver.MiPushReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="com.xiaomi.mipush.RECEIVE_MESSAGE" />
    </intent-filter>
    <intent-filter>
        <action android:name="com.xiaomi.mipush.MESSAGE_ARRIVED" />
    </intent-filter>
    <intent-filter>
        <action android:name="com.xiaomi.mipush.ERROR" />
    </intent-filter>
</receiver>
```
以上{$packageName}字符串需要替换为接入方应用的实际包名

## SDK接入
### 必须步骤
下述步骤中，mPushAgent是通过`PushAgent.getInstance()`获得的。

1. 如果接入应用有自定义的Application，则需要在onCreate()中调用`PushAgent.initPushSDKEnvironment(Context, OnInitCompleteCallback)`方法，并将onCreate()方法中其他所有的逻辑放到OnInitCompleteCallback回调当中。
2. 调用`mPushAgent.initAppInfo(String,String)`，传入申请得到的appId和appKey以初始化push数据
3. 调用`mPushAgent.setPushNotifyIcon(int)`设置收到Push后提醒用户Notification的图标。默认为apk的ic_launcher图标。
4. 在Activity的onCreate()回调中，调用`mPushAgent.openPush(Activity, OnPushOpenCallBack)`开启push
5. 在Activity的onDestroy()中，调用`mPushAgent.destroy()`释放资源。

```Java
mPushAgent = PushAgent.getInstance(MainActivity.this);
//初始化push信息
mPushAgent.initAppInfo(mAppId, mAppkey);
//设置Notification显示图标
mPushAgent.setPushNotifyIcon(R.drawable.ic_launcher);
//在logcat中输出日志
mPushAgent.setShowLog(true);
//设置点击通知栏后的业务跳转

mPushAgent.openPush(this, new PushAgent.OnPushOpenCallBack() {
    @Override
    public void openSuccess() {
        Log.v("tag","open success");
    }

    @Override
    public void openFail(final int code) {
        Log.v("tag","open failed");
    }
});
```

### 可选步骤
- 调用`mPushAgent.getDeviceId()`方法可以获得设备唯一标识。获取设备唯一标识建议在`openPush`方法的openSuccess回调分支中进行获取，此时推送的服务已经成功激活，保证了设备ID的有效性。
- 调用`mPushAgent.setPushSDKNotificationClickListener(PushSDKNotificationClickListener)`方法，设置点击通知后的监听。自定义dealWithCustomAction方法，其中pushMessage字段是在One Push后台配置的附加字段。

```Java
mPushAgent.setPushSDKNotificationClickListener(new PushAgent.PushSDKNotificationClickListener() {
    @Override
    public void dealWithCustomAction(Context context, Map<String, Object> pushMessage) {
        if (pushMessage != null) {
            Toast.makeText(getApplicationContext(), pushMessage.toString(), Toast.LENGTH_LONG).show();
        }
    }
});
```
- 调用`mPushAgent.setOpenAppAfterClickNotifaciton(boolean)`方法，设置点击Notification后是否打开接入应用。
- 调用`mPushAgent.setShowLog(boolean)`方法，设置为true时，有相关的log输出便于调试与查找问题，Log的TAG为：PERFECT_PUSH
接入成功正式发布时可以将其设置为false。
- 调用`mPushAgent.setDebugMode(boolean)`方法，sdk将进入调试模式，链接测试服务器


### 携带额外参数
PushSDK允许在推送消息时携带自定义的key-value格式参数作为额外参数，当推送消息送达客户端时，客户端会发出Broadcast来对额外参数进行广播通知。如果接入方有携带额外参数的需求，请在项目中添加该广播对应的Receiver。

广播详情如下：
Intent Action ：{$packageName}.intent.action.PERFECT_PUSH_ARRIVED_NOTIFY_EXTRA
标红的包名需要替换为接入方应用的实际包名
Intent Extra ： key 为 extra，value为对应的自定义参数信息
使用intent.getStringExtra("extra")代码即可获取自定义参数信息。

接入者需要在项目中自行实现一个BroadcastReceiver来接收广播，下面代码给出一个PushExtraNotifyReceiver的实现作为一个简单示例 ：
```
public class PushExtraNotifyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        //该类根据接入者实际需要，如果需要使用Push消息携带额外参数的功能，则要根据实际需要，复写该部分逻辑，本例中只是一个Toast提示。
        //不使用Push消息携带额外参数功能的，不需使用本类。
        String packageName = context.getApplicationContext().getPackageName();
        if (TextUtils.equals(packageName + ".intent.action.PERFECT_PUSH_ARRIVED_NOTIFY_EXTRA", intent.getAction())) {
            if (!TextUtils.isEmpty(intent.getStringExtra("extra"))) {
                //获取对应包的extra，即推送后台的附加字段key value对，JsonObject.toString类型
                Toast.makeText(context, "Received Push Extra Data: " + intent.getStringExtra("extra"), Toast.LENGTH_LONG).show();
            }
        }
    }
}
```


同时需要在AndroidManifest.xml注册广播接收组件：
```
<!--    ************ 接收push后台附加字段的广播 ************   -->
<receiver android:name="{$packageName}.PushExtraNotifyReceiver">
    <intent-filter>
         <action android:name="{$packageName}.intent.action.PERFECT_PUSH_ARRIVED_NOTIFY_EXTRA" />
    </intent-filter>
</receiver>
```

## 混淆
Push SDK提供的PushSDK_Android.jar包已经进行了混淆，如果接入Push SDK的应用需要进行混淆操作，需要在混淆文件中加入以下keep操作（混淆使用的Proguard版本为Proguard 4.10，可在官方网站下载）：
PushSDK涉及的第三方jar包，需要在混淆文件中加入以下keep操作：
```
###############接入 PushSDK 的 demo 需要###############
-keep class com.wanmei.push.** { *; }
-dontwarn push.**
###############Push SDK 相关###############
#keep Gson
-keep class com.pwrd.google.gson.** { *; }
-keep class com.wanmei.push.Proguard { *; }
-keep class * implements com.wanmei.push.Proguard { *; }
#keep ormlite
-keep class com.j256.** { *; }
-keep interface com.j256.** { *; }
-keep public class * extends com.j256.ormlite.android.apptools.OrmLiteSqliteOpenHelper { *; }
-keep class android.support.v4.** { *; }
-keep class com.wanmei.push.bean.** { *; }
-keep class com.wanmei.push.receiver.** { *; }
-keep class org.pwrd.paho.client.mqttv3.** { *; }
-keep class com.wanmei.push.PushAgent { *; }
-keep class com.wanmei.push.PushAgent$OnPushOpenCallBack { *; }
#1.2.0 版本新加入
#keep 小米推送
-keep class com.wanmei.push.receiver.MiPushReceiver { *; }
#1.5.0 版本新加入
-keep class android.os.Binder{ *; }
-keep class com.wanmei.push.util.LogUtils{ *; }
-keep class com.wanmei.push.manager.PushManager { *; }
-keep class com.wanmei.push.PushAgent$PushSDKNotificationClickListener { *; }
-keep class * implements android.os.IInterface { *; }
#1.6.0版本新加入
-keep class com.wanmei.push.manager.KeepAliveManager { *; }
-keep class com.wanmei.push.service.KeepPushAliveJobSchedulerService { *; }
-keep class * extends android.content.BroadcastReceiver
```
