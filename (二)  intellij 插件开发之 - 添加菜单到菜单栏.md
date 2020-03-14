# 添加菜单到菜单栏


## 什么是 Action？

在 intellij 的 IDE 体系里面，每一个菜单、每一个工具栏、甚至每一个可以点击的按钮等等，其实都对应着一个 Action，这个 Action 是某一个操作的抽象。


### IDE 自带的 Action

* 顶部菜单栏的 "File -> New -> Project"

[action-0](/images/action-0.png)

其实这个就对应着 IDE 中的一个 Action，我们可以按下两下 shift，查看有没有对应的类：

shift + shift，输入 "NewProjectAction"，记得把弹窗右上角那里 "Include non-project items" 勾上，可以看到下面的内容

[action-1](/images/action-1.png)

我们看到的确有一个类叫做 "NewProjectAction"，是 ideaIC 里面的一个类，

* 我们看到顶部有个 "Terminal" 的按钮，其实也是有对应的 Action 的，我们搜索一下发现的确有一个类叫做 "TerminalAction":

[action-3](/images/action-3.png)

[action-2](/images/action-2.png)


### Action 和菜单是什么关系

我们现在已经知道了，我们 IDE 中的每一个可以点击的地方都对应着一个 Action，Action 代表了我们要进行的某一种操作。

**每一个菜单实际上也是一个 Action，当我们点击菜单的时候，对应的 Action 里面的对应方法会被执行**

[action-4](/images/action-4.png)

> 在 IDE 中任何可以点击的按钮、菜单都对应着一个 Action。


## 怎样添加一个菜单？

现在我们已经知道了菜单和 Action 大概是什么一个关系，现在我们需要做的就是添加一个 Action，我们看看官网文档怎么说：

An action is a class derived from the abstract class AnAction. The IntelliJ Platform calls methods of an action when a user interacts with a menu item or toolbar button.

也就是说，一个 action 是 AnAction 类的子类，当用户与菜单栏里的菜单或者工具栏按钮产生交互的时候，IDE 会调用这个类里面的方法。

具体说，其实是 `AnAction.actionPerformed()` 方法。


### 创建 Action 类

我们先在我们的插件项目 [my-first-plugin](https://github.com/eleven26/my-first-plugin) 里面新建一个 "Hello World" 的 Action：

```
package com.baiguiren.actions;

import com.intellij.notification.Notification;
import com.intellij.notification.NotificationType;
import com.intellij.openapi.actionSystem.AnAction;
import com.intellij.openapi.actionSystem.AnActionEvent;
import org.jetbrains.annotations.NotNull;

public class HelloWorldAction extends AnAction {
    @Override
    public void actionPerformed(@NotNull AnActionEvent e) {
        Notification notification = new Notification("my first plugin", "Message", "Hello World!", NotificationType.INFORMATION);
        notification.notify(e.getProject());
    }
}

```

这个类继承自 `com.intellij.openapi.actionSystem.AnAction`，里面只有一个 `actionPerformed` 方法，这个方法就是 Action 的核心所在，当用户点击对应的菜单的时候，这个方法会被执行。

这个方法里面只有两行代码，这两行代码的作用是在 IDE 右下角弹出提示消息。


### 注册 Action

我们现在已经实现了 Action 类，但是 IDE 还不知道我们有这个 Action，所以这个时候需要提到 `META-INF/plugin.xml` 了，这个文件就是我们插件的全部配置。我们所有的 Action 或者其他插件功能都要在这里进行配置，这样 IDE 才知道我们的插件有什么功能。

我们在 `plugin.xml` 里面的 `<actions></actions>` 里面添加一个 action，里面配置的 action 类是上面的 `HelloWorldAction`:

```
<actions>
    <action class="com.baiguiren.actions.HelloWorldAction" id="hello_world" text="Hello World" description="Hello world">
        <add-to-group group-id="ToolsBasicGroup" anchor="first"/>
    </action>
</actions>
```

`action` 的 class 指明了这个 action 触发时所调用的类。

这里简单说一下用到的 xml 标签、属性:

`actions`: 所有 action 需要配置在这个标签里面

`action`: 表示某一个具体的 action

`class`: action 对应的类，当用户点击该 action 的时候，所调用的类

`id`: action 唯一 id

`text`: 这个 action 在 UI 上面显示的文字

`description`: action 的一段描述性文字，用来说明这个 action 是做什么的

`add-to-group`: 表示将 action 加到哪个地方，这里的 `ToolsBasicGroup` 表示是菜单栏的 `Tools` 菜单

`anchor`: 表示该 action 在 `ToolsBasicGroup` 中的位置，first 表示置于顶部，last 表示置于底部


### 启动一个 IDE 来调试

双击 "Gradle" 工具栏里面的 "runIde"，即可启动一个 IDE 来调试我们的插件:

[action-7](/images/action-7.png)

我们启动这个调试的 IDE 之后，可以在顶部菜单栏的 "Tools" 下面找到我们的 "Hello World" 子菜单:

[action-5](/images/action-5.png)

点击的时候，可以发现 IDE 的右下角有个弹窗:

[action-6](/images/action-6.png)

到这里，一个最基本的菜单已经添加到我们的 IDE 中了。


## 如何打包插件？

我们写插件往往不是自己一个人使用，有时候可能需要构建一个分享给其他人，其实官网提供的 Gradle 中已经提供了对应的 task，我们可以在 Gradle 工具栏中看到 "build"，双击它即可构建一个可进行分发的插件压缩包了。

[action-8](/images/action-8.png)

构建之后，我们可以看到插件项目根目录下多了一个 build 目录，而里面的 `distributions` 文件夹就是保存可分发插件的地方。

[action-9](/images/action-9.png)


## 如何在其他 IDE 中使用这个插件？

我们打开 IDE 的设置页面，选择 "plugins"，点击右上角齿轮，再点击 "Install Plugin from Disk"，然后选择上面 build 出来的 zip 文件，最后重启 IDE 即可。

[action-10](/images/action-10.png)

如下，我们实现了在其他 IDE 中使用我们的插件:

[action-11](/images/action-11.png)


## 参考链接

- [action_system](https://www.jetbrains.org/intellij/sdk/docs/basics/action_system.html)

- [Creating Actions](https://www.jetbrains.org/intellij/sdk/docs/tutorials/action_system/working_with_custom_actions.html)

