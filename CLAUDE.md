# 水浒·天下争锋 — 开发文档

纯前端单文件回合制策略游戏。水浒题材，6 阵营争夺 28 据点。

## 快速启动
- 文件：`index.html`（约 3600 行，零依赖）
- 打开：双击或用浏览器直接打开
- 在线版：`https://myles60.github.io/shuihu-game/`

## 核心系统（按代码位置）

### 数据层
- `FACTIONS[]` — 8 阵营（朝廷/辽国/梁山泊/大名府/祝家庄/田虎/王庆/方腊）
- `LOCATIONS[]` — 28 据点（坐标 2200×1400 设计网格）
- `CONNECTIONS[]` — 据点间连线（road/mountain/water）
- `HERO_POOL[]` — 150 位好汉（含 4 位隐藏彩蛋：贾诗涵/张浩宇/张永旭/麻昇）
- `FORMATIONS[]` — 4 阵型（方圆→锋矢→雁行→鱼鳞→方圆 循环克制）

### 游戏状态
- `gameState.locations[id] = { owner, grain }` — 据点状态（兵力全在军团里）
- `gameState.factions[id].legions[]` — 军团列表 `{ hero, troops, locationId }`
- `getEffectiveOwner(locId)` — 有军团驻守才算有效占领，否则变中立
- 重要：无 garrison 概念，兵力 100% 绑定军团。无将=无兵。

### 核心循环
- `endTurn()` — 粮草结算→AI顺序执行（带动画）→新回合
- `doPatrol(locId)` — 巡视（事件20%/招募8%/流民18%/献粮7%/打劫7%/情报15%/伏击3%）
- `doAttackWithLegions()` — 进攻→触发战斗界面
- `startBattle()` — 回合制战斗（选将→选阵→多回合单挑）

### 战斗系统（约 1000 行）
- 阶段：`pick`→`formation`→`duel`→`combat_result`→循环→`finished`
- `calculateDuelRound()` — 每回合伤害（基于当前兵力+阵型+克制+战力+地形）
- `executeDuelRound()` — 执行单挑回合
- `retreatFromDuel()` / `retreatInPickPhase()` — 撤退可选相邻己方/无主据点
- 阵型克制方伤害+30%，被克方加成减半

### AI 系统（智能版）
- `executeAITurn(factionId, animations)` — 资源自适应+智能进攻+防守部署
- 动态风格：穷→防守，富→进攻
- 进攻六维评估：战力/地形/克制/中立/价值/风险

### 动画
- `routeAnimations[]` — AI 行军光点动画（阵营色+拖尾）
- `showFloatingDamage()` — 飘伤害数字
- `triggerScreenShake()` — 武将阵亡震动

### 地图渲染
- `renderMap()` — 浅色底（#f5f0e0），清晰道路，大节点
- `setupMapClick()` — 点击选据点+悬停提示 tooltip
- `scaleCoord(x,y)` — 设计坐标→Canvas 映射（/2200, /1400）

### UI
- 顶栏：回合/行动力/据点数/面板开关
- 侧边栏：据点详情+操作按钮+军团列表+事件日志
- 战斗浮层：武将卡片+阵型选择+单挑界面
- 阵营选择弹窗

## 最近改动（2026-06-10）
- **战斗系统全面重做**：
  - 监狱系统：AI投降武将进玩家监狱，释放花50粮草加入玩家阵营
  - 玩家投降：1v1中可选投降保全武将（归降敌方，日后可重新俘回）
  - AI投降：基于忠诚度+战力比的智能判断，忠诚高死战不退
  - AI撤退：选将阶段预判、智能目的地选择、动态门槛（攻方25%/守方40%）
  - AI vs AI自动战斗：完整的撤退+投降机制，与玩家战斗一致
  - 归属确认：玩家投降→归敌方；AI投降玩家→进监狱；AI投降AI→直接加入
- 武将技能移除、招募据点固定化、150人武将池

## 更早改动（2026-06-08）
- 阵营调整：去曾头市 → 加辽国/田虎/王庆，8阵营28据点
- 武将池大扩：54→84人，新增辽国/田虎/王庆部将+梁山/朝廷/方腊补充+中立武将
- 据点专属巡视招募：`LOCATION_HEROES`表，20个据点各1-3名可招武将
- AI行为扩展：新增运输（调兵）、征兵（花粮补兵），AI现在做5种行动
- 防守战斗界面：AI进攻玩家时触发防守战斗
- 各阵营预设武将增强：每阵营4-5名初始武将
- 罗真人技能：天玄（每回合+40粮，巡视无负面事件）
- 所有英雄技能均为被动触发
- 道路行军动画+AI顺序执行

## AI进攻玩家流程
- `executeAITurn` 检测目标为玩家 → 设 `_pendingDefense` → return
- `executeAITurnWithAnim` 返回 `'defense_triggered'`
- `runNextAI` 保存 `_resumeAI` → 调 `startBattle(isPlayerAttacker=false)`
- `endBattle` 检查 `_pendingDefense` → 调 `_resumeAI()` 恢复AI循环
- `aiPickBoth` 检测玩家参战 → 只选AI方武将 → 切换回合给玩家

## 修改注意事项
- 文件是单文件 HTML，CSS+JS 全部内嵌
- JS 在 `<script>` 标签内，约在 290 行之后
- 修改坐标时注意设计网格为 2200×1400
- `calculateCombat()` 是旧版（AI 用），`calculateDuelRound()` 是新版（玩家战斗用）
- 好汉技能需要在对应的战斗/巡逻/休整函数中实现
- 隐藏武将 `hidden: true` 只通过特定地点巡视触发
