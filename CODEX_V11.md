# CODEX V11 — 猫爪像素画变色（非叠方块）

## 问题
V10 的 chargePaws 在猫 sprite 上叠 6 个小方块，丑陋。改为在像素画层面生成新的猫帧，爪子和四肢真正变色。

## 改动

### A. `drawCatFrame(ctx, ox, frame)` → 加 `pawColor` 参数

```js
function drawCatFrame(ctx, ox, frame, pawColor) {
```

当 `pawColor` 存在时，以下像素使用 pawColor 替换原本颜色：

**左臂**（hang 帧特有）：
`px(ctx, ox + 10, 3, 3, 10, pawColor || fur)`

**右臂**：
`px(ctx, ox + 20, 3, 3, 10, pawColor || fur)`

**左爪垫**（hang 帧特有）：
`px(ctx, ox + 9, 1, 5, 3, pawColor || pink)`

**右爪垫**：
`px(ctx, ox + 19, 1, 5, 3, pawColor || pink)`

**左后腿**（body 部分，所有帧共用，但 charge 帧只用 hang 姿态所以 OK）：
`px(ctx, ox + 10 + lean, bodyY + 10, 4, 5, pawColor || fur)`

**右后腿**：
`px(ctx, ox + 19 + lean, bodyY + 10, 4, 5, pawColor || fur)`

其他所有像素保持不变。注意：`pink` 变量（耳内粉）在非 pawColor 时仍要用原色，所以写成 `pawColor || pink` 会给耳朵也变色——应该只改爪垫位置的 px 调用，保留 `pink` 变量本身不变。

### B. 扩展 Spritesheet 到 9 帧

```js
const { canvas, ctx } = makeCanvas(32 * 9, 32);
for (let i = 0; i < 6; i++) {
  drawCatFrame(ctx, i * 32, i);
}
// 3 个蓄力帧 = hang姿态 + 不同爪色
drawCatFrame(ctx, 6 * 32, 1, "#FF8C00");  // 暗橙
drawCatFrame(ctx, 7 * 32, 1, "#FF5500");  // 橙红
drawCatFrame(ctx, 8 * 32, 1, "#FF0000");  // 亮红
```

`scene.textures.addSpriteSheet("cat", canvas, { frameWidth: 32, frameHeight: 32 })` 不变，自动按 32px 切 9 帧。

### C. 添加 3 个单帧动画

```js
this.anims.create({ key: "cat-charge0", frames: [{ key: "cat", frame: 6 }], frameRate: 1 });
this.anims.create({ key: "cat-charge1", frames: [{ key: "cat", frame: 7 }], frameRate: 1 });
this.anims.create({ key: "cat-charge2", frames: [{ key: "cat", frame: 8 }], frameRate: 1 });
```

### D. updateCharge() 替换 chargePaws

删除 `drawChargePaws()` 和 `hideChargePaws()` 的调用，改为根据 t 播放对应帧：

```js
if (t < 0.1) {
  this.cat.play("cat-hang");
} else if (t < 0.4) {
  this.cat.play("cat-charge0");
} else if (t < 0.7) {
  this.cat.play("cat-charge1");
} else {
  this.cat.play("cat-charge2");
}
```

### E. 删除所有 chargePaws 相关

- 删除 `this.chargePaws` 创建
- 删除 `drawChargePaws()` 方法
- 删除 `hideChargePaws()` 方法
- 删除所有对 `this.hideChargePaws()` 和 `this.chargePaws` 的引用
- 在 `handleJump` / `catchPoint` 中不需要额外操作，因为猫帧已被 `cat.play()` 覆盖

### F. 恢复正常帧

蓄力释放后 `handleJump` 会 `cat.play("cat-jump-xxx")`，自动回到正常帧。
`catchPoint` 会 `cat.play("cat-hang")`，也自动回到正常帧。
所以不需要额外重置。

## 文件
只修改 index.html。
