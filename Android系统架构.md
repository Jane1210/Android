## 第一章 Android系统架构+项目结构+日志

### 一 Android系统架构

#### 1.Linux内核层

为Android设备的各种硬件提供了底层的驱动，如显示驱动、照相机驱动、电源驱动等。

#### 2.系统运行层库

通过一些C/C++库来为Android系统提供了主要的特性支持。如SQlite库提供了数据库的支持、Webkit提供浏览器内核支持等；

这一层还有Android运行时库，它主要提供了一些核心库，能够允许开发者使用Java来编写Android应用。

#### 3.应用框架层

这一层主要提供了构建应用程序时可能用到的各种API。

#### 4.应用层

所有安装在手机上的应用程序都是属于这一层的。

### 二 Android应用开发特色

#### 1.四大组件

活动（Activity）：Android应用程序的门面

服务（Service）：后台运行

广播接收器（BroadcastReceiver）：允许应用接收来自各处的广播信息，如电话、消息等；或发出广播消息

内容提供器（ContentProvider）：为应用程序之间共享数据提供了可能，比如要读取电话簿联系人，得由它实现

#### 2.丰富的系统控件

#### 3.SQLite数据库

轻量级、运算速度极快的嵌入式关系型数据库。

#### 4.强大的多媒体

#### 5.地理位置定位

### 三 创建Android项目

#### 1.project模式的项目结构

.gradle 和 .idea ：这两个目录下放置的是Android studio 自动生成的一些文件，无需关心，不用手动编辑。

.app：项目中的代码、资源等内容几乎都是放置在这个目录，开发工作基本都是在这个目录。

.build：无需关心，它主要包含了一些在编译时自动生成的文件。

.gradle: 包含了gradle wrapper的配置文件。

.gitignore: 用来将指定的目录或文件排除在版本控制之外。

build.gradle: 全局的gradle配置文件，在这里配置的属性将会影响到项目中所有的gradle编译脚本。

gradlew 和 gradlew.bat: 用来在命令行界面中执行gradle命令，前者是在Linux或Mac系统使用，后者Windows。

.iml: 用于标识这是一个IntelliJIDEA项目，无需修改。

local.properties: 指定本机中的Andriod SDK路径，自动生成。无需修改，除非本机中Andriod SDK位置变化。

settings.gradle: 用于指定项目中所有引入的模块，如app模块。通常自动完成，无需修改。

##### app目录：

1.build

与外层build目录类似，包含一些在编译时自动生成的文件，不过内容更复杂。无需太多关心。

2.libs

放置第三方jar包。

3.androidTest

用来编写AndroidTest测试用例。

**4.java**

放置所有java代码。

**5.res**

图片放在drawable目录下，布局放在layout下，字符串放在values下等等。

**6.AndroidManifest.xml**

整个Android项目的配置文件，定义的所有四大组件需要在这里注册，还可以在这里给应用程序添加权限声明。

7.test

编写Unit Test测试用例。

8..gitignore

用来将app模块内的指定的目录或文件排除在版本控制之外，作用和外层的类似。

9.app.iml

**10.build.gradle**

app模块的gradle构建脚本，会指定很多项目构建相关的配置。

11.proguard-rules.pro

用于指定项目代码的混淆规则，防止安装包文件被人破解。

#### 2.详解build.gradle文件

**外层：**

通常不需要修改，除非要添加全局的项目构建配置。

```java
buildscript {
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.3'
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

jcenter()：jcenter是个代码托管仓库，声明这行配置后，就可以在项目中引用任何jcenter上的开源项目了。

classpath：声明gradle插件，版本号。

**app目录下的build.gradle文件：**

```java
apply plugin: 'com.android.application'

android {
    compileSdkVersion 28
    defaultConfig {
        applicationId "com.vivo.lijianzhen.myapplication1"
        minSdkVersion 15
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:28.0.0-alpha3'
    implementation 'com.android.support.constraint:constraint-layout:1.1.2'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
}
```

- apply plugin: 插件，一般有两种值可选，com.android.application表示是一个应用程序模块，com.android.library表示是一个库模块。区别在于，一个是可以直接运行的，一个只能作为代码库依附于别的应用程序模块来运行。

- android闭包：

  compileSdkVersion：用于指定项目的编译版本。

  *defaultConfig闭包：细节配置

  applicationId：项目包名，要修改的话在这里修改

  minSdkVersion：用于指定项目最低兼容的Android系统版本，15表示Android4.0系统

  targetSdkVersion：指定的值表示你在该目标版本上已经做过了充分的测试，系统将会为你的应用程序启用一些最新的功能和特性

  versionCode：项目版本号（在生成安装文件时非常重要）

  versionName：项目版本名（在生成安装文件时非常重要）

  *buildTypes闭包：指定生成安装文件的相关配置

  release闭包：指定生成生成正式版安装文件的配置

  minifyEnabled：指定是否对项目的代码进行混淆，false是不混淆

  proguardFiles：指定混淆时使用的规则文件，proguard-android.txt是在Android SDK目录下的，proguard-rules.pro是在当前项目的根目录下的

  通过Android studio直接运行项目生成的都是测试版安装文件。

- dependencies闭包：指定当前项目所有的依赖关系。通常有3种，本地依赖、库依赖、远程依赖。

  本地依赖：可以对本地的jar包或目录添加依赖关系

  库依赖：可以对项目中的库模块添加依赖关系

  远程依赖：可以对jcenter库上的开源项目添加依赖关系

  fileTree：本地依赖声明

  com.android.support:appcompat-v7:28.0.0-alpha3：远程依赖声明，com.android.support是域名部分，appcompat-v7是组名，28.0.0-alpha3是版本号

  testImplementation：声明测试用例库

### 四 掌握日志工具的使用

日志工具类Log(android.util.log)，有以下5个方法用来打印日志：

- [ ] Log.v()

级别最低，用于打印最为琐碎的、意义最小的日志信息，对应级别verbose。

- [ ] Log.d()

打印一些调试信息，对应debug。

- [ ] Log.i()

打印比较重要的数据，可以帮助分析用户行为。对应info。

- [ ] Log.w()

打印警告信息，对应warn。

- [ ] Log.e()

打印程序中的错误信息，对应error，级别最高。

方法外面输入logt  tab键，自动以类名作为值生成一个TAG常量。

自定义过滤器：

Edit Filter Configration               支持正则



















