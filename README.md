此项目是原生项目集成rn:
1,步骤按照rn中文官网:
https://reactnative.cn/docs/integration-with-existing-apps/

2,部分注意事项:
1)network_security_config.xml必须配置,在AndroidManifest.xml中配置android:networkSecurityConfig="@xml/network_security_config"
新增xml文件夹中的文件
<network-security-config>
    <!-- allow cleartext traffic for React Native packager ips in debug -->
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="false">localhost</domain>
        <domain includeSubdomains="false">10.0.2.2</domain>
        <domain includeSubdomains="false">10.0.3.2</domain>
        <domain includeSubdomains="false">10.247.37.122</domain>
    </domain-config>
</network-security-config>

2)增加MainApplication继承ReactApplication.主要是 SoLoader.init(this, /* native exopackage */ false)方法,否则so找不到.

3)app工程下面的build.gradle需要配置.
project.ext.react = [
        entryFile: "index.js",
        enableHermes: false,  // clean and rebuild if changing
]
def jscFlavor = 'org.webkit:android-jsc:+'
def enableHermes = project.ext.react.get("enableHermes", false)
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.core:core-ktx:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    ...
	//主要是下面的配置
    implementation "com.facebook.react:react-native:+" // From node_modules
    implementation 'com.facebook.android:facebook-android-sdk:5.4.0'
    if (enableHermes) {
        def hermesPath = "../../node_modules/hermesvm/android/"
        debugImplementation files(hermesPath + "hermes-debug.aar")
        releaseImplementation files(hermesPath + "hermes-release.aar")
    } else {
        implementation jscFlavor
    }
}

4)assets文件夹需配置,并在工程目录中运行//react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res

5)
  // 注意这里的MyReactNativeApp必须对应“index.js”中的
        // “AppRegistry.registerComponent()”的第一个参数
        mReactRootView?.startReactApplication(mReactInstanceManager, MODULE_NAME, null)

6)工程目录下的build.gradle配置:
allprojects {
    repositories {
       ...
        maven {
            // All of React Native (JS, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
        maven {
            // Android JSC is installed from npm
            url("$rootDir/../node_modules/jsc-android/dist")
        }
    }
}

7)注意warning "react-native@0.52.2" has unmet peer dependency "react@16.2.0".这样的警告.
执行命令:yarn add react@16.2.0

8) package.json 需要写
