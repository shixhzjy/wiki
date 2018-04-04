### 为什么使用嵌入式tomcat
*最原始的部署项目步骤如下：*</br>
1. 先将web项目打成war包.
2. 然后将war包上传到tomcat的webapp下，启动tomcat来部署项目。通常还要指定端口、项目位置等，且这些都是重复的操作。

使用嵌入式tomcat后，不需要自己手动打包，不需要安装tomcat，直接启动运行项目即可。


build.gradle
````shell
apply plugin: 'com.bmuschko.tomcat'

dependencies {
    compile("org.springframework:spring-webmvc:4.3.14.RELEASE")

//    compile("javax.servlet:javax.servlet-api:3.1.0")

    compile("org.apache.commons:commons-lang3:3.4")

    tomcat "org.apache.tomcat.embed:tomcat-embed-core:7.0.85",
            "org.apache.tomcat.embed:tomcat-embed-jasper:7.0.85",
            "org.apache.tomcat.embed:tomcat-embed-logging-juli:7.0.85"
}

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.bmuschko:gradle-tomcat-plugin:2.5'
    }
}
````
gradle视图中就出现下图中的task：</br>
![gradle视图](https://github.com/shixhzjy/wiki/blob/master/gradle/imgs/gradle-view.jpg?raw=true)

### 进入debug模式
关键参数：`-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005`。</br>
步骤：</br>
1. 编辑tomcatRun的VM options，如图：![tomcatRun参数](https://github.com/shixhzjy/wiki/blob/master/gradle/imgs/edit-tomcatRun-configurations.jpg?raw=true)
2. 添加remote，如图：![remote-debug](https://github.com/shixhzjy/wiki/blob/master/gradle/imgs/remote-debug.jpg?raw=true)
3. 以run运行tomcatRun task，如图：![Run-tomcatRun](https://github.com/shixhzjy/wiki/blob/master/gradle/imgs/Run-tomcatRun.jpg?raw=true)
4. 以debug运行remote的debug task，![Debug-remote_debug](https://github.com/shixhzjy/wiki/blob/master/gradle/imgs/Debug-remote_debug.jpg?raw=true)

注：若要debug spring容器启动过程，不要等步骤3完成后再执行步骤4，可以在步骤3启动过程中就执行步骤4。

关于tomcat的版本及相关配置参考[https://github.com/bmuschko/gradle-tomcat-plugin/blob/master/README.md](https://github.com/bmuschko/gradle-tomcat-plugin/blob/master/README.md)</br>
Gradle properties[https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_configuration_properties](https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_configuration_properties)
