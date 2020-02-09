---
title: 【问题解决】yarn安装node-sass失败
date: 2019-12-24 15:36:53
tags: node
---

记录解决react项目给中安装node-sass依赖失败的原因和方法

<!-- more -->

node-sass下载在国内受到限制，如果不能科学上网，一般通过设置淘宝镜像：

* yarn：

        yarn config set registry https://registry.npm.taobao.org -g
        yarn config get registry // 查看设置的镜像地址

* npm：

        npm config set sass_binary_site https://npm.taobao.org/mirrors/node-sass/
        npm config get registry // 查看设置的镜像地址

使用上述解决办法基本就可以解决node-sass下载不成功的问题。

但是我在设置好淘宝镜像后，下载node-sass过程中仍然提示失败，查看报错信息，提示node版本和node-sass版本不匹配，[点击这里][1]查看不同版本node-sass所对应的node版本。

我的node版本安装的是10.15.3，项目中node-sass的版本是4.7.2，但是4.7.2版本并不支持10.x.x版本的node，所以只好更改node版本。

当然，我不可能把node卸载了，再装一个低版本的，这样太麻烦。推荐使用nvm管理电脑上的node版本，[nvm-windows下载地址][2]。Mac可以使用[nvm][3]或者[n][4]，[n][4]不支持windows系统。

nvm的使用请参考[这篇文章][5]。

配置npm源还有一种方式是使用nrm。安装nrm并设置淘宝源：

    npm i -g nrm
    nrm use taobao

使用`npm ls`查看其他源的地址：

    nrm ls

      npm -------- https://registry.npmjs.org/
      yarn ------- https://registry.yarnpkg.com/
      cnpm ------- http://r.cnpmjs.org/
    * taobao ----- https://registry.npm.taobao.org/
      nj --------- https://registry.nodejitsu.com/
      npmMirror -- https://skimdb.npmjs.com/registry/
      edunpm ----- http://registry.enpmjs.org/


[1]: https://github.com/sass/node-sass/tags
[2]: https://github.com/coreybutler/nvm-windows/releases
[3]: https://github.com/nvm-sh/nvm
[4]: https://github.com/tj/n
[5]: https://www.jianshu.com/p/17d3249e0619
