# CODEX V10 — 手指方向 + 猫爪蓄力

## 跳跃方向改为手指相对猫

- `handleCatRelativeJump(pointer)` → `direction = pointer.x < this.cat.x ? -1 : 1`
- 不再是屏幕半区，而是**手指在猫的左边还是右边**
- 移除旧逻辑：不再根据 pointer.x < GAME_WIDTH/2

## 蓄力中可实时变向

- 添加 `this.input.on("pointermove", ...)` 监听
- 仅当 `catState === CHARGING` 时，根据当前手指位置更新方向
- `updateJumpArrow()` 读取 `this.chargeDirection` 实时更新箭头方向

## 去掉蓄力进度条

- 删除 `this.chargeBar`、`drawChargeBar()` 方法及其所有调用
- 猫头顶不再显示进度条

## 猫爪颜色显示蓄力

蓄力时在猫的四肢/爪子位置绘制发光小方块，颜色随蓄力变化：

### 爪子位置（相对于猫 sprite 中心，displaySize 48×48）
```
左前爪: cat.x - 16, cat.y + 14  (左前腿底部)
右前爪: cat.x + 16, cat.y + 14  (右前腿底部)
左后爪: cat.x - 14, cat.y + 19  (左后腿底部)
右后爪: cat.x + 14, cat.y + 19  (右后腿底部)
左臂:   cat.x - 16, cat.y - 17  (左臂/挂墙的手)
右臂:   cat.x + 16, cat.y - 17  (右臂/挂墙的手)
```

### 颜色变化（根据 charge t）
- t=0：不显示（爪子正常）
- t=0~0.3：`#FF9F43`（猫毛色，微弱光晕）
- t=0.3~0.6：`#FF6B35`（橙色，中等亮度）
- t=0.6~0.8：`#FF3D00`（亮橙红）
- t=0.8~1.0：`#FF0000`（红色）

使用 Phaser Graphics 绘制，每个爪位置画一个 4×4 的小方块。
Depth 设为 11（在猫之上）。
清除旧图形并重绘：graphics.clear()。

### 外观建议
- 每个爪子画一个小光晕方块（4×4px，带半透明 outer glow 更好）
- 或画 2×2 实心方块（干净利落）
- 看起来像猫的爪子在蓄力时"充能发光"

## 文件

只修改 `/tmp/cat-tale/index.html`。
