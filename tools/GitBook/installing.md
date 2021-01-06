# `GitBook` 安装使用教程

#### 本机配置
> 1. 系统: macOS Big Sur 11.0.1
> 2. 内存: 16GB
> 3. 安装时间: 2021-01-05

#### `GitBook` 简介
* [`GitBook` 官网](https://www.gitbook.com)
* [`GitBook` 文档](https://docs.gitbook.com)

`GitBook` 是一个基于 `Node.js` 的命令行工具，所以想要安装 `GitBook` 需要先安装 `NodeJs`

#### 准备工作
**熟悉** `Markdown` [语法教程](http://www.markdown.cn)


#### 开始安装

##### 安装 `NodeJs`

 1. 已安装，跳过此步骤
 2. 未安装，访问 [`NodeJS` 官网](https://nodejs.org/en/) 下载安装即可
 - 本机版本
 ```
 [~/gitbook] → node -v
 v14.15.4
 [~/gitbook] → npm -v
 6.14.10
 ```

##### 安装 GitBook
因为网络原因，建议使用淘宝源安装 GitBook 
 * 快速替换[淘宝源 `npm`](https://developer.aliyun.com/mirror/NPM?from=tnpm),使用命令 `npm config set registry https://registry.npm.taobao.org`
 * 可以使用 `npm config get registry` 查看源是否设置成功
 * 安装 `npm install gitbook-cli -g`
 * 通过 `gitbook -V` 查看 `GitBook` 版本并作为检测是否安装成功

```
[~/gitbook] → gitbook -V
CLI version: 2.3.2
Installing GitBook 3.2.3

# 第一次会有一个安装过程，会列出相关组件等
gitbook@3.2.3 ../../
├── escape-html@1.0.3
├── escape-string-regexp@1.0.5
├── destroy@1.0.4
├── ignore@3.1.2
├── bash-color@0.0.4
├── gitbook-plugin-livereload@0.0.1
├── cp@0.2.0
├── graceful-fs@4.1.4
├── nunjucks-do@1.0.0
...
```

###### tips
如果这个过程中出现如下问题
```
if (cb) cb.apply(this, arguments)
                 ^
TypeError: cb.apply is not a function
```
原因则是 `NodeJs` 版本问题，降低版本即可，个人使用版本为：
```
 [~/gitbook] → node -v
 v14.15.4
 [~/gitbook] → npm -v
 6.14.10
```

#### 首次使用
安装好，创建一个想要写书的目录：
```
[~] → mkdir book_name
[~] → cd book_name
[~/book_name] → gitbook init
warn: no summary file in this book
info: create README.md
info: create SUMMARY.md
info: initialization is finished
[~/book_name] → ls
total 16
-rw-r--r--  1 staff  staff    16B  1  5 20:16 README.md
-rw-r--r--  1 staff  staff    40B  1  5 20:16 SUMMARY.md
```

可以看到初始化之后会创建两个文件，其中 `README.md` 是说明文档，而 `SUMMARY.md` 是书的章节目录

接下来我们可以在书籍目录终端输入 `gitbook serve` 命令
```
[~/gitbook] → gitbook serve
Live reload server started on port: 35729
Press CTRL+C to quit ...

info: 7 plugins are installed
info: loading plugin "livereload"... OK
info: loading plugin "highlight"... OK
info: loading plugin "search"... OK
info: loading plugin "lunr"... OK
info: loading plugin "sharing"... OK
info: loading plugin "fontsettings"... OK
info: loading plugin "theme-default"... OK
info: found 1 pages
info: found 0 asset files
info: >> generation finished with success in 0.3s !

Starting server ...
Serving book on http://localhost:4000
```
然后在浏览器访问 `http://localhost:4000` 即可预览书籍

#### 插件
`GitBook` 还支持许多插件，可以从 `npm` 上搜索，一般命名方式为： 插件：`gitbook-plugin-X` ，主题：`gitbook-theme-X` 
[插件官网](https://plugins.gitbook.com)

希望使用插件需要在书籍目录(`README.md` 同层)创建 `book.json` (默认未创建)，并修改里面内容

修改完 `book.json` 文件后，终端输入命令 `gitbook install` 即可

附上 `book.json` 文件部分格式
```json
{
  "title": "标题",
  "author": "作者",
  "description": "描述简介",
  "language": "zh-hans",
  "gitbook": "3.2.3",
  "styles": {
    "website": "./styles/website.css"
  },
  "structure": {
    "readme": "README.md"
  },
  "plugins": [
    "-sharing",
    "expandable-chapters-small",
    "github",
    "github-buttons",
    "favicon"
  ],
  "pluginsConfig": {
    "github": {
      "url": "https://github.com/BestKind"
    },
    "github-buttons": {
      "buttons": [{
        "user": "BestKind",
        "repo": "glory",
        "type": "star",
        "size": "small",
        "count": true
      }
      ]
    },
    "sharing": {
      "douban": false,
      "facebook": false,
      "google": false,
      "hatenaBookmark": false,
      "instapaper": false,
      "line": false,
      "linkedin": false,
      "messenger": false,
      "pocket": false,
      "qq": false,
      "qzone": false,
      "stumbleupon": false,
      "twitter": false,
      "viber": false,
      "vk": false,
      "weibo": false,
      "whatsapp": false,
      "all": [
        "google", "facebook", "weibo", "twitter",
        "qq", "qzone", "linkedin", "pocket"
      ]
    },
    "anchor-navigation-ex": {
      "showLevel": false
    },
    "favicon":{
      "shortcut": "./source/images/favicon.jpg",
      "bookmark": "./source/images/favicon.jpg",
      "appleTouch": "./source/images/apple-touch-icon.jpg",
      "appleTouchMore": {
        "120x120": "./source/images/apple-touch-icon.jpg",
        "180x180": "./source/images/apple-touch-icon.jpg"
      }
    }
  }
}
```
