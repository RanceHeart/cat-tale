# V7 — 修两个 bug

## Bug 1: 顶部黑一块
`backgroundColor: "#3D3D3D"` 太暗，相机顶部露出深色块。

修复：
- `backgroundColor` 改成与 room 0 背景色一致的颜色 `#3D3D3D` → 改成 `#3D3D3D`
- 实际上是 createRooms() 没覆盖到 WORLD_TOP 最顶部，扩展 roomHeight 计算确保全覆盖
- 或者在 init 里加一个覆盖全世界的背景矩形，颜色 = room 0 的墙色

最简单修复：createRooms() 里 for 循环条件从 `i <= topRoom` 改成 `i <= topRoom + 1` 或者直接 set world bounds 背景为 room 0 色。

更简单的：把 Phaser backgroundColor 改为底部房间的颜色即可。

## Bug 2: 左右应该是猫的左右
当前 `handleJump(pointer.x < GAME_WIDTH / 2 ? -1 : 1)` 按屏幕左右分区。
但猫在墙上时，它的"左"不是屏幕的左半。

修复：去掉屏幕分区，不管点哪里，方向由猫当前所在位置决定：
```javascript
// 猫在左墙 → 向右跳 (+1)
// 猫在右墙 → 向左跳 (-1)
this.input.on("pointerdown", () => {
  if (this.catState === CatState.IDLE || this.catState === CatState.CATCHING) {
    const direction = this.cat.x < GAME_WIDTH / 2 ? 1 : -1;
    this.handleJump(direction);
  }
});
```

键盘控制同理也简化：`A` 和 `D` / `←` 和 `→` 也都调用这个逻辑。

## 不改
- 物理参数
- 抓取点生成
- 猫的 spritesheet
- 房间数量/样式
