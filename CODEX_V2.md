# 猫猫物语 V2 — 完全重写

## V1 的问题（必须避免）

1. **KAPLAY + Rough.js 组合不兼容** — Rough.js 渲染到 canvas → 转 dataURL → 加载为 KAPLAY sprite，管线冲突导致碰撞检测不准 + 动画卡顿
2. **引擎不够成熟** — KAPLAY 物理系统太轻量，对精确平台跳跃支持差
3. **手感差** — 加速度曲线不对，跳跃没重量感，碰撞检测有 bug

## V2 方案

**Phaser 3 (v3.90.0)** + **Arcade Physics** + **程序化像素画精灵**

Phaser 3 是目前最成熟的 HTML5 2D 游戏引擎，Arcade Physics 专门为马里奥类平台游戏设计。摒弃所有花哨库组合，一个引擎干所有事。

## CDN
```
https://cdn.jsdelivr.net/npm/phaser@3.90.0/dist/phaser.min.js
```

## 核心机制

- 2D 横版平台跳跃
- 玩家控制猫猫 小咪
- 方向键 / WASD 移动，Space / Up / W 跳跃
- 踩敌人消灭，收集鱼加分，到达旗杆过关
- 3 条命，掉落即死（无血条）
- 3 个关卡，递增难度

## 手感规格（这是关键，V1 就是这些没做好）

**必须严格使用以下数值：**

```
GRAVITY = 1200
PLAYER_SPEED = 200
PLAYER_ACCELERATION = 1800    // 从 0 加速到 max 约 0.11s
PLAYER_DRAG = 1200            // 松键后减速约 0.17s
JUMP_VELOCITY = -460          // 基础跳跃高度约 88px
JUMP_HOLD_FORCE = -80         // 按住 Space 时每秒额外向上的力（变高跳跃）
JUMP_HOLD_MAX_TIME = 0.32     // 最多蓄力时间
JUMP_BUFFER_TIME = 0.12       // 落地前 120ms 按跳都生效（buffer）
COYOTE_TIME = 0.08            // 离开平台后 80ms 内还能跳
```

**为什么这些值重要：**
- GRAVITY 1200 + JUMP_VELOCITY -460 → 跳跃曲线跟 SMB 接近，不飘不沉
- ACCELERATION 1800 → 按就立刻响应，不肉
- DRAG 1200 → 松键很快停下，不滑冰
- JUMP_HOLD → 按得久跳得高，轻按 = 小跳（过矮平台），长按 = 大跳
- BUFFER + COYOTE → 操作容错，手感从"苛刻"变"舒适"

## 角色动画

不要用 Rough.js，用 **程序化像素画** 生成 spritesheet：

方法：定义 2D 颜色数组作为像素数据，用 canvas 绘制每一帧，合并为 Phaser spritesheet。

猫猫 32×32 像素，6 帧动画：
- 帧 0-1: idle（呼吸微动，尾巴摆动）
- 帧 2-3: run（脚步交替，尾巴跟随）
- 帧 4: jump（四肢伸展，尾巴向上）
- 帧 5: fall（四肢张开，尾巴向下）

核心设计：
- 橙色虎斑猫（主色 #FF8C42）
- 圆脸、三角耳、大眼、粉色鼻子、胡须
- 长尾巴（始终在运动，比身体晚 1 帧 → overlapping action）
- 白色肚皮（#FFF3E0）
- 深色条纹（#E0672A）

用辅助函数 `generateCatSpritesheet()` 生成所有帧到一张 canvas，然后用 `this.textures.addSpriteSheet()` 加载。

## 视觉风格

像素风 + 明亮配色，不要灰暗：

```
天空: #87CEEB → #B7E6FF 渐变
草地: #7EC850
土地: #8B5E3C
砖块: #C8A87C (暖米色)
鱼(金币): #FFD700
敌人: #E74C3C (红)
```

敌人也是程序化像素画：
- 乌鸦（Level 1）：紫色 #8B5CF6，32×24，简单前后巡逻
- 狗（Level 2）：棕色 #8B4513，40×28，比乌鸦大，跑得更快
- 鳄鱼 Boss（Level 3）：绿色 #2E8B57，56×32，在平台上水平移动，需要多次踩

## 关卡设计（用 Phaser Tilemap / 2D 数组）

```
const LEVELS = [{
  name: "花园",
  theme: '#87CEEB',
  map: [
    '                                                         ',
    '                                                         ',
    '      %                        %                @        ',
    '                                                         ',
    '                      ^                                  ',
    '  B=========B  B=============B  B=========B  B=====B    ',
    '                                                         ',
    '=========================================================',
    '                                                         ',
    '=========================================================',
  ]
}, ...];
```

三个关卡，跟 V1 一样：
- **Level 1 "花园"**：平地 + 简单平台 + 1 个乌鸦，教学关
- **Level 2 "屋顶"**：更多跳跃 + 移动平台 + 乌鸦 + 狗
- **Level 3 "皇宫"**：窄平台 + 尖刺 + Boss

## Phaser 结构

```javascript
const config = {
  type: Phaser.AUTO,
  width: 800,
  height: 480,
  physics: {
    default: 'arcade',
    arcade: { gravity: { y: 1200 }, debug: false }
  },
  scene: [BootScene, TitleScene, GameScene, GameOverScene, WinScene]
};
```

### Scenes

1. **BootScene** — 生成所有像素画精灵、音效、加载资源
2. **TitleScene** — "猫猫物语" 标题，按 Space 开始，BGM 播放
3. **GameScene** — 主要游戏循环：关卡渲染、玩家控制、敌人 AI、碰撞、HUD、相机跟随
4. **GameOverScene** — 分数显示，按 Space 重试
5. **WinScene** — 全通关祝贺，彩纸效果

### GameScene 核心逻辑

```
create():
  - 根据当前关卡索引加载 map 数据
  - 创建静态物理组（地面、平台）
  - 创建玩家精灵（带物理体）
  - 创建敌人组（带物理体 + 巡逻 AI）
  - 创建收集物组（鱼）
  - 创建旗杆（过关目标）
  - 设置碰撞处理器
  - 创建 HUD（分数左上，生命右上）
  - 设置相机跟随

update(time, delta):
  - 检测方向键输入 → 应用加速度/拖拽
  - 检测跳跃输入 → coyote/buffer 逻辑
  - 更新敌人 AI（巡逻、转向）
  - 更新玩家动画状态（idle/run/jump/fall）
  - 检查掉落深渊 → 减命或 Game Over
```

## 音效（Web Audio API）

不要音频文件，程序化生成 sfx：
- 跳跃: sine 400→700Hz, 0.1s
- 收集: sine 800→1200Hz, 0.08s  
- 踩敌: square 200Hz, 0.05s
- 受伤: sawtooth 150→80Hz, 0.2s
- 着陆: sine 200Hz, 0.03s
- 过关: sine 500→800→1200Hz, 0.3s 上行琶音

音量约为 BGM 的 25%。

## BGM

"Hep Cats" Kevin MacLeod (CC BY 4.0)
下载: https://incompetech.com/music/royalty-free/mp3-royaltyfree/Hep%20Cats.mp3
用 Phaser 的 `this.sound.add()` 加载并循环播放。

如果 Hep Cats 加载失败，fallback 到：
"Electro Cabello" https://incompetech.com/music/royalty-free/mp3-royaltyfree/Electro%20Cabello.mp3

## HUD

- 左上角: ❤❤❤ (生命图标) + 猫猫小头像
- 右上角: 🐟 × 分数 (鱼图标 + 数字)
- 关卡名: 场景切换时短暂显示（2秒后消失）

## 移动端

简单的触摸按钮覆盖在 canvas 下方：
- 左 | 右 | 跳（大按钮，方便拇指）
- 用 Phaser 的 `this.input.on('pointerdown')` 检测

## 实现顺序

### Phase 1: 骨架（让它能跑）
1. Phaser 3 init + config, 5 个 scene 骨架
2. BootScene: 程序化生成猫猫 spritesheet（6 帧 32×32 像素数组）
3. BootScene: 生成敌人 spritesheet, 地面/砖块/鱼/旗杆纹理
4. TitleScene: 静态标题 + "按 SPACE 开始"
5. GameScene: 渲染 Level 1 地图, 玩家移动 + 跳跃（用上述物理数值！）
6. 物理碰撞: 玩家 ↔ 地面/平台

### Phase 2: 玩法（让它好玩）
7. 敌人巡逻 AI + 踩灭逻辑 + 玩家弹跳奖励
8. 鱼收集 + 分数
9. 3 条命 + 掉落死亡 + 重生
10. 旗杆过关 + Level 2/3
11. 相机跟随

### Phase 3: 手感调校（最重要！）
12. 验证 Coyote Time + Jump Buffer 正常工作
13. 验证 Variable Jump 轻按/长按有区别
14. 验证加速/减速曲线不滑
15. 验证碰撞不卡边、不穿墙

### Phase 4: 视觉 + 音效
16. GameOver / Win 场景
17. HUD 分数 + 生命
18. Web Audio SFX（所有 6 种）
19. BGM 加载循环
20. 触摸按钮

### Phase 5: 完整关卡
21. Level 2: 屋顶场景 + 移动平台 + 狗敌人
22. Level 3: 皇宫场景 + 尖刺 + 鳄鱼 Boss
23. 全部通关验证

## Git 工作流

```bash
# Codex 完成后手动执行：
cd /tmp/cat-tale
git add -A
git commit -m "feat: V2 — rewrite with Phaser 3"
git push
# GitHub Pages 自动部署
```

## 绝对不要做的事

- ❌ 不要用任何非 Phaser 的库（No KAPLAY, No Rough.js, No anime.js）
- ❌ 不要程序化生成音乐（用 Kevin MacLeod 现成曲目）
- ❌ 不要添加故事/对话/菜单选项
- ❌ 不要搞 RPG 元素、升级、背包
- ❌ 不要写超过 2 个 HTML 文件（单文件 index.html）
- ❌ 不要用 ES Modules 或 import（用 script src 加载 Phaser CDN）
