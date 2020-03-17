# intellij-plugin-nodes

## Why

有时候一些操作重复来重复去开发体验极差，希望改善一下开发的体验。另外网上可以找到的资料非常少，唯有自己探索了。


## 常用链接

* [官网文档](https://www.jetbrains.org/intellij/sdk/docs/intro/welcome.html)

* [插件里面的 since-build 对应版本](https://www.jetbrains.org/intellij/sdk/docs/basics/getting_started/build_number_ranges.html)


## 插件开发系列文章

* [(一) intellij 插件开发之 - 创建第一个插件.md](https://github.com/eleven26/intellij-plugin-notes/blob/master/(%E4%B8%80)%20intellij%20%E6%8F%92%E4%BB%B6%E5%BC%80%E5%8F%91%E4%B9%8B%20-%20%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AA%E6%8F%92%E4%BB%B6.md)

* [(二) intellij 插件开发之 - 添加菜单到菜单栏.md](https://github.com/eleven26/intellij-plugin-notes/blob/master/(%E4%BA%8C)%20intellij%20%E6%8F%92%E4%BB%B6%E5%BC%80%E5%8F%91%E4%B9%8B%20-%20%E6%B7%BB%E5%8A%A0%E8%8F%9C%E5%8D%95%E5%88%B0%E8%8F%9C%E5%8D%95%E6%A0%8F.md)

* [(三) intellij 插件开发之 - 添加菜单到右键菜单.md](https://github.com/eleven26/intellij-plugin-notes/blob/master/(%E4%B8%89)%20intellij%20%E6%8F%92%E4%BB%B6%E5%BC%80%E5%8F%91%E4%B9%8B%20-%20%E6%B7%BB%E5%8A%A0%E8%8F%9C%E5%8D%95%E5%88%B0%E5%8F%B3%E9%94%AE%E8%8F%9C%E5%8D%95.md)


## TODO

* ~~(一) intellij 插件开发之 - 创建第一个插件.md~~

* ~~(二) intellij 插件开发之 - 添加菜单到菜单栏.md~~

* ~~(三) intellij 插件开发之 - 添加菜单到右键.md（编辑器右键菜单、ProjectView 右键菜单）~~

* (四) intellij 插件开发之 - plugin.xml 入门.md

* (五) intellij 插件开发之 - 编辑器光标操作.md（Caret）

* (六) intellij 插件开发之 - 编辑器文本操作.md（Vfs）

* (七) intellij 插件开发之 - 常用组件.md（Project、Document、Editor）

* (八) intellij 插件开发之 - PSI 入门.md

* (八) intellij 插件开发之 - Gradle 入门.md


## 内置图标使用

所有内置图标所在路径：

```
ideaIC-2019.1/lib/icons.jar
```

这里的 `ideaIC-2019.1` 是 插件项目里面的 build.gradle 里面的 intellij.version 版本的 idea

解压这个 icons.jar 即可看到哪些图标可以直接使用。
