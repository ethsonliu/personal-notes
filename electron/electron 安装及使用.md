以下为 Windows 平台上的配置。

## 安装 Node.js

下载地址：<https://nodejs.org/en/>，最好下载稳定版本，然后安装，安装完成后，看下 PATH 路径是否已经把路径加入，如果没有，请手动加进去。

比如，我的安装目录是 `E:\Nodejs`，那么就把这个安装目录放进 PATH 中。

打开 cmd，验证是否安装成功，如下，

```shell
C:\Users\liuyi>node -v
v12.18.0
```

## 安装 cnpm

打开 cmd，输入 `npm install -g cnpm --registry=https://registry.npm.taobao.org`，以后以 cnpm 代替 npm 即可。

## 一个项目

新建一个文件夹用于放置项目，在此下新建一个文件 `package.json`，模板如下，

```json
{
  "name": "hellomqtt",
  "version": "1.0.0",
  "description": "A MQTT Client based on Electron.",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/HelloTools/HelloMQTT.git"
  },
  "author": "EthsonLiu",
  "license": "GPL-3.0",
  "bugs": {
    "url": "https://github.com/HelloTools/HelloMQTT/issues"
  },
  "homepage": "https://github.com/HelloTools/HelloMQTT#readme"
}
```

注：关于 scripts 的使用，参照 <https://www.ruanyifeng.com/blog/2016/10/npm_scripts.html>。

再新建文件 `index.js`，模板如下，

```js
var app = require('app');  // 控制应用生命周期的模块。
var BrowserWindow = require('browser-window');  // 创建原生浏览器窗口的模块

// 保持一个对于 window 对象的全局引用，不然，当 JavaScript 被 GC，
// window 会被自动地关闭
var mainWindow = null;

// 当所有窗口被关闭了，退出。
app.on('window-all-closed', function() {
  // 在 OS X 上，通常用户在明确地按下 Cmd + Q 之前
  // 应用会保持活动状态
  if (process.platform != 'darwin') {
    app.quit();
  }
});

// 当 Electron 完成了初始化并且准备创建浏览器窗口的时候
// 这个方法就被调用
app.on('ready', function() {
  // 创建浏览器窗口。
  mainWindow = new BrowserWindow({width: 800, height: 600});

  // 加载应用的 index.html
  mainWindow.loadURL('file://' + __dirname + '/index.html');

  // 打开开发工具
  mainWindow.openDevTools();

  // 当 window 被关闭，这个事件会被发出
  mainWindow.on('closed', function() {
    // 取消引用 window 对象，如果你的应用支持多窗口的话，
    // 通常会把多个 window 对象存放在一个数组里面，
    // 但这次不是。
    mainWindow = null;
  });
});
```

再新建文件 `index.html`，模板如下，

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using io.js <script>document.write(process.version)</script>
    and Electron <script>document.write(process.versions['electron'])</script>.
  </body>
</html>
```

## 安装 electron

在项目根目录下执行，

```
cnpm i -D electron@latest
```

## 运行项目

因为上面的 electron 是局部安装，所以需要这样执行，

```
.\node_modules\.bin\electron.cmd .
```
