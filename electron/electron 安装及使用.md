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
const { app, BrowserWindow } = require('electron')

function createWindow () {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true
    }
  })

  win.loadFile('index.html')
  win.webContents.openDevTools()
}

app.whenReady().then(createWindow)

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow()
  }
})
```

再新建文件 `index.html`，模板如下，

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
    <meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline';" />
</head>
<body>
    <h1>Hello World!</h1>
    We are using node <script>document.write(process.versions.node)</script>,
    Chrome <script>document.write(process.versions.chrome)</script>,
    and Electron <script>document.write(process.versions.electron)</script>.
</body>
</html>
```

## 安装 electron

在项目根目录下执行，

```
cnpm i -D electron@latest
```

## 运行项目

因为上面的 electron 是局部安装，所以需要这样执行（Windows 上），

```
.\node_modules\.bin\electron.cmd .
```
