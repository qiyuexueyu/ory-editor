# 简介

欢迎来到 ORY 编辑器指南。请先知悉使用 ORY 编辑器需要熟悉 ReactJS 知识，比如 webpack 工具， ES6。 如果你缺乏这些知识可以在 [Chat](https://gitter.im/ory-am/editor) 寻求帮助。

## 示范

眼见为实，让我们从一个 Demo 开始。

![ORY Editor Demo](./images/inline-edit-md.gif)

这里有一个 demo，请自行尝试 [editor.ory.sh](https://editor.ory.sh/) 

## ORY 编辑器的不同之处

为了学习， 我们创建像[维基百科](https://de.serlo.org)这样的网站。它的内容来自互联网用户，每月有超过 50 万的人使用这个平台。我们意识到现有的开源内容编辑器解决方案通常有下面三个缺陷的一个或多个。

- 产生大量的标记，必须进行大量的清理工作，同时 XSS  的威胁总是存在。
- 在制作内容以前，作者需要学习特殊的标记语言，比如 markdown。这些基于文本的解决方案通常无法指定布局，像表这样复杂的数据结构编辑起来很麻烦。
- 无法使用布局（如 flexbox 或 grids）

我的结论是，解决方案必须符合以下原则：

- 状态使用规范的 json, 不牵涉 HTML。
- 它是一个不需要编程经验和特殊训练的虚拟编辑器。
- 基于可服用的 React 组件。 让开发，作者和设计人员以一种更好的方式在一起工作。
- 可以在移动手机和触摸设备上使用.

基于这些原则，我出实现 ORY 编辑器，你现在看到的就是它。

## ORY 编辑器是如何工作的

ORY 编辑器主要提供了一个创建和编辑布局的工具。在它的核心结构里面，他们是 Cells 和 Rows。布局系统和 Bootstrap 的栅格系统非常相似。

An exemplary structure of an editable (other editors call this "document") could be the following:

下面是一个可编辑的模范结构树（其他编辑器把他叫做文档）

```
1. Editable
+-1. Container cell
  +-1. Row
  | +-1. Cell (text)
  | |-2. Cell (parallax background image)
  |   +-1. Row 
  |     +-1. Cell (image)
  |     |-2. Cell (image)
  |-2. Row
  | +-1. Cell (image)
  | |-2. Cell (image)
```

四种数据类型

​	**Editable 可编辑内容**（模范结构树中的 1.）-  可编辑内容是一个包含单元格和行的容器，你可以在一个页面里包含多个可编辑内容，把一个单元格拖拽到另一个可编辑内容是可以的

​	**Container cell 容器单元格** (模范结构树中的 `1.1`) - 容器单元格是模范结构树中的一个不带插件的容器单元。它在需要的时候被自动生成，在不需要的的时候被自动移除。

​	**Content cell 内容单元格** (模范结构树中的 `1.1.1`, `1.2.1`, ... ) - 内容单元格通常是模范结构树中的叶子节点 (没有子节点) ，他的行为由内容插件（content plugin）定义（你可以自定义内容插件或从 npm 下载）。 内容插件通常是富文本，音频，视频，声云插件等组件。

​	**Layout cell 布局单元格** -   布局单元格包含一组嵌套的行（Rows）和单元格（Cell）(容器, 内容, 布局)。他的作用是给它包含的内容一个布局(如视差背景, 扰流框,红色的文本框等)。布局单元格通常至少包含一个子元素，否则它将被自动移除。布局单元格由布局插件定义。


<p>
  <figure align="center">
    <img alt="A content cell" src="./images/content-cell.png"><br>
    <figcaption align="center"><em>A content cell with the image plugin</em>	</figcaption>
  </figure>
</p>


<p>
  <figure align="center">
    <img alt="A layout cell" src="./images/layout-cell.gif"><br>
    <figcaption align="center"><em>A layout cell with a switchable background image plugin</em></figcaption>
  </figure>
</p>
一个栅格系统被植入到 ORY 编辑器。它负责所有的拖拽逻辑，大小重置，焦点侦测等。作为一个开发者，你主要是通过布局和内容插件来扩展功能。此外，编辑器关注全局的数据模型。插件仅仅是一个简单的 ReactJS 组件，它通过编辑器的 onChange`, `readOnly`, `state` 接收属性。你将在最后一个章节学习到插件是如何精密工作的，他们的 API 是什么样和如何开发一个自己的插件。