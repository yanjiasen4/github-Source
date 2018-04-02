---
title: Dota2-Bot深入了解和Debug
date: 2016-12-16 15:33:59
tags: [AI,Lua,Dota2]
categories: AI
---

![pic1](http://ohvmg8dgt.bkt.clouddn.com/1024x768_84fd96ab1bd01b6.jpg)
上一篇[Dota2-Bot编写探索](http://yanjiasen4.tech/2016/12/16/Dota2-Bot%E7%BC%96%E5%86%99%E6%8E%A2%E7%B4%A2/)大体介绍了一下Dota2 Bot脚本的总体设计和结构，
不过我意识到，仅仅靠这些，想写出一个像样子的Bot是远远不够的。  

从之前的阅读可以发现，Dota2的Bot脚本是“重写”机制的，你没有实现的部分将又默认系统实现，这就要求我们对系统自带的AI有所了解。当然，对一些相关API的了解也是基础。  

<!--more-->

# 代码分析
## 技能和物品使用主接口
ability_item_usage_lina.lua  
```Lua
castLBDesire = 0;
castLSADesire = 0;
castDSDesire = 0;

function AbilityUsageThink()

	local npcBot = GetBot();

	-- Check if we're already using an ability
	if ( npcBot:IsUsingAbility() ) then return end;

	abilityLSA = npcBot:GetAbilityByName( "lina_light_strike_array" );
	abilityDS = npcBot:GetAbilityByName( "lina_dragon_slave" );
	abilityLB = npcBot:GetAbilityByName( "lina_laguna_blade" );

	-- Consider using each ability
	castLBDesire, castLBTarget = ConsiderLagunaBlade();
	castLSADesire, castLSALocation = ConsiderLightStrikeArray();
	castDSDesire, castDSLocation = ConsiderDragonSlave();

	if ( castLBDesire > castLSADesire and castLBDesire > castDSDesire ) 
	then
		npcBot:Action_UseAbilityOnEntity( abilityLB, castLBTarget );
		return;
	end

	if ( castLSADesire > 0 ) 
	then
		npcBot:Action_UseAbilityOnLocation( abilityLSA, castLSALocation );
		return;
	end

	if ( castDSDesire > 0 ) 
	then
		npcBot:Action_UseAbilityOnLocation( abilityDS, castDSLocation );
		return;
	end

end
```

## Lua语言
lina的技能和物品使用AI脚本。首先介绍一下Lua语言吧，作为号称最轻量级的脚本语言，Lua通常被用在游戏、nginx中。语言本体是C实现的，语言风格也是类C的。还有一些基础特性值得注意  
*  区分大小写，跟C一样没什么好说的。
*  变量默认为全局，除非加了`local`关键字限定。
*  同javascript一样，Lua对变量类型也是隐式声明的，其实Lua的变量分8种类型，并且其中的数字Number实际都是double类型。  
*  函数长得跟javascript也很像，返回值可以多个。  
*  不等的判断是~=

不过不用担心，对Lua不熟悉也没关系，配合百度简单的使用问题不大。下面我们看看这个示例代码  

## 如何让Bot选择释放的技能
```Lua
castLBDesire = 0;
castLSADesire = 0;
castDSDesire = 0;
```
三个变量的定义，分别代表莉娜对其三个技能释放的~~欲望值~~意愿值(Desire)，其实就是根据一系列对当前局势的判断估算出某个技能被释放的价值是多少。然后就是关键的`AbilityUsageThink`函数了，这个函数每一帧会调用一次，Bot技能的释放将在这里被处理。  

```Lua
function AbilityUsageThink()

	local npcBot = GetBot();

	-- Check if we're already using an ability
	if ( npcBot:IsUsingAbility() ) then return end;

	abilityLSA = npcBot:GetAbilityByName( "lina_light_strike_array" );
	abilityDS = npcBot:GetAbilityByName( "lina_dragon_slave" );
	abilityLB = npcBot:GetAbilityByName( "lina_laguna_blade" );

    -- Consider using each ability
	castLBDesire, castLBTarget = ConsiderLagunaBlade();
	castLSADesire, castLSALocation = ConsiderLightStrikeArray();
	castDSDesire, castDSLocation = ConsiderDragonSlave();

```

使用局部变量`npcBot`并调用`GetBot`得到LinaBot的实例。然后先检查当前帧是否正在释放技能。如果没有，则通过`GetAbilityByName`接口得到莉娜的三个技能对象，在下面可能的释放中会用到。然后调用三个函数得到三个技能各自的释放期望  
```Lua
if ( castLBDesire > castLSADesire and castLBDesire > castDSDesire ) 
	then
		npcBot:Action_UseAbilityOnEntity( abilityLB, castLBTarget );
		return;
	end

	if ( castLSADesire > 0 ) 
	then
		npcBot:Action_UseAbilityOnLocation( abilityLSA, castLSALocation );
		return;
	end

	if ( castDSDesire > 0 ) 
	then
		npcBot:Action_UseAbilityOnLocation( abilityDS, castDSLocation );
		return;
	end

end
```
根据上面得到的期望值，设置机器人莉娜的技能释放。这个函数的写法比较regular，对每个技能做一次价值的估算，然后释放价值最大的那个或是选择不用技能。具体到某个技能的决策，则就要根据技能的特点来写了
```Lua
function ConsiderDragonSlave()

	local npcBot = GetBot();

	-- Make sure it's castable
	if ( not abilityDS:IsFullyCastable() ) then 
		return BOT_ACTION_DESIRE_NONE, 0;
	end;

	-- If we want to cast Laguna Blade at all, bail
	if ( castLBDesire > 0 ) then
		return BOT_ACTION_DESIRE_NONE, 0;
	end

	-- Get some of its values
	local nRadius = abilityDS:GetSpecialValueInt( "dragon_slave_width_end" );
	local nCastRange = abilityDS:GetCastRange();
	local nDamage = abilityDS:GetAbilityDamage();

	--------------------------------------
	-- Mode based usage
	--------------------------------------

	-- If we're farming and can kill 3+ creeps with LSA
	if ( npcBot:GetActiveMode() == BOT_MODE_FARM ) then
		local locationAoE = npcBot:FindAoELocation( true, false, npcBot:GetLocation(), nCastRange, nRadius, 0, nDamage );

		if ( locationAoE.count >= 3 ) then
			return BOT_ACTION_DESIRE_LOW, locationAoE.targetloc;
		end
	end

	-- If we're pushing or defending a lane and can hit 4+ creeps, go for it
	if ( npcBot:GetActiveMode() == BOT_MODE_PUSH_TOWER_TOP or
		 npcBot:GetActiveMode() == BOT_MODE_PUSH_TOWER_MID or
		 npcBot:GetActiveMode() == BOT_MODE_PUSH_TOWER_BOTTOM or
		 npcBot:GetActiveMode() == BOT_MODE_DEFEND_TOWER_TOP or
		 npcBot:GetActiveMode() == BOT_MODE_DEFEND_TOWER_MID or
		 npcBot:GetActiveMode() == BOT_MODE_DEFEND_TOWER_BOTTOM ) 
	then
		local locationAoE = npcBot:FindAoELocation( true, false, npcBot:GetLocation(), nCastRange, nRadius, 0, 0 );

		if ( locationAoE.count >= 4 ) 
		then
			return BOT_ACTION_DESIRE_LOW, locationAoE.targetloc;
		end
	end

	-- If we're going after someone
	if ( npcBot:GetActiveMode() == BOT_MODE_ROAM or
		 npcBot:GetActiveMode() == BOT_MODE_TEAM_ROAM or
		 npcBot:GetActiveMode() == BOT_MODE_GANK or
		 npcBot:GetActiveMode() == BOT_MODE_DEFEND_ALLY ) 
	then
		local npcTarget = npcBot:GetTarget();

		if ( npcTarget ~= nil ) 
		then
			if ( CanCastDragonSlaveOnTarget( npcTarget ) )
			then
				return BOT_ACTION_DESIRE_MODERATE, npcEnemy:GetLocation();
			end
		end
	end

	return BOT_ACTION_DESIRE_NONE, 0;

end
```

上面的是莉娜对技能龙破斩释放的释放价值计算函数`ConsiderDragonSlave`。这个函数会返回释放该技能的意愿程度和一些与释放技能相关的信息（如技能目标，位置等）。意愿程度从无到最高分为：
* BOT_ACTION_DESIRE_NONE
* BOT_ACTION_DESIRE_LOW
* BOT_ACTION_DESIRE_MODERATE
* BOT_ACTION_DESIRE_HIGH

![pic2](http://ohvmg8dgt.bkt.clouddn.com/Dragon_Slave_icon.png)

还记得上面所说的Lua的特性吗，变量默认情况下都是全局的，所以这里第六行可以直接使用在`AbilityUsageThink`中定义的变量`abilityDS`。首先判断技能能否被释放，就是是不是没有在转CD状态。
然后这个AI被设置为如果有放大招灭神斩的意图，就让选择放弃这个技能的决策，直接返回，优先释放大招。  

![pic3](http://ohvmg8dgt.bkt.clouddn.com/Laguna_Blade_icon.png)  

（你是大招了不起啊？！）  

其实这样设置有一定的道理，在对灭神斩的决策函数`ConsiderLagunaBlade`中，有两个释放神灭斩的条件：  
```Lua
-- If a mode has set a target, and we can kill them, do it
local npcTarget = npcBot:GetTarget();
if ( npcTarget ~= nil and CanCastLagunaBladeOnTarget( npcTarget ) )
then
	if ( npcTarget:GetActualDamage( nDamage, eDamageType ) > npcTarget:GetHealth() and UnitToUnitDistance( npcTarget, npcBot ) < ( nCastRange + 200 ) )
	then
		return BOT_ACTION_DESIRE_HIGH, npcTarget;
	end
end
```
如果当前模式设置的目标能够被神灭斩秒杀  
```Lua
-- If we're in a teamfight, use it on the scariest enemy
local tableNearbyAttackingAlliedHeroes = npcBot:GetNearbyHeroes( 1000, false, BOT_MODE_ATTACK );
if ( #tableNearbyAttackingAlliedHeroes >= 2 ) 
then

	local npcMostDangerousEnemy = nil;
	local nMostDangerousDamage = 0;

	local tableNearbyEnemyHeroes = npcBot:GetNearbyHeroes( nCastRange, true, BOT_MODE_NONE );
	for _,npcEnemy in pairs( tableNearbyEnemyHeroes )
	do
		if ( CanCastLagunaBladeOnTarget( npcEnemy ) )
		then
			local nDamage = npcEnemy:GetEstimatedDamageToTarget( false, npcBot, 3.0, DAMAGE_TYPE_ALL );
			if ( nDamage > nMostDangerousDamage )
			then
				nMostDangerousDamage = nDamage;
				npcMostDangerousEnemy = npcEnemy;
			end
		end
	end

	if ( npcMostDangerousEnemy ~= nil )
	then
		return BOT_ACTION_DESIRE_HIGH, npcMostDangerousEnemy;
	end
end
```
如果在打团，就对最后威胁的敌人释放。  

好像跑题了......咳咳,我们继续看龙破斩。  
```Lua
-- If we're pushing or defending a lane and can hit 4+ creeps, go for it
	if ( npcBot:GetActiveMode() == BOT_MODE_PUSH_TOWER_TOP or
	     npcBot:GetActiveMode() == BOT_MODE_PUSH_TOWER_MID or
	     npcBot:GetActiveMode() == BOT_MODE_PUSH_TOWER_BOTTOM or
	     npcBot:GetActiveMode() == BOT_MODE_DEFEND_TOWER_TOP or
	     npcBot:GetActiveMode() == BOT_MODE_DEFEND_TOWER_MID or
	     npcBot:GetActiveMode() == BOT_MODE_DEFEND_TOWER_BOTTOM ) 
	then
	     local locationAoE = npcBot:FindAoELocation( true, false, npcBot:GetLocation(), nCastRange, nRadius, 0, 0 );

	     if ( locationAoE.count >= 4 ) 
	     then
	        return BOT_ACTION_DESIRE_LOW, locationAoE.targetloc;
	     end
         end
```

嗯，收兵D，能杀四个以上才放，没毛病。  

```Lua
-- If we're going after someone
	if ( npcBot:GetActiveMode() == BOT_MODE_ROAM or
		 npcBot:GetActiveMode() == BOT_MODE_TEAM_ROAM or
		 npcBot:GetActiveMode() == BOT_MODE_GANK or
		 npcBot:GetActiveMode() == BOT_MODE_DEFEND_ALLY ) 
	then
		local npcTarget = npcBot:GetTarget();

		if ( npcTarget ~= nil ) 
		then
			if ( CanCastDragonSlaveOnTarget( npcTarget ) )
			then
				return BOT_ACTION_DESIRE_MODERATE, npcEnemy:GetLocation();
			end
		end
	end
```

在ROAM、TEAM_ROAM、GANK和DEFEND_ALLY模式下，如果能够对目标释放龙破斩，就返回一个中等级别的释放请求。

## 总结
到这里，每一帧Bot对技能的决策就清楚了：
1.  调用单个技能的Consider（意愿）函数，获得它们的意愿值
2.  通过调用主接口`AbilityUsageThink`根据第一步获得的每个技能的意愿值决定技能的释放  

当然这只是根据玩家的一些经验，对Bot做出的设置，还有许多可以优化的地方。而且最重要的一点是，我们写了一个自认为不错的Bot逻辑，但在实战中出现的各种复杂状况，很有可能让这个逻辑变得不符合逻辑。要解决这个问题，就需要Debug。

# Debuging

## 最愉悦也最痛苦的Debug
一般程序最简单的Debug过程就是run一遍，或是设置一些断点，单步执行，gdb打印一些关键值。总结起来就是运行（部分）、观察（可能出错的）变量。  

对Dota2 Bot脚本的Debug也不例外，你需要进游戏跟你的Bot脚本站在Dota2的战场上，也就是run你的脚本，才能进行Debug。而观察这一步则可以通过dota2提供的`控制台/console`来实现。  

在Dota2游戏中自建一个房间填充AI，然后进入游戏，按'\\'建打开控制台，输入`dota_bot`即可看到几个相关指令。  

### dota_bot_debug_team   
输入这个指令加上空格和一个数字（2表示天辉、3表示夜魇）会在屏幕的左上角开启一个数据面板，可以看到许多有用的信息，包括：
1. 团队级别中，对推塔、刷钱、防守和打肉山的意愿等级
2. Bot的名字和等级
3. Bot当前的最大'power' levels，关于什么是power levels后面会讲到。
4. 当前被选中的模式和改模式的意愿值
5. 一个Bot的所有模式的意愿值
6. （如果存在）当前Bot的目标
7. 每个Bot“思考”的时间和总共“思考”的时间  
![pic3](http://ohvmg8dgt.bkt.clouddn.com/dota_bot_1.png)  
（这个Lanning Desire翻译成“对线欲望”是什么鬼）

### dota_bot_debug_gird\dota_bot_debug_minimap  
这个两个指令会在地图上现实一些辅助线以助调试，指令后面的参数决定显示的内容：
- 0- 关闭
- 1 - 用红色标记天辉Bot应当回避的区域
- 2 - 用红色标记夜魇Bot应当回避的区域
- 3 - 天辉敌人的潜在位置
- 4 - 夜魇敌人的潜在位置
- 5 - 天辉敌人的视野
- 6 - 夜魇敌人的视野
- 7 - 高度图（就是地图的高度）
- 8 - 通过性，显示一些不能通过的地区  

### dota_bot_select_debug
对选中的Bot显示额外信息。大概就是这个效果  
![pic4](http://ohvmg8dgt.bkt.clouddn.com/dota_bot_2.png)  
可以看到对面宙斯身上有几行字，不过这字体和颜色实在费眼睛......  

### dota_bot_select_debug_attack  
显示某个Bot对附近的单位的攻击意愿倾向，需要注意的是这两个"select"的指令生效貌似有延迟。

### dota_bot_debug_clear  
清楚上面两个指令的效果

### dota_bot_debug_lanes  
显示所有的“线”，包括双方的上中下三路。  

### dota_bot_debug_ward_locations  
显示Bot猜测可能有眼的位置（Bot会反眼？） 

## 其他概念
有了这些工具，我们的Debug之路会简单一些，但是上面提到了一些AI设计中的概念，我们必须了解。

### 潜在位置
Bot的Team每帧会计算对面消失英雄可能在的位置及其概率，称为潜在位置，每个可能在的位置用一个0到255的整数表示其可能性大小，并且这个位置对应着上面所说的gird网格。
![pic5](http://ohvmg8dgt.bkt.clouddn.com/dota_bot_3.png)  
小地图上的红色系区域即夜魇Bot战队猜测我可能所在的地区概率分布。  

这个猜测使用了`floodfill`算法，这里不再赘述，但是其仍旧有一些局限性：  
* 没有对TP或者对移动有帮助（如加速符、变狼、闪烁等）的技能考虑在内
* 计算不考虑英雄消失的原因或根据消失的位置进行预测（比如残血消失大概率是回家） 

### 英雄的“力量”Power
~~反正完美这样翻译的我不背锅~~  
刚刚提到在`dota_bot_debug_team`面板上有显示英雄的Power Levels这一数据，那么英雄的Power到底是什么呢？  

它是对一个英雄当前综合能力的一个粗略估计，我方哪个队友最有用（保了个废物），对面哪个英雄对你威胁最大（拉谁说话），许多决策都需要这一概念来帮助。那么英雄的能力值是如何被计算的呢？  
* 对对方的每个英雄单独计算
* 计算一个英雄5秒内的输出，对于有控制技能的英雄，加上这个控制的时间，有减速技能的英雄，加上减速时间的一半
* 这个输出包括技能，而技能的计算包括抬手、消耗、沉默状态等因素
* 伤害的计算还把攻击特效和debuffs计算在内
* 这个伤害值是假设平均地打在对方所有英雄身上的  

可以看出这个能力值的计算确实有些粗糙，但是在多数情况下，确实是一个简单而有效的指标。  

未完待续......


