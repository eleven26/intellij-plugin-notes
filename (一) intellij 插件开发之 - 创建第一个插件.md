# 创建第一个插件


## 环境依赖

1. Java 环境

2. Intellij IDEA


## 借助 Gradle 来进行插件开发

在官网 [Plugins-Getting Started](https://www.jetbrains.org/intellij/sdk/docs/basics/getting_started.html) 里面提供了两种插件开发的方式，第一种是基于 [Gradle](https://www.jetbrains.org/intellij/sdk/docs/tutorials/build_system.html)，第二种是基于 ~~[DevKit](https://www.jetbrains.org/intellij/sdk/docs/basics/getting_started/using_dev_kit.html)~~。

~~DevKit 这种是早期使用的方式，现在已经很少使用，因为不够智能，而且配置需要手动配置，过于麻烦，并且容易出错。有些网上的文章会使用这种方式，但那些应该是很多年前的文章了。~~

> **使用 Gradle 的话，可以节省很多配置环境的时间，而且大大减少出错的机会。** 但是唯一缺点是国内的网络可能在下载依赖的时候会很慢，这个可以在 IDE 里面配置代理。如果没有代理那就要等很久很久了。


## 开始基于 Gradle 开发插件

[gradle-intellij-plugin](https://github.com/JetBrains/gradle-intellij-plugin) 这个包为我们解决了插件开发的依赖问题，文档可在 [gradle-intellij-plugin](https://github.com/JetBrains/gradle-intellij-plugin) 查看，还是比较详细的。

关于如何新建一个基于 Gradle 的插件工程，官网的文档里面已经有了：[build_system/prerequisites](https://www.jetbrains.org/intellij/sdk/docs/tutorials/build_system/prerequisites.html)、[gradle](https://www.jetbrains.com/help/idea/gradle.html#)、[getting-started-with-gradle](https://www.jetbrains.com/help/idea/getting-started-with-gradle.html#)，这里不再细说。


这里直接截图了，下面是 Intellij IDEA 的操作了：

1. 创建新项目，选择 "File -> New -> Project"

![file-new-project-1](https://github.com/eleven26/intellij-plugin-notes/blob/master/images/new-gradle-project-1.png)


2. 这一步填写一下 Name、GroupId 即可，点击 Finish

![file-new-project-2](https://github.com/eleven26/intellij-plugin-notes/blob/master/images/new-gradle-project-2.png)


3. 我们在点击了 Finish 之后，打开了这个项目，发现依赖自动就开始下载了

![file-new-project-3](https://github.com/eleven26/intellij-plugin-notes/blob/master/images/new-gradle-project-3.png)

这个过程如果没有代理会非常漫长，这里我使用了代理来下载。

如果本地有 ss 客户端，那就可以在 IDE 中打开配置页面，mac 下面是顶部菜单栏的 "Intellij IDEA -> Preference"，在里面搜索 "proxy":

![file-new-project-4](https://github.com/eleven26/intellij-plugin-notes/blob/master/images/new-gradle-project-4.png)

这个时候再把 ss 客户端设置为全局模式，然后点击底部的旋转图标即可，然后慢慢等待就可以了（当然代理也不能太慢，否则可能要等几个小时）。

![file-new-project-5](https://github.com/eleven26/intellij-plugin-notes/blob/master/images/new-gradle-project-5.png)


## Gradle 插件项目的目录结构

在等待下载的漫长过程中，我们可以先来看看这个 Gradle 项目的目录结构：

运行命令

```
tree .
```

目录结构

```
.
├── build.gradle                        # 定义一些依赖的信息、版本信息、插件信息等
├── gradle                              # 这个目录不用管，但是需要提交到版本库
├── gradlew                             # 这个是手动运行 gradle 的一个脚本，但是我们一般不手动，因为 IDEA 提供了一个
├── gradlew.bat                         # Gradle 的工具窗口，在里面双击对应的 task 即可完成手动运行 gradle 的操作
├── settings.gradle                     # 这里面写的 project 的名字，一般不需要改动
└── src                                 # 这就是插件源码目录
    ├── main
    │   ├── java
    │   └── resources
    │       └── META-INF
    │           └── plugin.xml          # 这个算是最重要的文件了，里面写我们插件的配置
    └── test                            # 这个不用多说，测试文件目录
        ├── java
        └── resources
```

## build.gradle

在等待了 4m29s 之后，我的依赖已经下载完了，先来了解几个重要的文件的内容。首先是 build.gradle：

```
# 这个 plugins 定义了我们插件所需要的依赖（这些是 gradle 插件，可以在 [https://plugins.gradle.org/](https://plugins.gradle.org/) 上面找到）
# 一个是 [java](https://docs.gradle.org/current/userguide/java_plugin.html)，一个是 [org.jetbrains.intellij](https://github.com/JetBrains/gradle-intellij-plugin/)
plugins {
    id 'java'
    id 'org.jetbrains.intellij' version '0.4.16'
}

# 下面的 group 是我们源码里面的包名前缀
group 'com.baiguiren'
# gradle build 时候生成的插件文件版本号
version '1.0-SNAPSHOT'

# 源码兼容性，指的是 JDK 的版本，不能低于 JDK8
sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
}

# 这个还是很重要的，这个指定了插件需要在什么 IDE 的版本上运行，这个主要用于测试，测试的过程中，运行 `gradle runIde`，会下载一个 IDEA 来运行我们的插件，版本就是下面指定的版本
// See https://github.com/JetBrains/gradle-intellij-plugin/
intellij {
    version '2019.3.3'
}
# 这个可先不在意是什么，可自行探索
patchPluginXml {
    changeNotes """
      Add change notes here.<br>
      <em>most HTML tags may be used</em>"""
}
```

这里提到了 `gradle runIde`，我们可以启动一个 IDE 来测试我们的插件，运行下面的命令：

```
./gradlew runIde
```

当然 windows 可能不一样，应该是使用:

```
gradlew.bat runIde
```

运行之后，会启动一个 IDE，在这个 IDE 里面，我们当前项目的插件功能都可以使用。


## plugin.xml

在开发之前再来看看这个 `plugin.xml`，这个算是项目的核心所在了，我们插件的功能都需要在这个文件中配置。

先来看看最原始版本的 `plugin.xml`。

```
<idea-plugin>
    # 插件唯一 ID，不能和其他插件 ID 一样
    <id>com.baiguiren.my-first-plugin</id>
    # 插件名称
    <name>Plugin display name here</name>
    # 开发者信息
    <vendor email="support@yourcompany.com" url="http://www.yourcompany.com">YourCompany</vendor>

    # 插件描述，会在 IDE plugins 设置页面里面显示
    <description><![CDATA[
    Enter short description for your plugin here.<br>
    <em>most HTML tags may be used</em>
    ]]></description>

    # 定义插件的依赖
    <!-- please see http://www.jetbrains.org/intellij/sdk/docs/basics/getting_started/plugin_compatibility.html
         on how to target different products -->
    <depends>com.intellij.modules.platform</depends>

    <extensions defaultExtensionNs="com.intellij">
        <!-- Add your extensions here -->
    </extensions>

    # **插件的一些操作都会在这里定义，比如加了一个菜单**
    <actions>
        <!-- Add your actions here -->
    </actions>
</idea-plugin>
```


本文到此就结束了，后面会说如何开始开发一些小功能。
