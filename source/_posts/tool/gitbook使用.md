---
title: gitbook使用
tags:
  - gitbook
categories:
  - tool
date: 2019-03-19 21:48:04
---

# GitBook：借助 gitbook 工具创建一本书

## 创建步骤
### 1. 安装 [Node.js](https://nodejs.org/en/) 和 npm (Node.js 的安装包一般会包含 npm 的安装)；
### 2. 创建 gitbook 文件夹，并进到该文件夹：

   ```javascript
   $ mkdir /PATH/TO/gitbook
   $ cd /PATH/TO/gitbook
   ```
### 3. 安装 gitbook ：
   
   ```javascript
   $ npm install gitbook-cli -g
   ```
   添加全局配置，配置path
### 4. 创建你的书：
   
   ```javascript
   $ gitbook init
   ```
   上述命令行是会报错的，因为你的插件不是装在全局的；这时可用下面的命令行：
   ```javascript
   $ gitbook init
   ```
   
   init 成功之后会看到文件： `README.md` 和 `SUMMARY.md`。
### 5. 打开并编辑书目录文件 SUMMARY.md ：
    
   ```javascript
    * [Introduction](README.md)
	* [第一章：如何造火箭](ch1/build.md)
		* [1. 燃料学](ch1/fuel.md)
		* [2. 空气动力学](ch1/air.md)
		* [3. 总装工程学](ch1/enginer.md)
		* [小结](ch1/WRAPUP.md)
	* [第二章：如何回收火箭](ch2/recycle.md)
		* [1. 自动控制原理](ch2/ac.md)
		* [2. 二次利用要点](ch2/key.md)
		* [3. 三次利用要点](ch2/three.md)
		* [4. 四次利用要点](ch2/four.md)
	* [结束](end/SUMMARY.md)
   ```
   保存之后再执行下面的命令：
   
   ```javascript
   $ gitbook init
   ```
   你会发现 gitbook 为你建好了 ch1、ch2、end 三个文件夹，且把在 SUMMARY.md 列出来的 md 文件都建好放在了相应文件夹里。
   
   接下来我们只要对应的打开 md 文件填写我们的内容就好。
### 6. 预览一下书的样子：
   
   ```javascript
   $ gitbook serve
   ```
   执行成功之后会看到一个网址：
   
   ```javascript
   http://localhost:4000
   ```
   拷贝该网址在浏览器打开就可以预览书的样式了。
### 7. 将 md 文件 build 成 html 文件：
   
   ```javascript
   $ gitbook build
   ```
   执行成功之后你会看到多了一个 **_book** 文件夹，里面就是转换好的 html 文件。

### 8. 生成电子书
GitBook不仅可以生成静态网站，也可以将内容输出为电子书(ePub，Mobi，PDF)格式。

安装插件

```
苹果OS X系统

下载Caliber应用程序应用程序。将calibre.app移动到您的应用程序文件夹后，创建一个指向ebook-convert工具的软件链接：

sudo ln -s /Applications/calibre.app/Contents/MacOS/ebook-convert /usr/bin
这样就可以在任何目录下执行目录执行ebook-convert命令。

如果出现Operation not permitted异常，说明系统权限限制，需要配置环境变量的方式解决

sudo ln -s /Applications/calibre.app/Contents/MacOS/ebook-convert /usr/bin
ln: /usr/bin/ebook-convert: Operation not permitted
环境变量配置

先启动ebook-convert完成第一次启动配置，然后关闭。接着在命令行窗口修改环境配置文件，加入EBOOK_PATH（ebook-convert命令的所在目录）

vim ~/.bash_profile 

export EBOOK_PATH=/Applications/calibre.app/Contents/MacOS
export PATH=$PATH:$EBOOK_PATH
然后刷新一下刚刚的配置:

source ~/.bash_profile
最后测试一下ebook-convert指令是否能正常被调用：

$ ebook-convert --version
ebook-convert (calibre 2.81.0)
Created by: Kovid Goyal <kovid@kovidgoyal.net>
大功告成！下面就可以使用gitbook pdf ./ ./mybook.pdf 命令把你的项目生成pdf文档了！
```
```
＃生成PDF文件
$ gitbook pdf ./ ./mybook.pdf

＃生成ePub文件
$ gitbook epub ./ ./mybook.epub

＃生成Mobi文件
$ gitbook mobi ./ ./mybook.mobi
```



   
   
