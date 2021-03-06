# npm 依赖管理之 peerDependencies

插件开发中经常用到 peerDependencies

## npm2 中 dependencies 与 peerDependencies 区别

假设我们当前的项目是 MyProject，项目中有一些依赖，比方其中有一个依赖包 PackageA，该包的`package.json`文件指定了对 PackageB 的依赖：

```js
{
    "dependencies": {
        "PackageB": "1.0.0"
    }
}
```

如果我们在我们的 MyProject 项目中执行`npm install PackageA`, 我们会发现我们项目的目录结构会是如下形式：

```js
;MyProject | -node_modules | -PackageA | -node_modules | -PackageB
```

那么在我们的项目中，我们能通过下面语句引入"PackageA"：

```js
var packageA = require('PackageA')
```

但是，如果你想在项目中直接引用 PackageB:

```js
var packageA = require('PackageA')
var packageB = require('PackageB')
```

这是不行的，即使 PackageB 被安装过；因为 Node 只会在`“MyProject/node_modules”`目录下查找 PackageB，它不会在进入 PackageA 模块下的`node_modules`下查找。

所以，为了解决这个问题，在 MyProject 项目`package.json`中我们必须直接声明对 PackageB 的依赖并安装。

但是，有时我们不用在当前项目中声明对 PackageB 的依赖就可以直接引用，尤其是，PackageA 是一个类似于 grunt 的插件，例如`grunt-contrib-jshint`。

为什么在项目中不用声明就可以直接使用呢？这就不得不说说`peerDependencies`的作用了。

peerDependencies 的引入
为了解决这种问题：

于是 peerDependencies 就被引入了。例如上面 PackageA 的 package.json 文件如果是下面这样：

```js
{
    "peerDependencies": {
        "PackageB": "1.0.0"
    }
}
```

那么，它会告诉 npm：如果某个 package 把我列为依赖的话，那么那个 package 也必需应该有对 PackageB 的依赖。

也就是说，如果你`npm install PackageA`，你将会得到下面的如下的目录结构：

```
MyProject
|- node_modules
   |- PackageA
   |- PackageB
```

你可能注意到：

在 npm2 中，即使当前项目 MyProject 中没有直接依赖 PackageB，该 PackageB 包依然会安装到当前项目的`node_modules`文件夹中。

下面的代码现在可以正常工作了，因为两个包在`"MyProject/node_modules"`中被安装了：

```
var packageA = require('PackageA')
var packageB = require('PackageB')
```

总结一句话，`peerDependencies`的具体作用：

`peerDependencies`的目的是提示宿主环境去安装满足插件`peerDependencies`所指定依赖的包，然后在插件 import 或者 require 所依赖的包的时候，永远都是引用宿主环境统一安装的 npm 包，最终解决插件与所依赖包不一致的问题。

举个例子，就拿目前基于 react 的 ui 组件库 ant-design@3.x 来说，因该 ui 组件库只是提供一套 react 组件库，它要求宿主环境需要安装指定的 react 版本。具体可以看它 package.json 中的配置：

```
  "peerDependencies": {
    "react": ">=16.0.0",
    "react-dom": ">=16.0.0"
  }
```

它要求宿主环境安装 react@>=16.0.0 和 react-dom@>=16.0.0 的版本，而在每个 antd 组件的定义文件顶部：

```
import * as React from 'react';
import * as ReactDOM from 'react-dom';
```

组件中引入的 react 和 react-dom 包其实都是宿主环境提供的依赖包。

## npm2 和 npm3 中 peerDependencies 的区别

正如上一节谈论的，在 npm2 中，PackageA 包中`peerDependencies`所指定的依赖会随着 npm install PackageA 一起被强制安装，所以不需要在宿主环境的`package.json`文件中指定对 PackageA 中 peerDependencies 内容的依赖。

但是在 npm3 中，`peerDependencies`的表现与 npm2 不同：

npm3 中不会再要求 peerDependencies 所指定的依赖包被强制安装，相反 npm3 会在安装结束后检查本次安装是否正确，如果不正确会给用户打印警告提示。

就拿上面的例子来说，如果我们`npm install PackageA`安装 PackageA 时，你会得到一个警告提示说：

PackageB 是一个需要的依赖，但是没有被安装。
这时，你需要手动的在 MyProject 项目的 package.json 文件指定 PackageB 的依赖。

另外，在 npm3 的项目中，可能存在一个问题就是你所依赖的一个 package 包更新了它 peerDependencies 的版本，那么你可能也需要在项目的 package.json 文件中手动更新到正确的版本。否则会出现的警告信息：
