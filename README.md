# HelloReact
React native嵌入到原生Android应用中混合使用

在Windows系统下Android Studio开发环境下原生App中嵌入React Native的详细说明文档。
前提条件，React Native开发环境搭建完成。
具体步骤如下：
1、新建工程目录，本例中工程目录为HelloReact，然后在该目录下新建子目录android。
2、在HelloReact\android目录下新建Android原生代码工程，本例中以一个空Activity为例，新建工程后，在该页面添加一个按钮，点击跳转到新页面，而这个新页面也就是我们今天需要创建的React native实现的。
3、命令行打开到HelloReact目录，运行以下命令：
```
  ①  $ npm init
  ②  $ npm install --save react react-native
  ③  $ curl -o .flowconfig https://raw.githubusercontent.com/facebook/react-native/master/.flowconfig
```
  其中，第三步如果没有安装curl可以直接从该链接下载.flowconfig文件或者从其他React Native项目中复制到当前目录下。
4、打开package.json文件，在scripts标签下添加npm依赖：
  "start": "node node_modules/react-native/local-cli/cli.js start"
  添加完毕后如下：
  ```
  {
  "name": "helloreact",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "author": "rain",
  "license": "ISC",
  "dependencies": {
    "react": "^15.4.2",
    "react-native": "^0.40.0"
  }
}
  ```
5、创建index.android.js文件，将以下代码复制到该文件中：
```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

'use strict';

import React from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View
} from 'react-native';

class HelloReact extends React.Component {
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

AppRegistry.registerComponent('helloreact', () => HelloReact);

```

6、此时HelloReact根目录下的目录结构创建完毕，如下：
![](https://github.com/sereinli/HelloReact/blob/master/android/pictures/project%20root.png)
![](https://github.com/sereinli/HelloReact/blob/master/android/pictures/project%20android%20root.png)

7、Android原生代码project下build.gradle添加React Native本地maven目录：
```
allprojects {
    repositories {
        ...
        maven {
            // All of React Native (JS, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
    }
    ...
}
```

8、修改app module下build.gradle
```
android {
  buildToolsVersion "23.0.1"
  minSdkVersion 16
  
  //为了解决一个冲突问题
  configurations.all {
        resolutionStrategy.force 'com.google.code.findbugs:jsr305:1.3.9'
    }
 }
 
 dependencies {
   compile 'com.android.support:appcompat-v7:23.0.1'  //必须是23.0.1
 
   compile "com.facebook.react:react-native:+" // From node_modules.
 }
 ```
 
 9、新建从主界面跳转到的React Native页面（ReactActivity），代码如下：
 ```
 package me.rain.android.myapplication;

import android.app.Activity;
import android.os.Bundle;
import android.view.KeyEvent;

import com.facebook.react.ReactInstanceManager;
import com.facebook.react.ReactRootView;
import com.facebook.react.common.LifecycleState;
import com.facebook.react.modules.core.DefaultHardwareBackBtnHandler;
import com.facebook.react.shell.MainReactPackage;

/**
 * Created by CBS on 2017/1/25.
 */

public class ReactActivity extends Activity implements DefaultHardwareBackBtnHandler {
    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                                    .setApplication(getApplication())
                                    .setBundleAssetName("index.android.bundle")
                                    .setJSMainModuleName("index.android")
                                    .addPackage(new MainReactPackage())
                                    .setUseDeveloperSupport(BuildConfig.DEBUG)
                                    .setInitialLifecycleState(LifecycleState.BEFORE_RESUME)
                                    .build();

        mReactRootView.startReactApplication(mReactInstanceManager, "helloreact", null);

        setContentView(mReactRootView);
    }

    @Override
    protected void onPause() {
        super.onPause();

        if(mReactInstanceManager != null) {
            mReactInstanceManager.onHostPause();
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        if(mReactInstanceManager != null) {
            mReactInstanceManager.onHostResume(this, this);
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if(mReactInstanceManager != null) {
            mReactInstanceManager.onHostDestroy();
        }
    }

    @Override
    public void invokeDefaultOnBackPressed() {
        if(mReactInstanceManager != null) {
            mReactInstanceManager.onBackPressed();
        }else {
            super.onBackPressed();
        }
    }

    @Override
    public boolean onKeyUp(int keyCode, KeyEvent event) {
        if(keyCode == KeyEvent.KEYCODE_MENU && mReactInstanceManager != null) {
            mReactInstanceManager.showDevOptionsDialog();
            return true;
        }
        return super.onKeyUp(keyCode, event);
    }
}

 ```
 10、AndroidManifest.xml中添加Activity声明及权限声明
 ```
 <uses-permission android:name="android.permission.INTERNET"/>
 
 <activity android:name=".ReactActivity"
            android:theme="@style/Theme.AppCompat.Light.NoActionBar"/>
        <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
 ```
  
 #到这里就可以在模拟器或者机器上跑起来了，进行相关调试。记住需要在dev setting中设置本机的IP地址和端口，例如，PC的地址为 10.23.18.34，那么就设置为10.23.18.34：8081
 需要先运行命令 ：$ npm start
 
 最后，如果需要打包，通过android studio可以方便的打包，Build -> Generate Signed APK。但是在此之前，需要先生成index.android.bundle.
 运行以下命令：
 ```
 $ react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res/
 ```
 命令中的assets目录和res目录的路径需要根据工程实际的路径进行修改，另外，运行此命令前，确保assets目录已经被创建。
 生成成功后，会多出以下几个文件：
 ![](https://github.com/sereinli/HelloReact/blob/master/android/pictures/create%20react%20native%20bundle.png)
 
