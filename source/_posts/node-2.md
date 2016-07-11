title: node学习笔记(2)--包管理器npm
date: 2014-03-27 21:10:28
tags:
- 开始
- 我
- 日记
categories: node
---
### 简介

npm（node package manager），是node.js的一个包管理器，用于第三方模块的下载、安装和管理。
npm收录着庞大而丰富的第三方资源，
npm对于node.js的意义，比maven与java。

<!-- more -->
### 安装

	windows下 安装node 后 npm已经安装成功

	控制台输入npm

```bash
Usage: npm <command>

where <command> is one of:
    access, add-user, adduser, apihelp, author, bin, bugs, c,
    cache, completion, config, ddp, dedupe, deprecate, dist-tag,
    dist-tags, docs, edit, explore, faq, find, find-dupes, get,
    help, help-search, home, i, info, init, install,
    install-test, issues, it, la, link, list, ll, ln, login,
    logout, ls, outdated, owner, pack, ping, prefix, prune,
    publish, r, rb, rebuild, remove, repo, restart, rm, root,
    run-script, s, se, search, set, show, shrinkwrap, star,
    stars, start, stop, t, tag, team, test, tst, un, uninstall,
    unlink, unpublish, unstar, up, update, upgrade, v, verison,
    version, view, whoami

npm <cmd> -h     quick help on <cmd>
npm -l           display full usage info
npm faq          commonly asked questions
npm help <term>  search for help on <term>
npm help npm     involved overview

Specify configs in the ini-formatted file:
    C:\Users\Administrator\.npmrc
or on the command line via: npm <command> --key value
Config info can be viewed via: npm help config

npm@3.4.1 C:\Users\Administrator\AppData\Roaming\npm\node_modules\npm

```

### npm 使用

1、npm常用命令
npm init  会引导你创建一个package.json文件，包括名称、版本、作者这些信息等
npm install <name> 安装nodejs的依赖包
npm install <name> -g  将包安装到全局环境中
npm install <name> --save  安装的同时，将信息写入package.json中。项目路径中如果有package.json文件时，直接使用npm install方法就可以根据dependencies配置安装所有的依赖包
npm remove <name> 移除
npm update <name> 更新
npm ls  列出当前安装的了所有包
npm root  查看当前包的安装路径
npm root -g  查看全局的包的安装路径
npm help  帮助，如果要单独查看install命令的帮助，可以使用的npm help install

2、模式
npm有全局和本地两种模式。
本地模式是npm的默认模式，这种模式的工作范围仅限于当前的工作目录下，任何操作都不会影响电脑上的其他node.js代码。
```bash
npm install express
```

反之，全局模式是为电脑上所有的node.js项目服务的。

```bash
npm install -g express
```