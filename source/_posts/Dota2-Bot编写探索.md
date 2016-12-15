---
title: Dota2 Bot编写探索
date: 2016-12-16 01:14:21
tags: [AI,Lua,Dota2]
categories: AI
---

![pic1](http://ohvmg8dgt.bkt.clouddn.com/dota2_pic.png)
2016年的12月12日，对剁手党来说是一次普通的双二十，但是对dotaer来说，确实一次不寻常的节日。不会数数的G胖在这一天公布了dota2的7.0，直接从6.88跳跃至7.00。  

从这巨大的版本号增加就能感受出这次改动之大。但是身为程序猿，更新日志中最不起眼的“其他改动”中的一项，才最吸引了我的关注——社区原创机器人，也就是自定义AI。
Dota2自带的AI确实“智商”有限，这次V社把“锅”甩给了玩家，倒不是为一种明智之举。也许不久的将来，你就会被那个打dota不如你的学霸的AI所打败。  

<!--more-->

# 上手

## 简介
官方给出了Dota2开发者维基的连接，上手就从那里开始好了  

[Valve Developer Community](https://developer.valvesoftware.com/wiki/Main_Page)  

下面这个是关于bot的页面  

[Dota Bot Script](https://developer.valvesoftware.com/wiki/Dota_Bot_Scripting)

Dota2这次为我们开放的接口需要我们对一些Lua脚本进行编写。在游戏中已经有一套基础的AI系统（也就是现在开房所默认的Bot），我们脚本没有实现的将会由游戏默认的AI（用C++编写）来完成。
所以我们可以从小的功能开始，一步步完善自己的AI。  

## Levels —— 决策级别
从大的方向来说，Bot有三种决策级别（Levels），分别是：  
* Team Level   —— 团队
* Mode Level   —— 模式
* Action Level —— 动作  

团队级别的决策会影响所有Bot的决策，比如是推上塔还是打肉山或是带线。这个决策的制定跟单个Bot无关,也无法完全决定某个Bot的具体动作,它只会影响Bot的决策偏向。  

模式级别的决策是针对单个Bot的，某个Bot选择gank、推塔等大的行为路线将被模式级别的决策来决定。  

动作决策则会根据Bot当前的模式，进行相应的动作，细化到点击鼠标移动，技能的释放等等。  

整个决策的过程将会是这样：  
1. 根据当前整体局势，对局势的计算,在团队决策作出“指导意见”
2. 每个Bot接到团队级别的决策后，再根据自己当前的一些状态，评出得分最高的模式，作为自己的模式决策结果。
3. Bot根据模式决策结果进行具体的动作。  

## Directory Structure —— 目录结构
看上去清晰明了，但是貌似有点太High-level了，V社准备的也算齐全，在游戏目录下已经有一个Bot的实例工程了，就放在`game/dota/scripts/vscripts/bots_example`中，下面我们就接地气地看一下代码。  

![pic2](http://ohvmg8dgt.bkt.clouddn.com/dota2_pic1.png)  

前面说到，你可以只实现一部分AI，让默认的AI完成剩下的工作。比如上图第二和第三个.lua文件就分别是莉娜和斯温的技能与物品释放脚本。
而如果想完全重写（compelete takeover），也很简单，把以_generic.lua结尾的文件中的相应函数实现即可。  

类似的，几类文件构成了整个Dota2的Bot脚本：  
* ability_item_usage_XXX.lua  物品和技能释放
* hero_selection.lua 选英雄智能
* item_purchase_XXX.lua 购买物品
* mode_YYY_XXX.lua 某种特定模式决策的智能
* team_desire.lua 团队决策  

其中，XXX代表可以是一个特定英雄，或者是generic的；
YYY指模式名称，如：laning、attact、roam、retreat，在维基可以查到所有模式。  

未完待续......
