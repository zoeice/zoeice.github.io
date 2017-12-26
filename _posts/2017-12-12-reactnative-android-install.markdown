---
layout:     post
title:      "Android/iOS已有项目集成ReactNative"
date:       2017-12-12 16:01:00
author:     "ZoeIce"
header-img: "img/home-bg-o.jpg"
catalog: false
tags:
    - ReactNative
    - android
    - iOS
---

## Android/iOS已有项目集成ReactNative

# 准备工作
ReactNative环境配置。 [ 具体参考[官网文档](https://facebook.github.io/react-native/docs/getting-started.html) ]

#### 1. 推荐使用 [Homebrew](http://brew.sh/) 安装 Node、Watchman
```
$ brew install node
$ brew install watchman
```
安装完 node 后建议设置 npm 镜像以加速后面的过程（或使用科学上网工具）。注意：不要使用 cnpm！cnpm安装的模块路径比较奇怪，packager不能正常识别！
```
$ npm config set registry https://registry.npm.taobao.org --global
$ npm config set disturl https://npm.taobao.org/dist --global
```
如果已经安装了 node，确保在 4 版本或者更高版本（建议使用 5 或者更高版本）

#### 2. Yarn、React Native的命令行工具（The React Native CLI）

[Yarn](http://yarnpkg.com/) 是Facebook 提供的替代 npm 的工具，可以加速node模块的下载。React Native的命令行工具用于执行创建、初始化、更新项目、运行打包服务（packager）等任务。

```
npm install -g yarn react-native-cli
```

安装完 yarn 后同理也要设置镜像源：

```
yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global
```

如果你看到 `EACCES: permission denied` 这样的权限报错，那么请参照上文的 homebrew 译注，修复 `/usr/local` 目录的所有权：

```
sudo chown -R `whoami` /usr/local
```

安装完yarn之后就可以用yarn代替npm了，例如用`yarn`代替`npm install`命令，用`yarn add 某第三方库名`代替`npm install --save 某第三方库名`。

#### 3. iOS环境

Xcode 8.0 或更高版本
安装[CocoaPods](http://zoeice.com/2016/08/18/cocoapods-install/)

#### 4. Android环境
1.下载安装[JDK, 推荐JDK8或更高版本](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
2.下载[AndroidStudio](https://developer.android.com/studio/index.html) 并安装

AndroidStudio会默认安装最新版本的SDK, ReactNative会特别要求`SDK23(Android 6.0 Marshmallow)`

3.配置ANDROID_HOME环境
在`$HOME/.bash_profile`加入如下, 个人的特殊路径设置自己另外指定：
```
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

##### 5. Flow（推荐安装）

[Flow](http://www.flowtype.org/) 是一个静态的JS类型检查工具。译注：你在很多示例中看到的奇奇怪怪的冒号问号，以及方法参数中像类型一样的写法，都是属于这个flow工具的语法。这一语法并不属于ES标准，只是Facebook自家的代码规范。所以新手可以直接跳过（即不需要安装这一工具，也不建议去费力学习flow相关语法）。

```
brew install flow
```

# 开始集成

**1.** 创建MixReactNative目录作为ReactNative项目根目录,  根目录下新建`package.json`，输入如下：

```json
{
  "name": "MixReactNative",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "dependencies": {
    "react": "16.0.0",
    "react-native": "0.51.0"
  },
  "devDependencies": {
  }
}
```
**2.** 在根目录执行`npm install`安装npm相关依赖，会生成node_modules目录
**3.** 在根目录创建`index.js`文件，输入如下：

```javascript
import { AppRegistry } from 'react-native';
import App from './App';

AppRegistry.registerComponent('MixReactNative', () => App);
```
**4.** 在根目录创建`App.js`文件，输入如下：

```javascript
import React, {Component} from 'react';
import {
    Platform,
    StyleSheet,
    Text,
    View
} from 'react-native';

const instructions = Platform.select({
    ios: 'Press Cmd+R to reload,\n' +
    'Cmd+D or shake for dev menu',
    android: 'Double tap R on your keyboard to reload,\n' +
    'Shake or press menu button for dev menu',
});

export default class App extends Component<{}> {
    render() {
        return (
            <View style={styles.container}>
                <Text style={styles.welcome}>
                    ReactNative集成成功！
                </Text>
                <Text style={styles.welcome}>
                    Welcome to React Native!
                </Text>
                <Text style={styles.instructions}>
                    To get started, edit App.js
                </Text>
                <Text style={styles.instructions}>
                    {instructions}
                </Text>
            </View>
        );
    }
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: '#F5FCFF',
    },
    welcome: {
        fontSize: 20,
        textAlign: 'center',
        margin: 10,
    },
    instructions: {
        textAlign: 'center',
        color: '#333333',
        marginBottom: 5,
    },
});
```

# android集成
**1.** 项目根目录下创建`android`文件夹，将已有的Android项目内文件复制到该文件夹下
**2.** 修改Project下的`build.gradle`文件，参考如下：
```python
buildscript {
   ......
allprojects {
    repositories {
        maven {
            // All of React Native (JS, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
      ......
    }
}
```
> 注意：这里的url路径容易出错，需要留意是否是正确路径

**3.** 修改app下的`build.gradle`文件，参考如下：

```python
apply plugin: 'com.android.application'

android {
    compileSdkVersion 26
    buildToolsVersion "26.0.2"

    defaultConfig {
        ...
        ndk {
            abiFilters "armeabi-v7a", "x86"
        }
    }
    splits {
        abi {
            reset()
            enable false
            universalApk false  // If true, also generate a universal APK
            include "armeabi-v7a", "x86"
        }
    }
}

dependencies {
   ...
    implementation 'com.android.support:appcompat-v7:26.1.0'
    implementation "com.facebook.react:react-native:+" // From node_modules.
}
```
**4.** 新建`MainApplication.java`文件继承`Application`并实现`ReactAppliaction`接口，输入如下:

```java
public class MainApplication extends Application implements ReactApplication {

    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
        @Override
        public boolean getUseDeveloperSupport() {
            return BuildConfig.DEBUG;
        }

        @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage()
            );
        }

        @Override
        protected String getJSMainModuleName() {
            return "index";
        }
    };

    @Override
    public ReactNativeHost getReactNativeHost() {
        return mReactNativeHost;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        SoLoader.init(this, /* native exopackage */ false);
    }
}
```
**5.** 在AndroidManifest.xml中加上配置

```xml
<application
        android:name=".MainApplication"
        ...
        >
        </application>

<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>

```

**6.** 新建一个继承`ReactActivity.java`的类，参考如下：

```java
public class ReactNativeActivity extends ReactActivity {

    /**
     * Returns the name of the main component registered from JavaScript.
     * This is used to schedule rendering of the component.
     */
    @Override
    protected String getMainComponentName() {
        return "MixReactNative";
    }
}
```
**7.** 记得在AndroidManifest.xml文件中加入该Activity, 加入ReactNative用于调试开发的Activitty
```xml
<activity android:name=".ReactNativeActivity"/>
<activity android:name="com.facebook.react.devsupport.DevSettingsActivity"/>
```

**8.** 在原生页面中编写跳转到`ReactNativeActivity`类的代码用于测试原生跳转RN页面是否正常，参考如下：

```java
findViewById(R.id.action_rn).setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            startActivity(new Intent(MainActivity.this, ReactNativeActivity.class));
        }
});
```
# IOS集成
#### 1. 配置CocoaPods的依赖
React Native 框架整体是作为 node 模块安装到项目中的。下一步我们需要在 CocoaPods 的 Podfile 中指定我们所需要使用的组件。

在你开始把 React Native 集成到你的应用中之前，首先要决定具体整合的是 React Native 框架中的哪些部分。而这就是`subspec`要做的工作。在创建`Podfile`文件的时候，需要指定具体安装哪些React Native的依赖库。所指定的每一个库就称为一个`subspec`。

可用的`subspec`都列在[`node_modules/react-native/React.podspec`](https://github.com/facebook/react-native/blob/master/React.podspec)中，基本都是按其功能命名的。一般来说你首先需要添加`Core`，这一`subspec`包含了必须的`AppRegistry`、`StyleSheet`、`View`以及其他的一些React Native核心库。如果你想使用React Native的`Text`库（即`<Text>`组件），那就需要添加`RCTText`的`subspec`。同理，`Image`需要加入`RCTImage`，等等。

创建 Podfile
在Podfile文件里添加以下内容:
```
target 'RNOCDemo' do
  # Uncomment the next line if you're using Swift or would like to use dynamic frameworks
  # use_frameworks!

  # Pods for RNOCDemo

  # 'node_modules'目录一般位于根目录中
  # 但是如果你的结构不同，那你就要根据实际路径修改下面的`:path`
  pod 'React', :path => '../node_modules/react-native', :subspecs => [
        'Core',
        'CxxBridge', # 如果RN版本 >= 0.45则加入此行
        'RCTText',
        'RCTNetwork',
        'RCTWebSocket', # 这个模块是用于调试功能的
        # 在这里继续添加你所需要的RN模块
        ]

   # 如果你的RN版本 >= 0.42.0，则加入下面这行
   pod "yoga", :path => "../node_modules/react-native/ReactCommon/yoga"
   # 如果RN版本 >= 0.45则加入下面三个第三方编译依赖
   pod 'DoubleConversion', :podspec => '../node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'
   pod 'GLog', :podspec => '../node_modules/react-native/third-party-podspecs/GLog.podspec'
   pod 'Folly', :podspec => '../node_modules/react-native/third-party-podspecs/Folly.podspec'

end
```
创建好之后，执行安装：
```
pod install
```

#### 2. 配置info.plist
还需要在`info.plist`文件中配置一下https的忽略：

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>localhost</key>
        <dict>
            <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
```
#### 3. 代码集成
```
#import <React/RCTRootView.h>
```
跳转到RN页面
```
- (IBAction)go2RNPage:(id)sender {
    NSURL *jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.bundle?platform=ios"];
    RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL: jsCodeLocation
                                                        moduleName: @"MixReactNative"
                                                 initialProperties: nil
                                                     launchOptions: nil];
    UIViewController *vc = [[UIViewController alloc] init];
    vc.view = rootView;
    [self.navigationController pushViewController:vc animated:YES];
}
```

# 运行服务

至此代码集成基本就绪，项目根目录下执行`react-native start`开启服务，然后在`Android Studio`中连接设备并安装运行app，点击跳转到对应的RN页面后通过摇晃手机进入开发调试页，配置同一网络环境下的ip和端口号，然后重新加载页面，不出意外的话就会成功看到RN页面成功加载。
