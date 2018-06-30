---
title: 'Jenkins+Github+eggjs+alinode:全家桶'
date: 2018-06-28 13:55:38
tags: [CICD, NodeJs, egg.js, Web, 性能监控]
---

CI/CD(持续集成/持续发布)是近几年来软件工程领域，也是DevOps中很流行的一种方法。通过自动的持续集成和发布，省去了大量人工测试和部署时间，的大幅降低了软件开发迭代的难度，提高了开发速度。
![thumb](https://archive.steve.ly/live/im_///d1.awsstatic.com/product-marketing/CodeDeploy/Partners/jenkins.069c3fa4bd28d680e13e191fddb65d1e8383161d.png)

<!-- more -->

# Jenkins
CI/CD界的两大豪杰：闭源派Atlassian家的Bamboo以及开源派的Jenkins。自己搭建CI/CD平台做测试当然是选择开源软件Jenkins，Jenkins基于Java开发，所以在各个平台都能运行。

## Jenkins安装
Jenkins在多个平台都可以使用包管理工具安装，也可以下载war包给web服务器运行。这里举CentOS7.0的例子：

* pre-installation: java
> yum install java

* 使用yum安装Jenkins
> sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo 
> sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key 
> yum install jenkins 

* 启动服务
> sudo service jenkins start

* 防火墙设置： 由于CentOS7.0更新了新的防火墙机制，并且默认端口是关闭的，我们必须打开8080端口
> firewall-cmd --zone=public --add-port=8080/tcp --permanent 
> firewall-cmd --reload

访问ip地址下的8080端口，即可看到Jenkins的欢迎界面，按照向导操作即可进入。
![figure1](http://ohvmg8dgt.bkt.clouddn.com/jenkins1.png)

## 项目建立
新建项目并选择“自由风格的软件”，我使用了github作为项目代码仓库，在源码管理一栏中选择Git,并填入项目github的地址即可。如果使用https协议，则Credentials填github账号密码；如果使用ssh则填入github的私钥。

![figure2](http://ohvmg8dgt.bkt.clouddn.com/jenkins2.png)

构建触发器我使用GitHub钩子触发器
> GitHub hook trigger for GITScm polling

这一选项的具体设置在下一节会详细叙述。我准备为一个egg.js（基于NodeJs的Web应用）项目做CI/CD，所以构建环境中选择Node环境。这一步时可能找不到和Node有关的选项，我们需要回到全局设置的“全局工具配置”中，填写一个NodeJs的环境。

![figure3](http://ohvmg8dgt.bkt.clouddn.com/jenkins3.png)

# Github Webhook

配置github webhook可以实现每次pull之后，github能够自动触发钩子，发送指定请求。

首先在全局配置里勾选启用github Webhook，找到Jenkins的webhook url。
![figure-webhook](http://ohvmg8dgt.bkt.clouddn.com/jenkins-webhook.png)

打开github对应项目仓库的Settings，进入Webhooks页面新建一个webhook，并在Payload URL中填入刚刚在Jenkins中找到的webhook url。这样github的Webhook就设置好了，每次有代码pull到github，都会触发一次构建。

# Egg.js
阿里的nodejs服务器框架，在koa上封装并做了一些修改，找到官方教程创建一个初始项目模板。

# alinode
最后一步，加入监控。egg.js官方推荐自家的免费[Node.js性能平台](https://www.aliyun.com/product/nodejs)，参照官方教程对项目进行接入。对于egg.js项目来说，接入比较简单，使用egg.js的官方插件，再写一些配置即可。但是前提是需要安装alinode和agenthub。

## 申请app
在[Node.js性能平台](https://www.aliyun.com/product/nodejs)点击使用并申请一个app，即可获得一个`appId`和对应的`app Secret`。

## 全局安装 alinode runtime
但是我在使用alinode本地安装监控所需控件时失败，报错信息为404 Not Found。不知道是仓库未更新还是阿里放弃了本地的空间安装包。最后无奈全局安装：
> wget -q https://raw.githubusercontent.com/aliyun-node/alinode-all-in-one/master/alinode_all.sh \
> bash -i alinode_all.sh

安装后使用`which node`和`which agenthub`查看node和agenthub路径是否包含`.tnvm`，即验证是否已经被替换未阿里的版本。

## 安装 egg-alinode 插
接下来安装egg-alinode插件:
> npm i egg-alinode --save

## 在 egg 项目中启用插件
egg启用插件的配置在config/plugin.js中：
```js
exports.alinode = {
    enable: true,
    package: 'egg-alinode'
}
```

## 在 egg 项目中添加监控配置
```js
...
module.exports = appinfo => {
    const config = exports = {};

    exports.alinode = {
        appid: 'your appId',
        secret: 'your secret',
        packages: [
            path.join(appInfo.baseDir, 'package.json'),
        ]
    };
    ...
};

```
注意此时的`node`已经被替换成了`alinode`，修改jenkins的构建代码为运行`alinode`而非jenkins安装的`node`。
> /root/.tnvm/versions/alinode/v3.11.3/bin/npm stop 
npm run autod 
npm i 
npm test 
/root/.tnvm/versions/alinode/v3.11.3/bin/npm start

（具体路径以自己的情况为准）

看一下监控页面
![figure4](http://ohvmg8dgt.bkt.clouddn.com/jenkins4.png)
![figure5](http://ohvmg8dgt.bkt.clouddn.com/jenkins5.png)
![figure6](http://ohvmg8dgt.bkt.clouddn.com/jenkins6.png)

到此，整个项目的CI/CD+Monitor搭建就大功告成了！

