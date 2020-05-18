---
title: react+ant搭建后台管理项目步骤记录
date: 2020-05-14 23:33:40
tags: react
---

利用create-react-app和antd搭建管理后台项目，记录搭建步骤。

<!-- more -->

本来是打算使用ant-design pro来进行项目初始化的，但是项目构建起来后发现ant-design pro是一个大而全的项目，很多组件也都做了封装，比如BasicLayoutProps组件，是高度封装的页面结构组件。个人感觉适合快速开发项目，但是修改的灵活性比较低。因此打算自己动手从零搭建一个项目。

## 项目初始化

初始化项目之前，首先要确定是用js还是ts开发，我用的是ts。

1. 初始化命令：参考[官方文档](https://zh-hans.reactjs.org/docs/static-type-checking.html#using-typescript-with-create-react-app)

    ```shell
    npx create-react-app my-app --template typescript
    ```
    注意：这里如果你的create-react-app版本比较低的话，会创建失败。这时需要把现在的create-react-app卸载掉，然后重新安装。

2. 配置代码规范

    主要是指配置项目中的点文件（例如.eslintrc）。我配置的有：
    .editorconfig、.eslintrc、.prettierrc、.babelrc

    这些文件的具体配置直接去看对应的官方文档即可。

3. 配置项目目录，我配置好的目录如下。
    ```
    ├── public
    │   ├── favicon.ico
    │   └── index.html
    ├── src
    │   ├── assets
    │       └── style
    │   ├── components
    │   ├── routes
    │   ├── pages
    │   ├── utils
    │   ├── App.css
    │   ├── App.js
    │   ├── App.test.js
    │   ├── index.css
    │   ├── index.js
    │   └── logo.svg
    ├── package.json
    ├── README.md
    └── yarn.lock
    ```
    目录配置完，配置reset.css和common.css文件，初始化项目基础样式。
4. 安装依赖

    这一步不可能一步到位，首次我安装的是antd和node-sass，node-sass是为了处理scss文件，因为样式开发采用的是scss。

    然后安装react-router-dom，为配置路由做准备。

    ts环境下，有时还需要安装第三方依赖的类型文件，例如@types/react-router-dom。

    redux还没确定用哪个方案，确定了再写。

## 开发

项目初始化完毕后，开始进入开发。
1. 配置入口页配置antd，配置路由文件，使用react-router-dom
2. reset.css

