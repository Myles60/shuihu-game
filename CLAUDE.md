# 水浒·天下争锋 — 开发文档

纯前端单文件回合制策略游戏。水浒题材，6 阵营争夺 28 据点。

## 快速启动
- 文件：`index.html`（约 3600 行，零依赖）
- 打开：双击或用浏览器直接打开
- 在线版：`https://myles60.github.io/shuihu-game/`

## 核心系统（按代码位置）

### 数据层
- `FACTIONS[]` — 6 阵营（朝廷/方腊/梁山泊/大名府/祝家庄/曾头市）
- `LOCATIONS[]` — 28 据点（坐标 2200×1400 设计网格）
- `CONNECTIONS[]` — 据点间连线（road/mountain/water）
- `HERO_POOL[]` — 54 位好汉（含 4 位隐藏彩蛋：贾诗涵/张浩宇/张永旭/麻昇）
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

## 最近改动（2026-06-08）
- 地图重绘为浅色清晰风格，据点间距大幅拉开
- 战斗系统重构（阵型+多回合+撤退目的地）
- AI 大幅强化（智能进攻+防守反击+资源自适应）
- 巡视概率全面调优，新增加富绅献粮/流寇打劫事件
- 好汉池扩至 54 位
- 好友彩蛋（贾诗涵@浔阳江、张浩宇@梁山泊、张永旭@景阳冈、麻昇@东京汴梁）
- 道路行军动画+AI顺序执行

## 修改注意事项
- 文件是单文件 HTML，CSS+JS 全部内嵌
- JS 在 `<script>` 标签内，约在 290 行之后
- 修改坐标时注意设计网格为 2200×1400
- `calculateCombat()` 是旧版（AI 用），`calculateDuelRound()` 是新版（玩家战斗用）
- 好汉技能需要在对应的战斗/巡逻/休整函数中实现
- 隐藏武将 `hidden: true` 只通过特定地点巡视触发
