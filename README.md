# copy_apk
执行打包任务，然后拷贝apk到指定目录，节省找apk的时间。

```groovy

//==================================================================================================
//  将此文件放置工程根目录与setting.gradle同级。然后在项目的gradle文件中进行配置
//==================================================================================================
//  以下是在项目的build.gradle中进行的配置
//  apply from: '../apk.gradle'
//  project.ext.APK_NAME = "修改为你想要的apk名字" //apk.gradle使用变量
//
//  android{
//      defaultConfig{...}
//
//      //在defaultConfig之后添加以下内容，用于修改app名字，并增加一些app的信息。APK_NAME在任意时刻都可以修改。
//      project.ext.APK_NAME = "appName_${defaultConfig.versionName}"//apk.gradle使用变量
//
//      signingConfigs {
//          可以没有debug的签名配置，但是必须有release的签名配置
//          debug{
//            keyAlias 'you alias'
//            keyPassword 'your key password'
//            storeFile file('your keystore file')
//            storePassword 'your keystore password'
//          }
//          release {
//              keyAlias 'you alias'
//              keyPassword 'your key password'
//              storeFile file('your keystore file')
//              storePassword 'your keystore password'
//          }
//      }
//      debug {
//          //如果要打包debug的apk的话，就要在debug配置里面添加以下内容
//          signingConfig signingConfigs.release//指定签名配置,这里指定的是release的签名配置,也可以指定debug的签名配置
//      }
//      release {
//          signingConfig signingConfigs.release
//      }
//  }
//  以上是在项目的build.gradle中进行的配置
//==================================================================================================
//配置完成后，在控制台输入 [gradlew APK_DEBUG 或者 APK_RELEASE] 打包。或者在Gradle任务窗口找到 Tasks-other-APK_RELEASE /app-Tasks-other-APK_RELEASE
//==================================================================================================
//打包和拷贝APK任务

tasks.create("APK_RELEASE") {
    //依赖Android的打包任务。当打包任务执行完成后，会生成一个APK文件。再将apk文件复制到指定位置/build/apk/目录下。
    dependsOn('assembleRelease')
    doLast {
        createApk("release")
    }
}


tasks.create("APK_DEBUG") {
    dependsOn('assembleDebug')
    doLast {
        createApk("debug")
    }
}

tasks.create("APK"){
    dependsOn(['APK_DEBUG','APK_RELEASE'])
    setGroup("build")
}
def createApk(String type ) {
    println("=====================APK Copy=====================")
    checkApkName()
    def appName = "${project.ext.APK_NAME}_${type}_${time()}.apk"
    copy {
        //需要知道任务assemble生成的APK文件的位置
        File apkFile = new File("${buildDir.absolutePath}/intermediates/apk/${type}/${model.name}-${type}.apk".replace("/", File.separator))//AGP7.1+位置
        if (!apkFile.exists()) {
            apkFile = new File("${buildDir.absolutePath}/outputs/apk/${type}/${model.name}-${type}.apk".replace("/", File.separator))//AGP7.1以前位置
        }
        if (apkFile.exists()) {
            def outputPath = "${buildDir.absolutePath}${File.separator}apk"
            def inputPath = apkFile.absolutePath

            println "APK File: ${apkFile}"
            println "APK Copy Path${outputPath}"
            println "APK Name: $appName"

            from(inputPath)
            into(outputPath)
            rename("${model.name}-${type}.apk",   appName)

            println "copy apk success !"
        } else {
            println("=====================APK Copy Error: not find apk=====================")
        }
    }
    println("=====================APK Copy end=====================")
}

def checkApkName(){
    try {
        project.ext.APK_NAME
    } catch (Exception e) {
        println("你没有设置APK的名，这里就使用默认名字了")
        project.ext.APK_NAME = "app"
    }
}

//为apk名字添加日期信息
static def time() {
    return new Date().format("MM月dd日HH时mm分", TimeZone.getTimeZone("GMT+08:00"))
}


```
![img](/img.png)


