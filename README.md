# copy_apk

执行打包任务，然后拷贝apk到指定目录，节省找apk的时间。

```groovy

/**
 *==================================================================================================
 *  将此文件放置工程根目录与setting.gradle同级。然后在项目的gradle文件中进行配置
 *==================================================================================================
 * app模块build.gradle
 *
 * apply from: '../apk.gradle'
 * project.ext.app_name = "我是app名字"
 *
 * android{ ...
 *
 *
 * 修改app产物apk的名字并，修改输出文件位置
 *==================================================================================================
 *配置完成后，在控制台输入 [gradlew assembleRelease ] 打包。
 *==================================================================================================
 */

project.ext.app_name = "app"
android {
    applicationVariants.all { variant ->
        variant.outputs.all { output ->
            def appName = project.ext.app_name // 应用名
            def flavorName = variant.flavorName // 渠道名
            def buildType = variant.buildType.name // 构建类型
            def versionName = variant.versionName // 版本号

            output.outputFileName = "${appName}_${buildType}_${flavorName}_v${versionName}_${time()}.apk"
        }

        // 打包完成之后，将apk复制到指定目录
        File outputFile = new File("${rootDir.absolutePath}/apk/${variant.buildType.name}")
        //  `assembleProvider` 代替 `variant.assemble` AGP-9.0Api, AGP 9.0以前版还是使用`variant.assemble`
        def assembleTaskProvider = variant.getAssembleProvider().get()
        assembleTaskProvider.doLast {
            copy {
                variant.outputs.all { file ->
                    copy {
                        from file.outputFile
                        into outputFile
                    }
                }
            }
        }

    }
}

//为apk名字添加日期信息
static def time() {
    return new Date().format("MM-dd_HH-mm", TimeZone.getTimeZone("GMT+08:00"))
}


```

