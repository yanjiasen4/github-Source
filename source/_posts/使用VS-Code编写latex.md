---
title: 使用VS Code编写latex
date: 2017-08-01 13:26:34
tags: [latex,vscode,ide]
categories: essay
---

![pic](http://ohvmg8dgt.bkt.clouddn.com/latexvscodeIcon.jpg)
第三方插件一直是新一代编辑器的核心功能，vscode也得益于其强大的插件。从初学LaTex到完成自己的本科毕业论文，我一直在使用CTex自带的编辑器WinEdt。最近闲来无事，在vscode上也折腾了一番，把自己的Latex书写环境搬运到了上面。

<!-- more -->

# Latex
Latex是一种基于Tex的排版系统。很长一段时间内对这种Tex有些混淆，前一段看了一些资料才渐渐搞懂。详细介绍请参见[这篇博客](http://blog.csdn.net/lnxfei/article/details/43900305)。我通常使用xeTex编译Latex来输出pdf文件，这篇文章也将按照这个模式来讲解。

# 插件配置
## Latex插件
配置Latex环境的插件我选择了[LaTex Workshop](https://marketplace.visualstudio.com/items?itemName=James-Yu.latex-workshop#review-details)。它包含了对.tex .bib .cls等文件的识别以及Latex语法的Linting，还实现了一些快捷方式来帮助你在VS Code中快速编译生成pdf或是开启一个tab浏览生成的pdf。


在VS Code的扩展（Ctrl+Shift+X）中搜索Latex，第一个插件就是LaTex Workshop。安装完成后，打开任意.tex文件后，会发现在编辑器左下角状态栏多了一个小标志


![pic](http://ohvmg8dgt.bkt.clouddn.com/latexworkshopIcon.png)

点击这个标志会出现一些选项

![pic](http://ohvmg8dgt.bkt.clouddn.com/selection.jpg)

* build LaTex project: 对当前tex文件进行一次编译
* view PDF file in web page/new tab: 开一个网页/新的tab浏览生成的pdf
* SyncTex from cursor: SyncTex实现了PDF和source的反向查询，可以快速找到当前光标位置源码对应的PDF部分
* Clean up auxiliary files: 清理生成PDF过程中的一些临时文件
* Open citation browser: 引用浏览器，挺方便的
* Count words in LaTex document: 单词计数，需要Perl解释器
* Show LaTex log: 显示LaTex的日志
* 后面几个是插件支持的相关功能，这里不再赘述

很显然，我们还不知道这个插件会用什么指令来编译tex文件，需要对编译指令进行设置。打开VS Code的文件/首选项/设置，修改`latex-workshop.latex.toolchain`的值即可。

```
"latex-workshop.latex.toolchain": [
  {
    "command": "xelatex",
    "args": [
      "--synctex",
      "--pdf",
      "--tex-option=\"-interaction=nonstopmode\"",
      "--tex-option=\"-file-line-error\"",
      "%DOC%.tex"
    ]
  }
]
```

为了使用.bib文件对引用进行管理，我的完整的编译链指令为

```
"latex-workshop.latex.toolchain": [
        {
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%"
            ]
        }, {
            "command": "bibtex",
            "args": [
                "%DOCFILE%"
            ]
        }, {
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%"
            ]
        }, {
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%"
            ]
        }
    ],
```

完整效果图
![pic](http://ohvmg8dgt.bkt.clouddn.com/test.jpg)

## Linting
LaTex workshop使用ChkTex作为Linting的工具，但是ChkTex没有官方对windows的支持，在windows上安装这个工具包会有一些麻烦。参考[ChkTex Manual](http://www.nongnu.org/chktex/ChkTeX.pdf)，还需要使用Cygwin之类的工具。而在Unix平台上安装ChkTex则方便许多。

## Spell Check
拼写检查是写文章必不可少的工具，这里推荐[Code Spellchecker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker)

LaTex workshop推荐的拼写忽略正则表达式，同样通过对VS Code的首选项设置实现。

```
"cSpell.ignoreRegExpList": [
  "\\\\\\w*(\\[.*?\\])?(\\{.*?\\})?",
  "\\$.+?\\$"
]
```

# 结语
这些插件已经可以让我在VS Code里愉快地写LaTex了，当然也有许多其他可选用的插件，VS Code包括其他的编辑器都建立了成熟的社区，有许多实用的插件有待我们的发掘。
