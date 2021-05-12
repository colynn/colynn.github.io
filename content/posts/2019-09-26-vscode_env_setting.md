---
title: VSCode 开发环境设置(go python)
categories: ["教程"]
tags: ["python", "go", "tools"]
date: 2019-09-26
lastmod: 2020-09-15
---

# 运行环境
* Mac OS 10.13.6
* VS code 1.31.1

# 快捷键

![image](https://user-images.githubusercontent.com/5203608/89495603-34b84780-d7eb-11ea-94b5-d48d3480b679.png)

## 解读
1. General 
    * Command + Shift + p, F1  显示命令面板
    * Command + p 快速打开
    * Command + Shite + N  新建vscode 窗口

2. Integrated Terminal
    * Ctrl + ` 显示或隐藏集成终端；
    * Ctrl + Shift + ` 新建终端；

3. navigation

![image](https://user-images.githubusercontent.com/5203608/89495608-384bce80-d7eb-11ea-9b99-f1128c223182.png)

# 常用设置

## 推荐插件

| 插件名      | 作用 |
| ----------- | ----------- |
| gitlens      | git工具       |
| indent-rainbow   | 彩色缩进   |
| Bracket Pair Colorizer |  彩色括号 |
| Rainbow CSV       |   彩色csv |


## 1. 隐藏.pyc文件
* 调整Workspace Settings添加如下图，

![image](https://user-images.githubusercontent.com/5203608/89495618-3bdf5580-d7eb-11ea-81bf-21e2b49b2bb4.png)

__设置路径__：Code -> Preferences -> Settings -> Workspace Settings, Search Settings

## 2. 配置markdown-pdf 插件
* 安装说明:

    Chromium download starts automatically when Markdown PDF is installed and Markdown file is first opened with Visutal Studio Code.

    However, it is time-consuming depending on the environment because of its large size (~ 170Mb Mac, ~ 282Mb Linux, ~ 280Mb Win).

    During downloading, the message Installing Chromium is displayed in the status bar.

    If you are behind a proxy, set the http.proxy option to settings.json and restart Visual Studio Code.

    If the download is not successful or you want to avoid downloading every time you upgrade Markdown PDF, please specify the installed Chrome or 'Chromium' with markdown-pdf.executablePath option.
* 设置 markdown-pdf.executablePath
    ```js
    "markdown-pdf.executablePath": "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome",
    ```
* 保存后，重启vscode 即可。
* 如何使用自定义css
    ```js
    // 路径和css文件名以你的文件为准
    "markdown-pdf.styles": ["/.markdown/github.css"],
    ```
    自定义样式参考：http://markedstyle.com/styles
    
    __设置路径__：Code -> Preferences -> Settings

## 3. 配置golang 环境

### 1. [Deprecated]安装Go相关插件

__注意__: 请先设置 ```GOPROXY```

```sh
export GOPROXY=https://goproxy.io
```

推荐安装的插件包如下：
To install the tools manually in the current ```$GOPATH```, just copy the below (after choosing the tools) in your terminal run:

```sh
go get -u -v github.com/ramya-rao-a/go-outline
go get -u -v github.com/acroca/go-symbols
go get -u -v github.com/mdempsky/gocode
go get -u -v github.com/rogpeppe/godef
go get -u -v golang.org/x/tools/cmd/godoc
go get -u -v github.com/zmb3/gogetdoc
go get -u -v golang.org/x/lint/golint
go get -u -v github.com/fatih/gomodifytags
go get -u -v golang.org/x/tools/cmd/gorename
go get -u -v sourcegraph.com/sqs/goreturns
go get -u -v golang.org/x/tools/cmd/goimports
go get -u -v github.com/cweill/gotests/...
go get -u -v golang.org/x/tools/cmd/guru
go get -u -v github.com/josharian/impl
go get -u -v github.com/haya14busa/goplay/cmd/goplay
go get -u -v github.com/uudashr/gopkgs/cmd/gopkgs
go get -u -v github.com/davidrjenni/reftools/cmd/fillstruct
go get -u -v github.com/alecthomas/gometalinter
gometalinter --install
```

__注__: 你可以直接配置使用 [`gopls`](https://github.com/golang/vscode-go#support-for-go-modules), 并且强烈推荐使用。

### 2. 配置VS code 的 settings.json

* 相关go的配置如下：

```js
"go.useLanguageServer": true, //using language server
"go.autocompleteUnimportedPackages": true,
"go.inferGopath": true,
"go.docsTool": "godoc",
"go.gocodePackageLookupMode": "go",
"go.gotoSymbol.includeImports": true,
"go.useCodeSnippetsOnFunctionSuggest": false, //使用代码片段提示
"go.useCodeSnippetsOnFunctionSuggestWithoutType": true,
"go.formatTool": "goimports",  // 代码格式化
"go.buildOnSave": "off", //保存代码时自动编译
"go.lintOnSave": "file", //保存代码时优化
"go.vetOnSave": "package", //保存代码时检查潜在错误
"go.coverOnSave": false //保存代码时执行测试

```

__NOTE__: If you are using Go modules, then we strongly recommend using the Go language server as it performs much better than the tools below.

[Refer to: Go tools that the Go extension depends on](https://github.com/Microsoft/vscode-go/wiki/Go-tools-that-the-Go-extension-depends-on)


### 3. reload VS code, then enjoy it.


## 4. vscode golint 代码规范解读
### 警告1
* 描述：exported function xxx should have comment or be unexported
* 环境：
```sh
=> ~$ go version
go version go1.12.1 darwin/amd64
=> ~$ gopls version
version v0.1.3-cmd.gopls, built in $GOPATH mode
```
* 解决：

Comment SentencesSee https://golang.org/doc/effective_go.html#commentary. Comments documenting declarations should be full sentences, even if that seems a little redundant. This approach makes them format well when extracted into godoc documentation. Comments should begin with the name of the thing being described and end in a period:

记录声明的注释应该是完整的句子，即使这看起来有点多余。这种方法使它们在提取到 godoc 文档时格式良好。注释应以所述物品的名称开始，并以句号结束:

```sh
// Request represents a request to run a command.
type Request struct { ...
// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

### 示例警告
```sh
recorder.go:55:5: exported var RecordBind should have comment or be unexported
recorder.go:158:1: exported function Record_ErrorRecord should have comment or be unexported
recorder.go:173:6: don't use underscores in Go names; type Data_MemStats should be DataMemStats
recorder.go:179:2: struct field FreeRam should be FreeRAM
```

golint 会检测的方面：
* 变量名规范
* 变量的声明，像```var str string = "test"```，会有警告，应该```var str = "test"```
* 大小写问题，大写导出包的要有注释
* x += 1 应该 x++


## 5. Go tool to modify struct field tags faster
### how to use 
1. add tags

在对应的文件下，打开命令面板（Command + Shift + p）执行 `Go: Add Tags` 即可添加json field;

2. remove tags

在对应的文件下，打开命令面板（Command + Shift + p）执行 `Go: Remove Tags` 即可添加json field

[github gomodifytags](https://github.com/fatih/gomodifytags)

## 6. 配置python 环境

### python项目的 `setting.json`的示例文件

```js
{
    "python.pythonPath": "venv/bin/python3",

    // Formatting doesn't affect the functionality of the code itself. 
    "python.formatting.provider": "autopep8",
    "python.formatting.autopep8Args": ["--max-line-length", "120", "--experimental"],

    // Linting, on the other hand, analyzes code for common syntactical, stylistic, and functional errors as well as unconventional programming practices that can lead to errors.
    "python.linting.pylintEnabled": true,
    "python.linting.pylintArgs": ["--load-plugins", "pylint_django"],

    // https://pycodestyle.pycqa.org/en/latest/intro.html#configuration
    "python.linting.pycodestyleEnabled": true,
    "python.linting.pycodestyleArgs": ["--ignore=E501"],

    "python.testing.pytestArgs": [
        "venv"
    ],
    "python.testing.unittestEnabled": false,
    "python.testing.nosetestsEnabled": false,
    "python.testing.pytestEnabled": false,
    "editor.formatOnSave": true,
    "python.terminal.activateEnvironment": true, // Activate Python env in Terminal created using the Extension
}

```

### py推荐插件

* [Python Docstring Generator](https://github.com/NilsJPWerner/autoDocstring)

* [Python Extension Pack](https://github.com/DonJayamanne/python-extension-pack)

* [AREPL for python](https://github.com/almenon/arepl-vscode.git)

* [Python Test Explore](https://github.com/kondratyev-nv/vscode-python-test-adapter.git)

_注_：Refer to: https://code.visualstudio.com/docs/python/editing#_formatting
