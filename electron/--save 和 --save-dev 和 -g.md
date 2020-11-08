```
# <=> 是等价于的含义

npm install   <=> npm i
--save        <=> -S     
--save-dev    <=> -D 
npm run start <=> npm start
```

**npm install**

1. 仅安装模块到项目 node_modules 目录下。
2. 不会将模块依赖写入 devDependencies 或 dependencies 节点。
3. 运行 npm install 初始化项目时不会下载模块依赖。

**npm install --save**

1. 安装模块到项目 node_modules 目录下。
2. 会将模块依赖写入 dependencies 节点。
3. 运行 npm install 初始化项目时，会将模块下载到项目目录下。
4. 运行 npm install --production 或者注明 NODE_ENV 变量值为 production 时，会自动下载模块到 node_modules 目录中。

一般用于项目（运行时、发布到生产环境时）依赖，例如 antd , element,react...

**npm install --save-dev**

1. 安装模块到项目 node_modules 目录下。
2. 会将模块依赖写入 devDependencies 节点。
3. 运行 npm install 初始化项目时，会将模块下载到项目目录下。
4. 运行 npm install --production 或者注明 NODE_ENV 变量值为 production 时，不会自动下载模块到 node_modules 目录中。

一般用于工程构建（开发时、打包时）依赖，例如 xxx-cli , less-loader , babel-loader...

**npm install -g**

1. 安装模块到全局，不会在项目 node_modules 目录中保存模块包。
2. 不会将模块依赖写入 devDependencies 或 dependencies 节点。
3. 运行 npm install 初始化项目时不会下载模块。


