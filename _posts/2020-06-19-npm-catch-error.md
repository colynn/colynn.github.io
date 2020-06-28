---
layout: post
title: npm使用过程中的报错记录
subtitle:
tags: [vue, npm]
comments: true
---

## 1. 制做docker镜像时无法安装依赖

### 问题描述
比如你在package.json里指定了 对于本地包或是 git 包的依赖，然后你在本地直接打包(`npm run build`)是没有问题的，但如果现在你想构建镜像，那么这么非常规的依赖可能会成为你的障碍。

也许你会遇到类似如下的错误：
1. 错误1：
    ```
    npm err! cb() never called!
    ```
2. 错误2：
    ```
    npm ERR! code ENOGIT
    npm ERR! No git binary found in $PATH
    npm ERR! 
    npm ERR! Failed using git.
    npm ERR! Please check if you have git installed and in your PATH.
    ```

### 解决方案
1. 错误1-解决方案：
如果是依赖本地包的问题，请将移动至docker可以识别的目录；

2. 错误2-解决方案：

如果是缺少 git 命令， 请参照如下 `dockerfile` 为你的node镜像添加`git`命令
```
## dockerfile
FROM node:12.2.0-alpine

# add git binary
RUN apk update && apk upgrade && \
    apk add --no-cache bash git openssh
```

## 附录
1. nginx config 

```
-s, --silent: --loglevel silent
-q, --quiet: --loglevel warn
-d: --loglevel info
-dd, --verbose: --loglevel verbose
-ddd: --loglevel silly
```

2. nginx install

*  `--package-lock` By default, package-lock.json is updated whenever you run npm install. However, this can be disabled globally by setting `package-lock=false` in `~/.npmrc`.

When the global `package-lock=false` setting is active, you can still force a project’s package-lock.json file to be updated by running: 

```npm instal --package-lock```

This command is the only surefire way of forcing a package-lock.json update.


* The `--no-optional` argument will prevent optional dependencies from being installed.

* The `--no-package-lock` argument will prevent npm from creating a package-lock.json file. When running with package-lock’s disabled npm will not automatically prune your node modules when installing.

* The `--nodedir=/path/to/node/source` argument will allow npm to find the node source code so that npm can compile native modules.

* The `--registry=https://xx.taobao.org` 
基于指定仓库安装相关包

* Using locked packages
    * 注意使用`package-lock`与不使用`package-lock`并没有什么不同， 因为任何更新`node_modules` and/or `package.json`依赖关系的命令都将自动同步现有的`package-lock.json`。 命令包括（`npm install/ npm rm/ npm update`等等）
    * 你可以使用`--no-save`来完全阻止这个更新, 或者使用`--no-shrinkwrap`更新`package.json`，同时保持`package-lock.json`或`npm-shinshwrap.json`完好无损。

    * 另外也强烈推荐将 lock 文件加入版本控制, 这样均可以获取统一的依赖树（不论是团队成员、还是CI工具等）
* `--package-lock-only`  npm will automatically resolve any conflicts for you and write a merged package lock that includes all the dependencies from both branches in a reasonable tree.

## 参考 

1. [npm config](https://docs.npmjs.com/misc/config#npmrc-files)
2. [npm install](https://docs.npmjs.com/cli/install)
3. [pakcage locks](https://docs.npmjs.com/configuring-npm/package-locks)