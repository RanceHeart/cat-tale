# CODEX V9 — 点击方向 + 蓄力跳

## 修复方向判断

- 恢复跳跃方向为 **屏幕分区**：点屏幕左半 → 跳左，右半 → 跳右
- 移除旧的"跳向下一个抓取点"逻辑，`handleCatRelativeJump()` 改为纯屏幕分区

## 蓄力时方向指示箭头

- 触屏时（pointerdown），根据手指点击方向显示箭头：
  - 点击左半屏 → 猫旁边显示 `◀`（蓄力过程持续闪烁）
  - 点击右半屏 → 猫旁边显示 `▶`
- **箭头方向来自手指位置，不是来自抓取点位置**（这与旧 V8 箭头逻辑不同）
- 蓄力期间箭头持续脉冲闪烁（alpha 0.4~1.0）
- pointerup 释放跳跃的同时箭头隐藏
- 跳跃/下落期间箭头隐藏（和现在一样）

## 新增蓄力跳系统

仅当 `catState === IDLE` 或 `CATCHING`（猫挂在墙上时）触发。

### 触摸流程

```
pointerdown → 开始蓄力，进入 CHARGING 状态
    ↓
蓄力期间：猫原地压缩拉伸动画（squash animation），同时显示蓄力进度条
    ↓
pointerup → 根据蓄力时间释放跳跃
```

### 蓄力参数

- 最大蓄力时间：`MAX_CHARGE = 1200ms`
- 最小值（轻触，蓄力 < 80ms）：速度 = 当前标准值 VX=±280, VY=-420
- 最大值（蓄满 1200ms）：VX=±500, VY=-700
- 中间值：线性插值
  ```
  t = clamp(chargeTime / MAX_CHARGE, 0, 1)
  vx = lerp(280, 500, t) * direction
  vy = lerp(-420, -700, t)
  ```
- `CatState.CHARGING` 新状态（防止跳跃中误触发）

### 视觉反馈

1. **猫的压缩动画**：蓄力时垂直缩放从 1→0.7（squash），水平从 1→1.15（stretch）
2. **蓄力进度条**：猫头顶显示一个窄条（宽 40px，高 4px），背景深色，填充金色，宽度随蓄力增长
3. **进度条颜色**：0-50% 金色 `#FFD700`，50-80% 橙色 `#FF8C00`，80-100% 红色 `#FF4444`
4. 蓄满 1200ms 时自动释放跳跃（不用等用户松手）

### 键盘适配

长按 A/D 键蓄力，松手释放。键的蓄力逻辑跟触屏一致。

### 状态机变更

```
新增: CatState.CHARGING = 4

IDLE/CATCHING → (pointerdown) → CHARGING
CHARGING → (pointerup / 蓄满) → JUMP
CHARGING → (pointer离开屏幕) → JUMP
```

### 其他

- 蓄力期间空气抓（air catch）不可用（本来就是挂墙状态，不需要）
- 蓄力时重力关闭，蓄力结束时恢复
- 进度条使用 Phaser Graphics 绘制（别用 Text）
- 蓄力跳跃的高度需要能跳过最远的垂直间隔（180px + 一些余量），满蓄力 VY=-700 应该够

## 文件

只修改 `/tmp/cat-tale/index.html`。
