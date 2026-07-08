# 猫爬架 V6.1 — 骨架

竖屏攀爬游戏。点左边猫往左跳，点右边往右跳，抓住墙面上的抓取点一直往上爬。

## 技术约束
- Phaser 3, 单文件 index.html
- CDN: `https://cdn.jsdelivr.net/npm/phaser@3.90.0/dist/phaser.min.js`
- 画布 390×690, `Phaser.Scale.FIT + CENTER_BOTH`, `pixelArt: true`
- 不要其他库

## 游戏常量
```
GRAVITY = 900
JUMP_VX = 280      // 水平速度
JUMP_VY = -420     // 垂直速度(向上)
CATCH_RADIUS = 20  // 抓取宽容像素
MAX_FALL_SPEED = 500 // 超过这个速度抓不住
```

## 输入
- `pointerdown` 点左半屏(x<195) = 猫向左上跳
- `pointerdown` 点右半屏(x>=195) = 猫向右上跳
- 键盘: A/← 左跳, D/→ 右跳
- 猫在空中时点一下 = 空气抓(一次坠落只能救一次)

## 猫的物理
猫是一个 Phaser sprite(48×48), 使用 arcade physics:

```
状态机:
  IDLE → JUMP → FALLING → CATCHING → IDLE
                    ↓
                 DEATH (掉出屏幕底部)

JUMP: 
  - 设置 velocity(x=±JUMP_VX, y=JUMP_VY)
  - 播放 jump 动画

FALLING:
  - velocity.y > 50 (下落中)
  - 每帧检测: 猫的矩形区域与所有 grabPoint 组碰撞
  - 如果猫在 CATCH_RADIUS 内, 且 velocity.y < MAX_FALL_SPEED → CATCHING
  - 如果 velocity.y > MAX_FALL_SPEED → 穿过去(掉太快抓不住)

CATCHING:
  - 猫停在抓取点位置
  - velocity = 0
  - body.allowGravity = false
  - 播放 hang 动画
  - 等待玩家再次点击 → 跳转到下一个点
  - 猫挂在上面时尾巴摆动

AIR_CATCH (空气抓):
  - 每轮坠落只能触发一次
  - 在 FALLING 状态再次点击
  - velocity.y = -300 (向上弹)
  - 重新尝试抓取
  - 没抓住 → 继续掉, 不能再用

DEATH:
  - 猫 y > 画布高度 + 100
  - Game Over
```

## 抓取点生成
```
生成 80 个点, 分布从 y=550 到 y=-3500

每局随机生成:
  第一个点: pos(195, 500) 左上方或右上方
  后续点: 
    - 交替侧边(左右左右)
    - 距上一个: 水平 60-140px, 垂直 100-180px
    - 方向朝上 + 对侧

抓取点不用 physics, 用 Phaser.Group 存储 {x, y, side}
在 update 中手动检测猫与点的距离。

每 12 个点换一个 roomId (0-6)
```

## 场景

### GameScene (只有一个场景, 先不做 Title/GameOver)
```
create():
  - 物理 world bounds(0, -4000, 390, 5000)
  - camera 跟随猫(只跟随 y)
  - 生成抓取点数组
  - 创建猫 sprite + physics body(48×48)
  - HUD: 高度显示(顶部居中)

update():
  - 检测输入 → 跳跃
  - 检测抓取
  - 检测坠落 → 重开
  - 更新高度显示
  - 更新抓取点摆动(moving 类型)
```

## 猫的 spritesheet
```
generateCatSpritesheet(): 
  32×32 × 6 帧, canvas 绘制:
  帧 0: idle(站姿)
  帧 1: hang(挂在墙上, 尾巴向下弯)
  帧 2: jump_left(向左跳, 身体左倾)
  帧 3: jump_right(向右跳, 身体右倾)
  帧 4: fall(下坠, 四肢张开)
  帧 5: air_catch(向上弹, 伸爪)

  颜色:
    fur: "#FF9F43"
    stripe: "#D97730"
    belly: "#FFF3E0"
    eye: "#2D5A27"
    pink: "#FF9EBB"
```

## 视觉(最小化)
只画 2 个房间背景(用纯色/简单图形):
- room 0(地下室): 深灰 #3D3D3D 墙, 水平线表示砖缝
- room 1(厨房): 暖白 #FFF8E7 墙, 淡蓝窗

抓取点画法:
- 地下室: 铁管接头(棕色小矩形)
- 厨房: 抽屉把手(金色小圆)

## 实现范围(本次只做这些)
- [x] 竖屏 Phaser 390×690
- [x] 猫 32×32 spritesheet(6帧)
- [x] 点左跳左点右跳右
- [x] 抓取检测 + 抓住后停在点上
- [x] 80 个随机抓取点生成(左右交替)
- [x] 猫掉出屏幕 → 重开
- [x] 高度显示
- [x] 2 个房间简单背景
- [x] 键盘 A/D 备用

## 本次不做
- 标题/结束场景(下次)
- 空气抓(下次)
- 移动抓取点(下次)
- 最高分存储(下次)
- 音效(下次)
- 7 个完整房间(下次)
