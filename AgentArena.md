# Agent竞技场

## Agent流程
```text
	AgentLogic : 在登陆时，进行模块的创建
	ModularMgr : ModularMgr.ModuleInitByData
	PubGameSrvObject : 主要用于方便管理 玩法服务 注册到相关公共玩法服务管理器
	ArenaMgr : 竞技场的数据初始化创建
```

## Agent数据结构

```text
	ArenaOdm: [
		PlayerInfos = {}
		seasonEndMs
		battleInfos
		records
		recordings
		sensonRankRecord
		firstShareRecordIds
	]
	ArenaPlayerOdm: [
		基础的玩家信息
		积分奖励
		battleRecords
	]
```

## 学习模块  PubGameSrvObject
```text
	Class("PubGameSrvObject",GameObject)   --用于方便管理 玩法服务 注册相关的公关玩法服务器

	在启动节点的时候，现将PubGame模块进行启动注册，之后将各个玩法进行创建初始化，注册出其他玩法的服务

	创建出ArenaMgr这样一个服务Srv

	在玩法服务像PubGame服务注册时，回调ArenaMgr.Init()进行ArenaModule初始化

```

## Arena 代码逻辑
### RPC接口
```text
	Arena_GetInfo(isOpen)
		info  ==>  Agent 通过 ArenaSnsa 拿到PubGame路径下的创建的数据 
			```lua
			Arena_GetInfo(playerId,isCreate:true,isOpen) 
			客户端主动吊起时，需要返回给信息给前端
			local curPlayerInfo = {} 拼接角色数据，若不存在则创建
			ArenaMgr:InitToRank(curPlayerInfo)  --将数据更新到排行榜
			```
	Arena_GetChallengeList
		question:ArenaMgr:FindChallengeList(playerId,findList,rateLower,rateUpper,matchMapInfo,challengeList,pos - #challengeList)   
			pos - #challengeList  ===>  1  ?
	Arena_Challenge
		{ 1、挑战次数判断 2、Arena的挑战封装到BattleArrayHandler }
	Arena_BuyTicket
		Agent下直接购买后，更新到节点上的Arena_UpdateBuy,更新竞技场数据
	Arena_GetTicketInfo
		Arena_GetInfo(isOpen).ticketLimit
	Arena_GetHeroInfos
		获取敌方防御阵容(1、机器人 2、玩家(获取玩家的竞技场防守阵容))
	Arena_GetBattleRecordInfo

	Arena_GetScoreReward

	Arena_GetScoreRewardList
```

## Attention
```text
	1:sucss:boolean 2:ret = xpcall()
```

## Summary
```text
	通过学习这个单服竞技场模块，我明白了，模板服务的创建和初始化顺序，以及调用关系，在以后的代码开发中，在Agent上实现相关单服逻辑，主要是得根据策划所需完成Agent与单服节点的信息通信。因为竞技场的多人互动性，所以需要将游戏数据放入单服节点处，供玩家操作时进行交互更改数据。在PubGame创建竞技场数据信息时，设计时将竞技场数据，和竞技场玩家对象分开设计存放，便于前后端的数据进行交互，也便于在书写代码时，之后的逻辑维护。可以通过相应的竞技场数据的字段精准定位到竞技场玩家对象的玩家个人数据。
```

## BattleHandler
```text
	BattleModule.BeginBattleFight()   ====>  获取战斗结果
	{	
		local fightData = battleComp:PackPlayerFightData()
		{
			--拼接战斗数据
			battelHandler:BattleModule.GetBattleArrayHandler
			通过BaseBattleArrayHandler：PackPlayerFightData(playerObj,fightId,gamePlayType,extraTb)
				{
					所有战斗数据的入口，玩法的独特玩法通过extraTb传,
						self:PackLeftFightData(playerObj, fightData, extraTb)
						self:PackRightFightData(playerObj, fightId, fightData, extraTb)
						self:PackInitRageData(fightId, fightData.fightSideDatas)
						self:PackEndFightData(playerObj, fightData, extraTb)
						self:PackCombatRestraint(fightData)
						self:PackGamePlayInfo(gamePlayType,extraTb.gamePlayInfo)
				}
		}
		return  battleComp:StarFight(playerObj,fightData)
	}
```
```lua
self:PackLeftFightData(playerObj, fightData, extraTb)

```

						self:PackRightFightData(playerObj, fightId, fightData, extraTb)
						self:PackInitRageData(fightId, fightData.fightSideDatas)
						self:PackEndFightData(playerObj, fightData, extraTb)
						self:PackCombatRestraint(fightData)
						self:PackGamePlayInfo(gamePlayType,extraTb.gamePlayInfo)
						