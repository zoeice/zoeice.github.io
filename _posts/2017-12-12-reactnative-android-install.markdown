---
layout:     post
title:      "android项目中加入ReactNative的方法"
date:       2017-12-12 16:01:00
author:     "ZoeIce"
header-img: "img/post-bg-android.jpg"
catalog: false
tags:
    - ReactNative
    - android
---

## 开发环境准备
配置android环境
1. 配置环境变量：
在.bash_profile中配置, ANDROID_HOME根据你自己android sdk 的路径修改
```
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools
```


## 安装JavaScript依赖包

### 1.安装JavaScript依赖包
在项目根目录下创建一个名为package.json的空文本文件，然后填入以下内容
```json
{
  "name": "MyReactNativeApp",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  }
}
```


#### 2.添加react和react-native 模块
命令控制台输入    
```Shell   
$ npm install --save react react-native
```
这些模块会被安装到项目根目录下的node_modules/目录中（这个目录我们原则上不复制、不移动、不修改、不上传，随用随装）

## 把React Native添加到你的应用中

#### 1.添加ReactNative相关依赖
在app的`build.gradle`文件中添加react-native依赖库
```groovy    
android {
    buildToolsVersion "23.0.1"
    ...
    defaultConfig {
        ...
        ndk {
            abiFilters "armeabi-v7a", "x86"
        }
    }
    splits {
        abi {
            reset()
            universalApk false  // If true, also generate a universal APK
            include "armeabi-v7a", "x86"
        }
    }

}
dependencies {
    ...
    compile "com.facebook.react:react-native:+" // From node_modules.
    compile 'com.android.support:appcompat-v7:23.0.1'
}
```
在project的`build.gradle`文件中添加添加一个 maven 依赖的入口
```groovy    
allprojects {
    repositories {
        ...
        maven {
            // All of React Native (JS, Android binaries) is installed from npm
            url "$rootDir/node_modules/react-native/android"
        }
    }
}
```

#### 2.配置相关权限
在 `AndroidManifest.xml` 清单文件中声明网络权限:
```xml    
<uses-permission android:name="android.permission.INTERNET" />
```


#### 3.添加index.js文件到项目中

###### (1).创建一个index.js文件

在项目根目录创建空的index.js文件(注意在react native 0.49版本之前是index.android.js文件)
>index.js是React Native应用在Android上的入口文件

###### (2).添加React Native代码
这里我们只是简单的添加一个<Text>组件，然后用一个带有样式的<View>组件把它包起来。如下添加自己的代码。
```java
import React from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View
} from 'react-native';

class HelloWorld extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.hello}>Hello, World</Text>
      </View>
    )
  }
}
var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  hello: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
});

AppRegistry.registerComponent('MyReactNativeApp', () => HelloWorld);
```
这里要注意，`AppRegistry.registerComponent(MyReactNativeApp, () => HelloWorld);`
里面的名称 必须和你的工程名一致。比如MyReactNativeApp。


## 代码集成
新建一个Activity，让其继承ReactActivity，并重写getMainComponentName(),返回我们在index.android.js
中注册的HelloWorld这个组件。
```java
public class MyReactActivity extends ReactActivity {

    @Override
    protected String getMainComponentName() {
        return "MyReactNativeApp";
    }
}
```

我们需要把 MyReactActivity 的主题设定为 Theme.AppCompat.Light.NoActionBar ，因为里面有许多组件都使用了这一主题。

```xml
<activity
  android:name=".MyReactActivity"
  android:label="@string/app_name"
  android:theme="@style/Theme.AppCompat.Light.NoActionBar">
</activity>
```
自定义一个Application，继承ReactApplication ，编写以下代码：
```java
public class App extends Application implements ReactApplication {

    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
        @Override
        public boolean getUseDeveloperSupport() {
            return BuildConfig.DEBUG;
            //return true;
        }

        @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage()
            );
        }
    };
    @Override
    public ReactNativeHost getReactNativeHost() {
        return mReactNativeHost;
    }
}
```
记得在AndroidManifest.xml中引用
```xml
<application
  android:name=".App"
  ...>
</application>
```

在MainActivity中通过按钮启动我们的ReactNativeActivity
```java
findViewById(R.id.action_rn).setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            startActivity(new Intent(MainActivity.this, MyReactActivity.class));
        }
});
```
下一步，app/src/main下新建assets文件夹。
运行如下命令
```
react-native start
```
如果控制台显示：`Loading dependency graph, done.`, 则在根目录下重新运行一个命令行并在其中输入如下命令
```Shell
react-native bundle --platform android --dev false --entry-file index.js --bundle-output app/src/main/assets/index.android.bundle --assets-dest app/src/main/res/
```

完成之后assets目录下会生成以下两个文件
- index.android.bundle
- index.android.bundle.meta

确认一下react native service处于运行状态，然后正常运行你的APP，点击start

运行：
```
react-native run-android
```
如果报这个错误：`Android project not found. Maybe run react-native android first?`
则运行如下命令：
```
react-native upgrade
```
在出现询问的提示下输入`y`，最后看到`Successfully upgraded this project to react-native v0.51.0`则表示更新完成

重新输入`react-native run-android`
