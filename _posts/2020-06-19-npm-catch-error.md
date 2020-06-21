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
npm ERR! code ENOGIT
npm ERR! No git binary found in $PATH
npm ERR! 
npm ERR! Failed using git.
npm ERR! Please check if you have git installed and in your PATH.
```

2. 错误2：
```
npm err! cb() never called!
```

### 解决方案
1. 错误1-解决方案：

如果是缺少 git 命令， 请参照如下 `dockerfile` 为你的node镜像添加`git`命令
```
## dockerfile
FROM node:12.2.0-alpine

# add git binary
RUN apk update && apk upgrade && \
    apk add --no-cache bash git openssh
```

2. 错误2-解决方案：
如果是依赖本地包的问题，请将移动至docker可以识别的目录；


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

When the global `package-lock=false` setting is active, you can still force a project’s package-lock.json file to be updated by running: `npm instal --package-lock`

This command is the only surefire way of forcing a package-lock.json update.


* The `--no-optional` argument will prevent optional dependencies from being installed.

* The `--no-package-lock` argument will prevent npm from creating a package-lock.json file. When running with package-lock’s disabled npm will not automatically prune your node modules when installing.

* The `--nodedir=/path/to/node/source` argument will allow npm to find the node source code so that npm can compile native modules.


## 参考 

1. [npm config](https://docs.npmjs.com/misc/config#npmrc-files)
2. [npm install](https://docs.npmjs.com/cli/install)