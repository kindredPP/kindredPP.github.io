---
layout: post
title:  "将现有用webpack构建的vue项目打包成桌面应用electron"
date: 2019-04-21 11:15:09
categories: vue electron
tags: vue web electron
excerpt: 将现有用webpack构建的vue项目打包成桌面应用electron
mathjax: true
---

## 将现有vue的webpack方式打包的web网站改造成桌面应用
参考：https://www.jianshu.com/p/30d44d929468

例子：打包的ivpm项目 ivpm-client.rar（同目录）

#### mac修改方法：
打包环境：mac系统、windows系统分别打包；环境node8.14.0，npm6.9.0，webpack3、vue init webpack xx 安装的vue项目、electron4.1.4

#### 本地开发包
本文是打包一个系统管理项目ivpm：

- 需要用到官网例子中的代码git clone https://github.com/electron/electron-quick-start
```
// main.js
mainWindow.loadURL(url.format({
  pathname: path.join(__dirname, 'index.html'),   //index.html即是入口html文件。
  protocol: 'file:',
  slashes: true
}))
```
```
// package.json
{
  "name": "electron-quick-start",
  "version": "1.0.0",
  "description": "A minimal Electron application",
  "main": "main.js",   //设置入口文件main.js // 打包exe时，改成electron.js
  "scripts": {
    "start": "electron ."
  }
  ...
  }
```

按上面命令，clone代码到本地，预留main.js和package.json文件备用
注意点：有可能下载的代码main.js中如下，而不是上面代码：
```
  mainWindow.loadFile('index.html')
```
没关系，按后面步骤修改即可

- 使用vue-cli创建一个新项目：vue init webpack ivpm，安装如下依赖
```
npm install electron electron-packager -D
```
这时候windows 和 mac 会分别下载相应的electron打包文件，electron-packager：是打包插件

- 将步骤1.中的main.js拷贝到ivpm项目的build目录下，并更名为electron.js(步骤1.中的package.json文件中的main属性也要修改)

- electron.js中

1. 添加模块引用

```
const url = require('url')
const path = require('path')
```

2. 修改文件路径，指向dist中的index.html


```
// electron.js
mainWindow.loadURL(url.format({
  pathname: path.join(__dirname, '../dist/index.html'),// 修改位置
  protocol: 'file:',
  slashes: true
}))
```
- 更改config/index.js中生产模式下（build）的assetsPublicPth, 原本为 /, 改为 ./

- 在新建项目package.json文件中增加一条指令
```
// ivpm/package.json
"scripts": {
   ...
    "lint": "eslint --ext .js,.vue src test/unit/specs test/e2e/specs",
    "build": "node build/build.js",
    "electron_dev": "npm run build && electron build/electron.js"   // 修改位置
    //增加这条,JSON文件不支持注释，引用时请清除
  },
```
- 执行npm run electron_dev ：会先 执行npm run build 生成dist目录 ，然后启动electron。
即可看到生成的应用程序，此方法主要是本地开发时候使用

--------------------------------------------------------------------------------
#### 打包exe（windows系统)或app（mac系统）文件

1. 复制build目录下的electron.js到dist目录中，并注意修改路径
```
// dist/electron.js
mainWindow.loadURL(url.format({
  pathname: path.join(__dirname, 'index.html'),// 修改位置
  protocol: 'file:',
  slashes: true
}))
```
2. 复制electron-quick-start例子中的package.json到dist目录中，注意修改路径
```
// package.json
{
  "name": "electron-quick-start",
  "version": "1.0.0",
  "description": "A minimal Electron application",
  "main": "electron.js",   // 修改位置
  "scripts": {
    "start": "electron ."
  }
  ...
  }
```
3. 在ivpm项目的package.json中（注意不是dist下的package.json）为之前下载好的electron-packager，增加一条启动命令
```
"scripts": {
   ...
    "lint": "eslint --ext .js,.vue src test/unit/specs test/e2e/specs",
    "build": "node build/build.js",
    "electron_dev": "npm run build && electron build/electron.js",
    "electron_build": "electron-packager ./dist helloworld --platform=darwin --arch=x64 --icon=./src/assets/home.ico --overwrite"   
    // mac增加这条
    "electron_build_windows": "electron-packager ./dist helloworld --platform=win32 --arch=x64 --icon=./src/assets/home.ico --overwrite"   
    // windows增加这条
},
```
```
electron-packager <sourcedir> <appname> –platform=<platform> –arch=<arch> [optional flags…]
```
sourcedir: 资源(dist/package.json)路径，在本例中既是./dist/

appname: 打包出的exe名称,这里取名为helloworld

platform: 平台名称（windows是win32，mac是darwin）

arch: 版本，本例为x64

后边的配置项都是选填，默认是没有这些的，这里只选填了图标。
（注意：不可通过重命名的方式将一个png或jpg格式的文件改为ico格式，会导致无法build成功）

#### 生成exe或app

执行npm run electron_build，可以看到项目目录中多了一个helloworld-darwin-x64文件，找到里面的helloworld.app运行即可。

执行npm run electron_build_windows，外层生成helloworld-win32-x64文件，找到里面的helloworld.exe运行即可。

（注意：mac系统如果想打包windows的包，需要安装额外的其他插件wine（ https://blog.csdn.net/cnclenovo/article/details/51954370 ） ，模拟windows系统）