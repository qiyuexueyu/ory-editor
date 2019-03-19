# Core: [ory-editor-core](https://www.npmjs.com/package/ory-editor-core)

Core 内核是 ORY Editor 的主系统。它包含创建和修改布局的逻辑和对组件操作的响应。

## 快速开始

ORY Editor 使用 Redux store 来管理内部状态。创建一个编辑器实例：

```jsx
import Editor from 'ory-editor-core'

const editor = new Editor()
```

Redux store 将被同步创建。因此,  不能在一个应用生命周期里创建多个编辑器实例（必须是单例模式）

```jsx
import Editor from 'ory-editor-core'

const editor = new Editor()
const editor2 = new Editor() // Don't do this.
```

## 添加插件

现在我们已经知道如何创建一个空的编辑器。接下来我们添加一些插件。我们将使用 ORY Editor 仓库中已经存在的插件。我们把这些插件叫做 ORY 插件，因为他们由我们编写和维护。

我们首先尝试图片插件（image plugin）。这是一个简单的内容插件，你可以指定一个图片地址（URL）。当前它还不支持图片上传。

<p>
  <figure align="center">
    <img alt="A content cell" src="../images/content-cell.png"><br>
    <figcaption align="center"><em>ory image plugin</em></figcaption>
  </figure>
</p>

为了安装图片插件，需要使用 npm:

```shell
npm install ory-editor-plugins-image
```

然后把它添加到编辑器实例:

```jsx
import Editor from 'ory-editor-core'

import image from 'ory-editor-plugins-image'
import 'ory-editor-plugins-image/lib/index.css'

const editor = new Editor({
  plugins: {
    content: [image],
  }
})
```

Let's start from top to bottom. First, we import the image plugin and the CSS required for it:

```jsx
import image from 'ory-editor-plugins-image'
import 'ory-editor-plugins-image/lib/index.css'
```

We assume that you are running webpack with a plugin capable of importing CSS. If this confuses you, go
to https://gitter.im/webpack/webpack and ask for help - they are very nice.

Next we create the editor instance and pass the image content plugin via the constructor.

```jsx
const editor = new Editor({
  plugins: {
    content: [image],
  }
})
```

It is also possible to add/set/remove both layout and content plugins during runtime
(meaning after creating the editor instance) as followed. Using these methods will force an editor re-render.

```jsx
editor.addContentPlugin(image)
editor.removeContentPlugin(image)
editor.setContentPlugins([image])
```

## Rendering

Next, we obviously want to render the editor. The core packages exports a ReactJS component called `Editable`. We assume
that our HTML looks something like this:

```html
<!DOCTYPE html>
<html>
  <body>
    <div id="editable-1"></div>
    <div id="controls"></div>
  </body>
</html>
```

In that case, the javascript application will look something like this:

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

import Editor, { Editable } from 'ory-editor-core'

import image from 'ory-editor-plugins-image'
import 'ory-editor-plugins-image/lib/index.css'

const editor = new Editor({
  plugins: {
    content: [image]
  }
})

ReactDOM.render((
  <Editable editor={editor} />
), document.getElementById('editable-1'))
```

All we did was render the `Editable` component using ReactJS and passing the `editor` instance. There's however a problem
here - nothing will happen. This is because there is no content available to render. Let's change that!

### Creating an empty state

To create an empty state, the core exports a method called `createEmptyState`. The result is a JSON object containing,
amongst others, a unique id.

```
import { createEmptyState } from 'ory-editor-core'

const editable = createEmptyState()
console.log(editable.id) // gives something like "29fb21c6-6e00-416f-a8e1-2be9fb84801c"
```

Adding this to the code from above we get:

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

import Editor, { Editable, createEmptyState } from 'ory-editor-core'

import image from 'ory-editor-plugins-image'
import 'ory-editor-plugins-image/lib/index.css'

const editable = createEmptyState()

const editor = new Editor({
  plugins: {
    content: [image]
  },
  editables: [editable],
})

ReactDOM.render((
  <Editable
    editor={editor}
    id={editable.id}
  />
), document.getElementById('editable-1'))
```

Note that we added `id={editable.id}` to `<Editable>`. This is required because we need to tell the component
which editable we want to render there, as we could also do:

```jsx
ReactDOM.render((
  <div>
    <div className="left">
      <Editable editor={editor} id="1" />  
    </div>
    
    <div className="right">
      <Editable editor={editor} id="2" />    
    </div>
  </div>
), document.getElementById('editable-1'))
```

## Saving and loading

Now that you know how to render an editor and fill it with an empty state, let's take a look at how
saving and loading looks like.

You can catch changes using the `onChange` property of `<Editable>`. `onChange` accepts a function with a single
argument - the editable's state.

```jsx
ReactDOM.render((
  <Editable
    editor={editor}
    id="1"
    onChange={(editable) => {
      console.log(editable)
    }}
  />
), document.getElementById('editable-1'))
```

The state you receive is a serialized version of the internal editor state. It contains primitive values (string, number, array, map)
only and is safe to serialize to JSON.

Let's say you have a function called `saveToBackend` which stores your content in a database. You can call this method
from onChange:

```jsx
ReactDOM.render((
  <Editable
    editor={editor}
    id="1"
    onChange={(editable) => {
      saveToBackend(editable)
    }}
  />
), document.getElementById('editable-1'))
```

To load the content, simply pass it to the constructor

```jsx
const editable = loadFromBackend() // just an example

const editor = new Editor({
  plugins: {
    content: [image]
  },
  editables: [editable],
})

or use the `trigger` API.

​```jsx
// const editor = new Editor( ...

const editable = loadFromBackend() // just an example

editor.editable.update(editable) // update adds an editable if it does not exist yet, or updates it when it exists in the store.
```
