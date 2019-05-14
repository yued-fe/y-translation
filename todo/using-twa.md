> 原文：[https://developers.google.com/web/updates/2019/02/using-twa](https://developers.google.com/web/updates/2019/02/using-twa)

**Trusted Web Activities** 是集成 Web 应用的新方法，你可以通过基于 Custom Tabs 的协议将 PWA 应用和 Android app 进行集成。

代码参考：

- [TrustedWebUtils Android Support Library API reference](https://developer.android.com/reference/android/support/customtabs/TrustedWebUtils.html)
- [Sample Trusted Web Activity application](https://github.com/GoogleChromeLabs/svgomg-twa)

Trusted Web Activities 和其他一些 Web 与 APP 集成的方式有所不同：

1. Trusted Web Activities 中的内容是**受信任的** - APP 及其打开的网站来自同一个开发者。（这是通过 [Digital Asset Links](https://developers.google.com/digital-asset-links/v1/getting-started) 来验证的。）
2. Trusted Web Activities 来自 **Web**：它们由用户的浏览器渲染，这与用户在浏览器中看到的东西完全相同，不过 TWA 可以全屏运行。Web 内容应该首先保证在浏览器中的可用性。
3. Chrome 浏览器不依赖于 Android 和你的 APP 进行更新，它在 Android Jelly Bean 也可以使用。这可以减小 APK 包的大小，并确保你可以使用现代的 Web 运行环境。（从 Android Lollipop 开始，WebView 也可以独立于 Android 进行了更新，但是有大量的用户使用比 Lollipop 更老的版本。）
4. APP 无法直接访问 Trusted Web activity 中的 Web 内容或其他 Web 状态，比如 cookie 和 localStorage 。不过，你可以通过在 URL 中传递数据（比如通过 query parameters，自定义 HTTP 头和  [intent URIs](https://developer.chrome.com/multidevice/android/intents) 。
5. Web 和 Native 之间的跳转是在 **Activities** 之间进行的。APP 的每个 Activity（即页面）要么由 Web 提供，要么由 Android Activity 提供。

为了便于测试，目前在 Trusted Web activities 中对打开的内容没有要求。但是，Trusted Web activities 可能也需要 [Add to Home Screen](https://developers.google.com/web/fundamentals/app-install-banners/#criteria) 权限。您可以使用 [Lighthouse](https://developers.google.com/web/tools/lighthouse/)  的 "*user can be prompted to Add to Home screen*" 审查来审核这些网站权限 。

如果用户的 Chrome 版本不支持 Trusted Web activities，Chrome 将展示基于 Custom Tab 的简单工具栏。其他浏览器也可以实现 Trusted Web activities 协议。虽然 APP 可以决定打开哪种浏览器，但我们建议使用与 Custom Tabs 相同的策略：使用用户的默认浏览器，只要该浏览器提供所需的功能即可。

## 入门

设置 Trusted Web Activity（TWA）不要求开发人员编写 Java 代码，但需要 [Android Studio](https://developer.android.com/studio/)。本指南基于 Android Studio 3.3。查看 [安装文档](https://developer.android.com/studio/install)。

### 创建 Trusted Web Activity 项目

使用 Trusted Web Activities 时，项目必须为 API 16 或更高版本。

打开 Android Studio，然后点击 *Start a new Android Studio project*。

Android Studio 将提示您选择 Activity 类型。由于 TWA 使用 support 库提供的 Activity，因此选择 *Add No Activity* 并点击 *Next*。

下一步，向导将提示项目的配置。以下是每个字段的简短描述：

- **Name:**  *Android桌面* 上的应用程序名称 。
- **Package Name:** Play 商店和 Android 设备上 Android 应用程序的唯一标识符。 有关为 Android 应用程序创建程序包名称的要求和最佳实践的请查看 [文档](https://developer.android.com/guide/topics/manifest/manifest-element#package)。
- **Save location:** Android Studio 将在电脑中创建项目的目录。
- **Language:** 该项目不需要编写任何 Java 或 Kotlin 代码。选择 Java 作为默认值。
- **Minimum API Level：** support 库至少需要 API 16。选择 API 16 以上的版本。

忽略其他选项，然后点击 *Finish*。

### 获取 TWA Support Library

要在项目中设置 TWA 库，您需要编辑几个文件。在 *Project Navigator* 查找 *Gradle Scripts* 部分。

第一个文件是 **Project** 级 `build.gradle`。

添加 [Jitpack](https://jitpack.io/) 配置到 `allprojects` 中：

```
allprojects {
   repositories {
       google()
       jcenter()
       maven { url "https://jitpack.io" }
   }
}
```

Android Studio 将提示 Sync 项目。点击 *Sync Now* 。

我们需要更改的第二个文件是 **Module** 级的 `build.gradle`。

Trusted Web Activities 库使用 [Java 8功能](https://developer.android.com/studio/write/java8-support) ，首先启用 Java 8。在 `android` 下添加 `compileOptions` 配置 ，如下所示：

```
android {
        ...
    compileOptions {
       sourceCompatibility JavaVersion.VERSION_1_8
       targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

下一步将 TWA Support 库添加到项目中。向 `dependencies` 添加新的依赖：

```
dependencies {
   implementation 'com.github.GoogleChrome.custom-tabs-client:customtabs:d08e93fce3'
}
```

点击 *Sync Now* 来同步项目。

### 添加 TWA Activity

通过编辑 [Android App Manifest](https://developer.android.com/guide/topics/manifest/manifest-intro) 来实现设置TWA活动 。

在*Project Navigator上*，展开 *app* 部分，然后展开 *manifests* 并双击 `AndroidManifest.xml ` 打开文件。

因为我们在创建项目时没有添加任何 Activity，因此 `manifest` 为空且仅包含 `application` 标记。

通过在 `application` 标记中插入 `activity` 标记来 添加 TWA Activity：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.twa.myapplication">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        tools:ignore="GoogleAppIndexingWarning">
        <activity
            android:name="android.support.customtabs.trusted.LauncherActivity">

           <!-- Edit android:value to change the url opened by the TWA -->
           <meta-data
               android:name="android.support.customtabs.trusted.DEFAULT_URL"
               android:value="https://airhorner.com" />

           <!-- This intent-filter adds the TWA to the Android Launcher -->
           <intent-filter>
               <action android:name="android.intent.action.MAIN" />
               <category android:name="android.intent.category.LAUNCHER" />
           </intent-filter>

           <!--
             This intent-filter allows the TWA to handle Intents to open
             airhorner.com.
           -->
           <intent-filter>
               <action android:name="android.intent.action.VIEW"/>
               <category android:name="android.intent.category.DEFAULT" />
               <category android:name="android.intent.category.BROWSABLE"/>

               <!-- Edit android:host to handle links to the target URL-->
               <data
                 android:scheme="https"
                 android:host="airhorner.com"/>
           </intent-filter>
        </activity>
    </application>
</manifest>
```

添加到 XML 的标记是标准的 [Android App Manifest](https://developer.android.com/guide/topics/manifest/manifest-intro)。Trusted Web Activities 有两个相关信息：

1. `meta-data` 标记告诉 TWA 活动打开哪个 URL 。修改 `android:value` 为你要打开的 PWA 页面的 URL。示例为 `https://airhorner.com`。
2. 在第二个 `intent-filter` 标签允许 TWA 拦截 Android Intents 打开 `https://airhorner.com`。该 `android:host` 内部属性 `data` 标签必须指向被 TWA 打开的 网址。

下一节将介绍如何设置 [Digital AssetLinks](https://developers.google.com/digital-asset-links/v1/getting-started) 以验证网站与应用之间的关系，并移除 URL 栏。

### 移除 URL 栏

Trusted Web Activities 需要在 Android 应用程序和网站之间建立关联以删除 URL 栏。

此关联是通过 [Digital Asset Links](https://developers.google.com/digital-asset-links/v1/getting-started) 创建的， 必须用两种方式建立关联：[从 APP 链接到网站](https://developers.google.com/digital-asset-links/v1/create-statement) ， [从网站链接到 APP](https://developer.android.com/training/app-links/verify-site-associations#web-assoc)。

也可以将 APP 设置为网站验证，并设置 Chrome 以跳过网站到 APP 验证，以进行调试。

#### 建立从 APP 到网站的关联

打开 string 资源文件`app > res > values > strings.xml`并在下面添加 Digital AssetLinks 语句：

```xml
<resources>
    <string name="app_name">AirHorner TWA</string>
    <string name="asset_statements">
        [{
            \"relation\": [\"delegate_permission/common.handle_all_urls\"],
            \"target\": {
                \"namespace\": \"web\",
                \"site\": \"https://airhorner.com\"}
        }]
    </string>
</resources>
```

更改 `site` 属性的内容以匹配 TWA 打开的 scheme 和 domain。

回到 `AndroidManifest.xml` 文件，通过在  `application` 标记 添加新 `meta-data` 标记链接到该 statements ：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.twa.myapplication">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <meta-data
            android:name="asset_statements"
            android:resource="@string/asset_statements" />

        <activity>
            ...
        </activity>

    </application>
</manifest>
```

我们现在已经建立了从 Android APP 到网站的关联。我们可以不创建网站到 APP 的验证来调试。

以下是在开发设备上测试它的方法：

#### 启用调试模式

1. 在开发设备上打开 Chrome，导航到 `chrome://flags`，在非root设备上搜索名为 *Enable command line* 的项目，然后将其更改为 **ENABLED**，然后重新启动浏览器。
2. 接下来，在终端上，使用 [Android Debug Bridge](https://developer.android.com/studio/command-line/adb) （随着 Android Studio一起安装），并运行以下命令：

```
adb shell "echo '_ --disable-digital-asset-link-verification-for-url=\"https://airhorner.com\"' > /data/local/tmp/chrome-command-line"
```

关闭 Chrome 并从 Android Studio 重新启动您的应用程序。现在应该以全屏显示应用程序。

### 建立从网站到APP的关联

开发人员需要从 APP 拿到2条信息才能创建关联：

- **Package Name:**  第一个信息是 APP 的包名称。这与创建 APP 时生成的包名称相同。可以在 *Gradle Scripts > build.gradle (Module: app)* 中找到 `applicationId` 的值。
- **SHA-256 Fingerprint:** 必须签名 Android 应用程序才能上传到 Play 商店。相同的签名用于通过上传密钥的 SHA-256 在网站和 APP 之间建立连接。

Android文档 [详细说明了如何使用Android Studio生成密钥](https://developer.android.com/studio/publish/app-signing#generate-key)。请记下密钥的*路径*，*别名* 和*密码*，因为下一步需要用到。

使用 [keytool](https://docs.oracle.com/javase/6/docs/technotes/tools/windows/keytool.html) 提取 SHA-256，使用以下命令：

```
keytool -list -v -keystore  -alias  -storepass  -keypass 
```

 *SHA-256* 打印在 *Certificate* 中。下面是个示例输出：

```
keytool -list -v -keystore ./mykeystore.ks -alias test -storepass password -keypass password

Alias name: key0
Creation date: 28 Jan 2019
Entry type: PrivateKeyEntry
Certificate chain length: 1
Certificate[1]:
Owner: CN=Test Test, OU=Test, O=Test, L=London, ST=London, C=GB
Issuer: CN=Test Test, OU=Test, O=Test, L=London, ST=London, C=GB
Serial number: ea67d3d
Valid from: Mon Jan 28 14:58:00 GMT 2019 until: Fri Jan 22 14:58:00 GMT 2044
Certificate fingerprints:
     SHA1: 38:03:D6:95:91:7C:9C:EE:4A:A0:58:43:A7:43:A5:D2:76:52:EF:9B
     SHA256: F5:08:9F:8A:D4:C8:4A:15:6D:0A:B1:3F:61:96:BE:C7:87:8C:DE:05:59:92:B2:A3:2D:05:05:A5:62:A5:2F:34
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3
```

得到这两条信息后，请打开 [assetlinks生成器](https://developers.google.com/digital-asset-links/tools/generator)，填写字段并点击 *Generate Statement*。复制生成的语句，并设置到你网站的 `/.well-known/assetlinks.json` 目录中。

### 打包

`assetlinks` 存在您的网站中，`asset_statements`在 Android APP 中配置完成后，下一步就是生成签名的 APP。请查看 [文档](https://developer.android.com/studio/publish/app-signing#sign-apk)。

可以使用 adb 将 APK 安装到测试设备中：

```
adb install app-release.apk
```

如果验证失败，则可以使用 adb 从连接设备并电脑终端查看错误消息。

```
adb logcat | grep -e OriginVerifier -e digital_asset_links
```

生成上传 APK 后，就 [可以将应用上传到 Play 商店](https://developer.android.com/studio/publish/upload-bundle) 了。
