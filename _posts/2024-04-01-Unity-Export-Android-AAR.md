---
layout: post
title: Unity导出原生AAR嵌入原生
date: 2024-04-01 10:14:00
description: 与原生交互相关
tags: Unity Workflow
categories: 
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

## Unity原生接入

为了更好的适应跨平台的开发需求，需要Unity结合原生开发

* 环境 Unity 2022.3.21f1f1

### Unity导出设置
切换当前平台到对应平台，这里用安卓平台举例

切换当前平台到对应安卓平台，如果未安装安卓插件去 Unityhub/Installs/your Editor/Add moudles

进入项目设置
<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Unity_Setting.png" class="img-fluid rounded z-depth-1" %}
</div> 具体目标平台根据项目需求选择，ScriptingBackend 选择IL2CPP

使用Android studio 打开(软件安装等不赘述)

### 原生项目设置
原生项目结构
```yml
launcher:
  manifest:
  res:
    mipmap:
    values:

unityLibrary:
  manifests:
  java:
```

* 需要把launcher/res/minmap 拷贝到 unityLibrary/res 路径下
* 需要把launcher/res/values/string.xml 拷贝到 unityLibrary/res 路径下

注释掉入口代码
```xml
<application android:extractNativeLibs="true">
    <meta-data android:name="unity.splash-mode" android:value="0" />
    <meta-data android:name="unity.splash-enable" android:value="True" />
    <meta-data android:name="unity.launch-fullscreen" android:value="True" />
    <meta-data android:name="notch.config" android:value="portrait|landscape" />
    <meta-data android:name="unity.auto-report-fully-drawn" android:value="true" />
    <activity android:name="com.unity3d.player.UnityPlayerActivity" android:theme="@style/UnityThemeSelector" android:screenOrientation="fullUser" android:launchMode="singleTask" android:configChanges="mcc|mnc|locale|touchscreen|keyboard|keyboardHidden|navigation|orientation|screenLayout|uiMode|screenSize|smallestScreenSize|fontScale|layoutDirection|density" android:resizeableActivity="false" android:hardwareAccelerated="false">
<!--      <intent-filter>-->
<!--        <category android:name="android.intent.category.LAUNCHER" />-->
<!--        <action android:name="android.intent.action.MAIN" />-->
<!--      </intent-filter>-->
      <meta-data android:name="unityplayer.UnityActivity" android:value="true" />
      <meta-data android:name="notch_support" android:value="true" />
    </activity>
```

### Gradle 导出
根据自己项目设置使用Gradle导出AAR包，提供给原生接入