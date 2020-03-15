# intellij 插件开发之 - 添加菜单到右键菜单


## 本文目标

* 添加一个菜单入口到右键菜单中

* 点击该菜单的时候获取所指向的文件名和文件路径

* 点击该菜单的时候获取光标所在行的内容

* 了解 AnActionEvent 相关功能

* 了解有什么内置 Action


## 开始之前

在本系列文章第二篇中，我们有一个由系统生成的 plugin.xml 文件，我们往里面写入了我们第一个菜单配置。但是一些自动生成东西还没有改，修改一下:

```
<idea-plugin>
    <id>com.baiguiren.my-first-plugin</id>
    <name>Hello World</name>
    <version>0.0.1</version>
    <vendor email="rubymay21s@gmail.com" url="https://www.baiguiren.com">rubys_</vendor>

    <description><![CDATA[
    This is my first plugin for jetbrains IDE.
    ]]></description>
    
    .... 其他内容
</idea-plugin>
```

我们把插件名称和版本号改掉。

另外我们把 `build.gradle` 里面的 version 改成和 `plugin.xml` 里面一样的版本号。这会是 build 之后生成 zip 文件的版本号。


## 新建一个获取当前文件的 Action

现在假设我们需要在右键菜单中加一个获取当前选中的文件的菜单，通过点击这个菜单可以获取到当前光标所指向的文件。

> 在 jetbrains IDE 中，右键指向的地方往往包含了很多有用的信息，比如，文件是什么、文件路径、光标所在行、对应的 Project 是哪个。通过获取这些信息，可以做一些快捷的操作，比如右键菜单里内置的 Run 菜单，点击的时候可以直接运行当前文件。

在 `com.baiguiren.actions` 包下面新建一个 `GetCurrentFileAction` 类:

```
package com.baiguiren.actions;

import com.intellij.openapi.actionSystem.AnAction;
import com.intellij.openapi.actionSystem.AnActionEvent;
import org.jetbrains.annotations.NotNull;

public class GetCurrentFileAction extends AnAction {
    @Override
    public void actionPerformed(@NotNull AnActionEvent e) {
    }
}
```

### 获取当前文件名、文件路径

Virtual file system(vfs) 是 Intellij 平台一个封装了大部分文件操作的组件。而 VirtualFIle 则代表了 vfs 中的一个具体的文件对象。

我们可以使用 VirtualFile 对象来实现一些对文件的操作，比如获取文件内容、重命名、移动到其他地方、删除等。

获取 VirtualFile 可以通过 AnActionEvent 来获取当前的 VirtualFile:

```
VirtualFile virtualFile = e.getData(CommonDataKeys.VIRTUAL_FILE);
```

如果我们想获取当前的文件名、文件路径，直接调用 `virtualFile` 的方法即可:

```
notify("file name: " + virtualFile.getName(), e);
notify("file path: " + virtualFile.getPath(), e);
```


### 获取光标所在行内容

除了 VirtualFile 之外，Intellij 也提供了一个光标的抽象和一个文档的抽象: Caret、Document。我们先获取这两个对象:

```
// 光标
Caret caret = e.getData(CommonDataKeys.CARET);
// 获取当前 Editor 对象
Editor editor = e.getData(CommonDataKeys.EDITOR);
if (editor == null) return;
// 也是当前操作的文件，不过这是是文件内容的抽象
Document document = editor.getDocument();
```

我们可以结合 Document 和 Caret 对象来获取到文件当前光标所在行:

```
// 获取当前行开始位置和结束位置
// CaretModel 是操作光标的对象
CaretModel caretModel = caret.getCaretModel();
int caretOffset = caretModel.getOffset();
// 光标所在行的行号
int lineNum = document.getLineNumber(caretOffset);
// 光标所在行起始位置
int lineStartOffset = document.getLineStartOffset(lineNum);
// 光标所在行结束位置
int lineEndOffset = document.getLineEndOffset(lineNum);

// 获取光标所在行的内容
String lineContent = document.getText(new TextRange(lineStartOffset, lineEndOffset));
```


## 添加到 plugin.xml 中

我们前面的文章提到过，我们插件的所有东西都需要在 plugin.xml 中配置，这个 `GetCurrentFileAction` 也不例外:

```
<actions>
    </action>
    <action class="com.baiguiren.actions.GetCurrentFileAction" id="get_current_file"
            text="Get Current File" description="Get current file">
        <add-to-group group-id="EditorPopupMenu" anchor="first"/>
        <add-to-group group-id="ProjectViewPopupMenu" anchor="first"/>
    </action>
</actions>
```

这个和上一篇文章不同的地方是:

1. group-id 改成了 EditorPopupMenu，表示的是将该 Action 放在编辑器的右键菜单中。

2. 有两个 add-to-group, 另一个的 group-id 是 ProjectViewPopupMenu，表示我们在左侧的 Project 视图里面右键的时候同样可以进行该操作。


## 所有代码

notify 方法是右下角显示消息通知的方法。

```
package com.baiguiren.actions;

import com.intellij.notification.Notification;
import com.intellij.notification.NotificationType;
import com.intellij.openapi.actionSystem.AnAction;
import com.intellij.openapi.actionSystem.AnActionEvent;
import com.intellij.openapi.actionSystem.CommonDataKeys;
import com.intellij.openapi.editor.Caret;
import com.intellij.openapi.editor.CaretModel;
import com.intellij.openapi.editor.Document;
import com.intellij.openapi.editor.Editor;
import com.intellij.openapi.util.TextRange;
import com.intellij.openapi.vfs.VirtualFile;
import org.jetbrains.annotations.NotNull;

public class GetCurrentFileAction extends AnAction {
    @Override
    public void actionPerformed(@NotNull AnActionEvent e) {
        // virtualFile 是当前指向的文件的抽象
        VirtualFile virtualFile = e.getData(CommonDataKeys.VIRTUAL_FILE);
        if (virtualFile == null) {
            notify("Can not get virtual file.", e);
            return;
        }

        // 1. 获取所指向的文件名、文件路径
        notify("file name: " + virtualFile.getName(), e);
        notify("file path: " + virtualFile.getPath(), e);

        // 2. 获取光标所在行的内容
        // Caret 是编辑器汇中的光标
        Caret caret = e.getData(CommonDataKeys.CARET);
        if (caret != null) {
            // 获取当前 Editor 对象
            Editor editor = e.getData(CommonDataKeys.EDITOR);
            if (editor == null) return;
            // 也是当前操作的文件，不过这是是文件内容的抽象
            Document document = editor.getDocument();

            // 获取当前行开始位置和结束位置
            // CaretModel 是操作光标的对象
            CaretModel caretModel = caret.getCaretModel();
            int caretOffset = caretModel.getOffset();
            // 光标所在行的行号
            int lineNum = document.getLineNumber(caretOffset);
            // 光标所在行起始位置
            int lineStartOffset = document.getLineStartOffset(lineNum);
            // 光标所在行结束位置
            int lineEndOffset = document.getLineEndOffset(lineNum);

            // 获取光标所在行的内容
            String lineContent = document.getText(new TextRange(lineStartOffset, lineEndOffset));
            notify("line content: " + lineContent, e);
        }
    }

    private static void notify(String msg, AnActionEvent e) {
        Notification notification = new Notification(
                "my first plugin",
                "Message",
                msg,
                NotificationType.INFORMATION
        );
        notification.notify(e.getProject());
    }
}

```

下图是光标所在行，我们在这里鼠标右键选择 "Get Current File":

![get-current-file-1](https://github.com/eleven26/intellij-plugin-notes/blob/master/images/get-current-file-1.png)

输出:

```
11:41 AM	Message: file name: test_sales.py

11:41 AM	Message: file path: /Users/ruby/code/YQY-API/tests/sales/test_sales.py

11:41 AM	Message: line content:         doc_url='/api/sales/#api-Sales-hasConsignor',
```

我们有个场景是，需要根据这个 url 来结合当前环境配置来在浏览器打开链接，而域名是环境变量配置的，所以这样就很方便的实现了动态的域名跳转。


![get-current-file-2](https://github.com/eleven26/intellij-plugin-notes/blob/master/images/get-current-file-2.png)

输出:

```
12:15 PM	Message: file name: .gitignore

12:15 PM	Message: file path: /Users/ruby/code/YQY-API/.gitignore
```

这一次没有在编辑器里面，所以也就没有行内容了。


## AnActionEvent 还能做什么

经过前面的了解，已经知道了，我们的 Action 在执行的时候可以拿到一个 AnActionEvent 参数，这个参数有什么作用呢？

* 获取当前 Project，Project 也是 Intellij 中的一个抽象概念，我们通过 File->New->Project 或者其他方式打开一个项目的时候，这次打开的项目在 IDE 里面就是一个 Project

```
e.getProject()
```

* 获取 com.intellij.openapi.actionSystem.CommonDataKeys 里面的对象

```
// 当前编辑器
e.getData(CommonDataKeys.EDITOR);
// 当前光标（插入符号）
e.getData(CommonDataKeys.CARET);
// 当前 VirtualFile
e.getData(CommonDataKeys.VIRTUAL_FILE);
// 当前 Project
e.getData(CommonDataKeys.PROJECT);
// ... 还有其他一些不一一列出了
```


## 内置 action

我们在插件的配置文件中 plugin.xml 中写了一些 group-id，拿来将我们的 action 放在和这些 action group 一起的地方:

```
<action class="com.baiguiren.actions.HelloWorldAction" id="hello_world"
        text="Hello World" description="Hello world">
    <add-to-group group-id="ToolsBasicGroup" anchor="first"/>
</action>
```

IDE 本身也是通过 xml 来管理这些配置的, IDE 所有的 Action 都可以在 resources.jar 里面找到:

![get-current-file-3](https://github.com/eleven26/intellij-plugin-notes/blob/master/images/get-current-file-3.png)



## 参考链接

1. virtual_file_system, https://www.jetbrains.org/intellij/sdk/docs/basics/virtual_file_system.html

2. virtual_file, https://www.jetbrains.org/intellij/sdk/docs/basics/architectural_overview/virtual_file.html

3. action_system, https://www.jetbrains.org/intellij/sdk/docs/basics/action_system.html
