# V8 — 两个 bug 修复

## Bug 1: 顶部黑一块
body 背景色 `#171717` 太深，画布缩放后上下露出黑边。

修复：把 body background 改为 `#3D3D3D`（跟房间 0 的墙壁色一致），这样上下黑边看起来就是墙壁的一部分。

或者更好的方案：去掉 body 背景色，改为 `background: #3D3D3D`

## Bug 2: 经常点反
当前：点任何地方 → 由猫的位置自动算方向 → 玩家没掌控感。

修复：**显示箭头提示，玩家决定何时跳**
- 猫挂墙上时，在猫旁边显示一个箭头指向下一个抓取点的方向
- 玩家点屏幕 → 猫沿着箭头方向跳
- 箭头持续闪烁（透明度 0.4↔1.0 循环）
- 这样玩家每次跳之前都看到方向，不会"点反"

具体修改：
```javascript
// 在 create() 里，猫抓住点后：
this.jumpArrow = this.add.text(0, 0, "▶", {
  fontSize: "18px", color: "#FFD700", stroke: "#000", strokeThickness: 3
}).setOrigin(0.5).setDepth(15);

// 在 update() 里更新箭头位置和方向：
if (this.catState === CatState.IDLE || this.catState === CatState.CATCHING) {
  const nextPoint = this.getNextGrabPoint();
  if (nextPoint) {
    const direction = nextPoint.x < this.cat.x ? "◀" : "▶";
    this.jumpArrow.setText(direction);
    this.jumpArrow.setPosition(this.cat.x + (nextPoint.x < this.cat.x ? -30 : 30), this.cat.y - 10);
    this.jumpArrow.setAlpha(0.5 + Math.sin(this.time.now / 160) * 0.5);
    this.jumpArrow.setVisible(true);
  }
} else {
  this.jumpArrow.setVisible(false);
}

// 处理跳跃（保持 auto-direction）：
this.input.on("pointerdown", () => {
  this.handleCatRelativeJump();
});
```

## 不改
- 物理参数
- 猫 spritesheet
- 房间系统
- 抓取点生成
