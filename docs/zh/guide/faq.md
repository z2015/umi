---
sidebarDepth: 3
---

# FAQ

## General

### 是否可用于生产环境？

umi 是蚂蚁金服的底层前端框架，已直接或间接地服务了 600+ 应用，包括 java、node、H5 无线、离线（Hybrid）应用、纯前端 assets 应用、CMS 应用等。

### 如何查看 react、react-dom、react-router 等版本号？

```bash
$ umi -v --verbose

umi@2.0.0
darwin x64
node@v10.6.0
umi-build-dev@1.0.0
af-webpack@1.0.0
babel-preset-umi@1.0.0
umi-test@1.0.0
react@16.4.2 (/Users/chencheng/code/github.com/ant-design/ant-design-pro/node_modules/react)
react-dom@16.4.2 (/Users/chencheng/code/github.com/ant-design/ant-design-pro/node_modules/react-dom)
react-router@4.3.1 (/Users/chencheng/code/github.com/umijs/umi/packages/umi-build-dev/node_modules/react-router)
react-router-dom@4.3.1 (/Users/chencheng/code/github.com/ant-design/ant-design-pro/node_modules/react-router-dom)
dva@2.4.0 (/Users/chencheng/code/github.com/ant-design/ant-design-pro/node_modules/dva)
dva-loading@2.0.5
dva-immer@0.2.3
path-to-regexp@1.7.0
```

### 如何引入 @babel/polyfill ？

先安装依赖，

```bash
$ yarn add @babel/polyfill
```

然后新建 `src/global.js`，内容如下：

```js
import '@babel/polyfill';
```

### 如何动态修改 title ？

可以通过 [react-helmet](https://github.com/nfl/react-helmet) 动态修改 title 。

> 注意：在混合应用中，ios 端 web 容器内，使用 react-helmet 失效的话，可以尝试使用[react-document-title](https://github.com/gaearon/react-document-title)。

## 报错

### `Object.values` is not a function

e.g.

<img src="https://gw.alipayobjects.com/zos/rmsportal/mTaaEfxKkkGAQicDOSeb.png" />

升级 node 版本，并确保版本是 8.10 或以上。

### `exports is not defined`

e.g.

<img src="https://gw.alipayobjects.com/zos/rmsportal/fLNyyPNyquAGoYQxxIDI.png" />

检查 babel 配置，删除不必要的 preset 和 plugin 。

### `Plugin umi-plugin-react:pwa initialize failed`

e.g.

<img src="https://gw.alipayobjects.com/zos/rmsportal/lSuOXlbtrZPLoMaLBODj.png" />

确保有 package.json 并配置 `name` 属性。

### `Conflicting order between [mini-css-extract-plugin]`

e.g.

<img src="https://gw.alipayobjects.com/zos/rmsportal/mjzdexbrmZulkjCAqzPC.png" />

这是 [webpack 插件的问题](https://github.com/webpack-contrib/mini-css-extract-plugin/issues/250)，不会影响 CSS 文件的正常生产，可暂时忽略。

### `umi` 不是内部或外部命令

e.g.

<img src="https://gw.alipayobjects.com/zos/rmsportal/fatmbcGwSOwDntHjmrtG.png" />

需配置 NODE_PATH 环境变量，如使用 yarn，可通过执行 `yarn global bin` 拿到 bin 路径。

## webpack

### 如何配置额外的 loader ?

比如 svg 我希望不走 base64，而是全部产生 svg 文件，可以这样配：

```js
export default {
  // 添加 url-loader 的 exclude
  urlLoaderExcludes: [/.svg$/],
  // 添加 loader
  chainWebpack(config) {
    config.module
      .rule('svg-with-file')
      .test(/.svg$/)
      .use('svg-with-file-loader')
      .loader('file-loader');
  },
};
```

## CSS

### 为啥我 import 的 css 文件不生效？

umi 默认是开启 css modules 的，请按照 css modules 的方式进行书写。

参考：

- [css-modules/css-modules](https://github.com/css-modules/css-modules)
- [CSS Modules 用法教程](http://www.ruanyifeng.com/blog/2016/06/css_modules.html)

### 如何禁用 css modules ？

修改 `.umirc.js`:

```json
{
  "disableCSSModules": true
}
```

但没有特殊的理由时，不建议关闭 css modules。

### 如何使用 sass ？

先安装额外的依赖，

```bash
$ npm i node-sass sass-loader --save
```

然后修改 `.umirc.js`:

```json
{
  "sass": {}
}
```

## Test

### 如何断点调试

确保 node 在 8.10 以上，然后执行：

```bash
$ node --inspect-brk ./node_modules/.bin/umi test
```

然后在浏览器里打开 chrome://inspect/#devices 进行 inspect 和断点。

## 部署

### build 后访问路由刷新后 404？

几个方案供选择：

- 改用 hashHistory，在 `.umirc.js` 里配 `history: 'hash'`
- 静态化，在 `.umirc.js` 里配 `exportStatic: true`
- 服务端配置路由 fallback 到 index.html

### build 之后图片丢失？

可能是图片没有正确引用，可以参考一下代码，正确引入图片。

```js
import React from 'react';
import logo from './logo.png'; // 告诉WebPACK这个JS文件使用这个图像

console.log(logo); // logo.84287d09.png

function Header() {
  // 导入图片
  return <img src={logo} alt="Logo" />;
}

export default Header;
```

在 css 中使用，注意不要使用绝对路径

```css
.Logo {
  background-image: url(./logo.png);
}
```

> 注意：图片大小小于 10 k 时会走 base64。即不会被拷贝到 public 文件夹下，而是以 base64 的资源存在。

## SSR

### document is not defined, navigator is not defined, \* is not not defined

原因：Umi SSR 先执行服务端代码，再执行客户端。`document`、`navigator` 等对象只在客户端使用。解决方案：

1. 建议将使用到客户端对象的代码，放在 `componentDidMount`、`useEffect` 中（服务端不会执行），避免过多副作用代码影响服务端渲染。
1. 在这些对象前加上判断 `typeof navigator !== 'undefined'` 或 `typeof document !== 'undefined'`

### SSR 没有样式，样式加载不对

原因：Umijs 配置的 [publicPath](https://umijs.org/zh/config/#publicpath) 未匹配服务端的路由。导致访问 `/umi.js` 等资源路径时，未正确映射到指定文件。解决方案：

1. 试着访问链接 `http://yourHost/umi.js` 或 `http://yourHost/dist/umi.js` 看哪个链接能返回正确的 js/css 文件内容。
1. 对应修改 `publicPath` 路径。

### styled-components 编译失败

添加 [babel-plugin-styled-components](https://github.com/styled-components/babel-plugin-styled-components) babel 插件。[#3508](https://github.com/umijs/umi/issues/3508#issuecomment-546610547)

```js
// .umirc.js
extraBabelPlugins: [
  "babel-plugin-styled-components"
],
```

## UMI UI

### Umi 版本过低，请升级到最新

Umi UI 需要 umi@2.9 或以上，如果本地项目的版本不匹配，会报这个错误。

解决方案就是升级到最新版。

* 如果 package.json 中的 umi 依赖是能自动匹配到最新版的，比如 `^2.9` 或者 `2.x`，删除 `node_modules` 重装依赖即可
* 如果 package.json 中的 umi 依赖不能匹配到最新版，比如 `~2.8` 或者 `2.8.0-beta.1`，那么需改成 `^2.9` 或其他能匹配到最新版的写法，然后删除 `node_modules` 再重装依赖

### EACCES: permission denied create-umi

Umi UI 创建项目需要有执行的权限。

解决方案是将提示的路径权限提升，给予执行权限。

### Terminal need node-pty module

Umi UI 未安装或编译成功 [node-pty](https://www.npmjs.com/package/node-pty) 模块，解决方案如下：

#### Windows

> Windows 用户确保已安装 [Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk) 并且 [Node.js](https://nodejs.org/en/download/) 版本在 10.x 以上。

* 请以管理员身份在 PowerShell 执行 `npm install --global --production windows-build-tools`。
* 重装 umi 依赖

<img src="https://user-images.githubusercontent.com/13595509/69021231-1955a400-09f2-11ea-8551-4a6dcf8fe28f.png" width="400" />

#### Linux/Ubuntu

> 确保 python 已安装，并且 Node 版本在 10.x 以上。

执行以下命令：

```
$ sudo apt install -y make python build-essential
```
