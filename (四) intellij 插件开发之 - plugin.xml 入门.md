# intellij 插件开发之 - plugin.xml 入门

> 本来想把标题起做~~详解~~的，但是看了一遍 plugin.xml 的配置，其实有几个关键的地方我都还没用过，所以改成了 "入门"。这里只讲一些常用的。

一个简单但相对完整的 plugin.xml 的内容如下:

> 这是一个 xml 文件，在 Java 的项目中基本都是用 xml 来记录配置信息。

```
<!-- url 属性是插件的主页链接 -->
<idea-plugin url="https://www.jetbrains.com/idea">

  <!-- 插件名称，用在插件配置那个对话框中，格式是不同单词空格分开，各个单词首字母大写 -->
  <name>Vss Integration</name>

  <!-- 插件的唯一标识，一般用顶级包名加上插件名称 -->
  <id>com.jetbrains.vssintegration</id>

  <!-- 插件描述，用在插件配置对话框，可以包含 HTML 标签，CDATA 标签  -->
  <description>Integrates Volume Snapshot Service W10</description>

  <!-- 这里写插件最近版本的变更 -->
  <change-notes>Initial release of the plugin.</change-notes>

  <!-- 插件版本，推荐使用语义化的版本号  -->
  <version>1.0.0</version>

  <!-- 插件开发者相关信息 -->
  <vendor url="https://www.company.com" email="support@company.com">A Company Inc.</vendor>

  <!-- 指明当前插件依赖, 可以是平台依赖或者对其他插件的依赖  -->
  <depends>com.intellij.modules.platform</depends>
  <depends>com.third.party.plugin</depends>

  <!-- 可选的依赖 -->
  <depends optional="true" config-file="mysecondplugin.xml">com.MySecondPlugin</depends>

  <!-- IDE 的 build number，比如目前最新的 2019.3.3 的 build number 是 193.* -->
  <!-- 可参考 https://www.jetbrains.org/intellij/sdk/docs/basics/getting_started/build_number_ranges.html -->
  <idea-version since-build="183" until-build="183.*"/>

  <!-- 一些静态资源依赖 -->
  <resource-bundle>messages.MyPluginBundle</resource-bundle>

  <!-- 已废弃, 不必再了解 -->
  <application-components>
  </application-components>

  <!-- 已废弃, 不必再了解 -->
  <project-components>
  </project-components>

  <!-- 已废弃, 不必再了解 -->
  <module-components>
  </module-components>

  <!-- Actions，最常用的部分，常常用来添加自定义菜单 -->
  <actions>
    <action id="VssIntegration.GarbageCollection" class="com.foo.impl.CollectGarbage" text="Collect _Garbage" description="Run garbage collector">
      <keyboard-shortcut first-keystroke="control alt G" second-keystroke="C" keymap="$default"/>
    </action>
  </actions>

  <!-- 提供给其他插件来扩展当前插件的，这个我也不懂 -->
  <extensionPoints>
    <extensionPoint name="testExtensionPoint" beanClass="com.foo.impl.MyExtensionBean"/>
  </extensionPoints>

  <!-- 可以有多个 extensions 标签，用来扩展不同的特性，或者其他插件 -->
  <extensions defaultExtensionNs="VssIntegration">
    <testExtensionPoint implementation="com.foo.impl.MyExtensionImpl"/>
  </extensions>
  
  <!-- Application-level listeners -->
  <applicationListeners>
    <listener class="com.foo.impl.MyListener" topic="com.intellij.openapi.vfs.newvfs.BulkFileListener"/>
  </applicationListeners>

  <!-- Project-level listeners -->
  <projectListeners>
    <listener class="com.foo.impl.MyToolwindowListener" topic="com.intellij.openapi.wm.ex.ToolWindowManagerListener"/>
  </projectListeners>
</idea-plugin>
```


## depends

有时候我们的插件需要依赖一些其他插件、或者一些平台依赖，这个时候需要在这里指定，比如：

```
<depends>org.jetbrains.plugins.terminal</depends>
```

这个 depends 表示我们的插件需要用到官方 terminal 插件的一些功能，除了在这里配置，我们还需要在 build.gradle 里面声明这个依赖：

```
intellij {
    plugins = [
            'terminal',
    ]
    ...
}
```

这样我们才能在我们的插件代码中使用这个 terminal 插件。


## extensions

这个是给我们用来扩展平台的一些功能的，又或者扩展其他的插件功能。


> 除了下面的 action 其他没怎么用过，以后用过的时候再来补充吧。


## actions

```
<!-- Actions -->
<actions>
  <!-- 
       在前两篇教程中已经讲过这个 Action 一般是 IDE 中某个可以点击的东西，最常见的是菜单。
       id：action 的唯一标识，不能和其他 action 的 ID 相同
       class：实现了 Action 操作的类，必须实现 actionPerformed 方法，actionPerformed 会在 action 触发的时候调用
       text：在 IDE 上显示的文字
       description：这个给开发者看的描述
       icon：图标，可以不写这个属性
  -->
  <action id="VssIntegration.GarbageCollection" class="com.foo.impl.CollectGarbage" text="Collect _Garbage" description="Run garbage collector" icon="icons/garbage.png">
    <!-- 
        <add-to-group> 配置这个 action 需要加到现有的哪个 action 分组中，我们其实也可以自己定义一个 action 分组，这个在 IDE 中实际表现为一个菜单分组。
        group-id：需要添加到的那个分组 id
        anchor：在分组中的位置，可选的有 first、last、before、after
        relative-to-action：相对于分组中其他某一个 action 的位置，这个需要配合 anchor 使用，比如 relative-to-action="a" anchor="after" 就表示放在 "a" action 的后面
    -->
    <add-to-group group-id="ToolsMenu" relative-to-action="GenerateJavadoc" anchor="after"/>

    <!-- 这个没什么好说的，键盘快捷键，可以设置快捷键，然后通过快捷键调用我们的 action，这个也可以在 IDE 的快捷键配置中配置  -->
    <keyboard-shortcut keymap="$default" first-keystroke="control alt G" second-keystroke="C"/>
  </action>

  <!-- 
       <group> 定义了一个 action 分组，然后我们通常会在里面添加多个 action。
       id：分组唯一 id
       class：我们需要在这个类的 getChildren 里面提供这个分组需要的所有可用的 action
       text：分组在 IDE 上显示的文字
       description：描述
       icon：图标
       popup：指定 action 是否出现在菜单中
       compact：指定分组禁用的时候该组中的操作是否可见
  -->
  <group class="com.foo.impl.MyActionGroup" id="TestActionGroup" text="Test Group" description="Group with test actions" icon="icons/testgroup.png" popup="true" compact="true">
    <action id="VssIntegration.TestAction" class="com.foo.impl.TestAction" text="My Test Action" description="My test action"/>
    <!-- separator 是一个菜单分隔线 -->
    <separator/>
    <!-- 引入另外一个分组 -->
    <group id="TestActionSubGroup"/>
    <!-- 可以使用 reference 引入其他 action -->
    <reference ref="EditorCopy"/>
    <!-- 把当前分组加到哪里 -->
    <add-to-group group-id="MainMenu" relative-to-action="HelpMenu" anchor="before"/>
  </group>
</actions>
```

![actions-0](https://github.com/eleven26/intellij-plugin-notes/blob/master/images/actions-0.png)

比如，这里的 "Tasks & Contexts" 可以看作一个 group，子菜单里面的 "+ Open Task..." 前面的 + 是图标，"Open Task" 是 text，最后那个其实就是快捷键。

而子菜单中间那个分割线就是 "<separator/>"。 


## 参考链接

1. 官网 plugin.xml 示例, [https://www.jetbrains.org/intellij/sdk/docs/basics/plugin_structure/plugin_configuration_file.html](https://www.jetbrains.org/intellij/sdk/docs/basics/plugin_structure/plugin_configuration_file.html)

2. action_system, [https://www.jetbrains.org/intellij/sdk/docs/basics/action_system.html](https://www.jetbrains.org/intellij/sdk/docs/basics/action_system.html)

