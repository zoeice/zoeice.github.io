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
随便起个项目名，比如：NativeAppDemo

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
    compile 'com.android.support:appcompat-v7:26.1.0'
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
#### 3.代码集成


#### 4.添加index.js文件到项目中

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


## 创建Activity
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


现在activity已就绪，可以运行一些JavaScript代码了。

## 测试集成结果
You have now done all the basic steps to integrate React Native with your current application. Now we will start the React Native packager to build the index.bundle package and the server running on localhost to serve it.

#### 1. 运行Packager
运行应用首先需要启动开发服务器（Packager）。你只需在项目根目录中执行以下命令即可：
```Shell   
$ npm start    
```
#### 2. 运行你的应用
保持packager的窗口运行不要关闭，然后像往常一样编译运行你的Android应用(在命令行中执行./gradlew installDebug或是在Android Studio中编译运行)。

>如果你是使用Android Studio来编译运行，有可能会导致packager报错退出。这种情况下你需要安装watchman。但是watchman目前没有稳定的Windows版本，所以在Windows下这种崩溃情况暂时没有特别好的解决方案。

编译执行一切顺利进行之后，在进入到MyReactActivity时应该就能立刻从packager中读取JavaScript代码并执行和显示

## 在Android Studio中打包
你也可以使用Android Studio来打release包！其步骤基本和原生应用一样，只是在每次编译打包之前需要先执行js文件的打包(即生成离线的jsbundle文件)。具体的js打包命令如下：

```Shell   
$ react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/com/your-company-name/app-package-name/src/main/assets/index.android.bundle --assets-dest android/com/your-company-name/app-package-name/src/main/res/
```

注意把上述命令中的路径替换为你实际项目的路径。如果assets目录不存在，则需要提前自己创建一个。

然后在Android Studio中正常生成release版本即可！
