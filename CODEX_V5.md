# V5 — 适配手机

不改玩法和美术，只改移动端适配。

## 问题

当前游戏虽然有 `Phaser.Scale.FIT` 和触摸按钮，但手机适配不彻底：

1. 触摸按钮固定尺寸，小屏手机太小
2. HUD 文字固定 px，手机上看不清
3. 标题/结束场景提示 "按 SPACE"，手机看不到
4. 触摸区域没针对竖屏优化

## 改动清单

### 1. 触摸按钮适配屏幕
改 `.touch-btn` 尺寸为视口相对单位：
```css
.touch-btn {
  width: 18vmin;
  height: 14vmin;
  max-width: 90px;
  max-height: 70px;
  font-size: 5vmin;
  border-radius: 2vmin;
}
.touch-btn.jump {
  width: 24vmin;
  height: 17vmin;
  max-width: 120px;
  max-height: 85px;
}
```
这样小屏自动缩小，大屏不会太大。

按钮 padding 用 `env(safe-area-inset-bottom)` 已经在了，确认生效。

### 2. 触屏检测 + 提示文字替换
所有场景中的 `"按 SPACE 开始"` → 检测设备，触屏设备显示 `"点击屏幕开始"`。

在 JS 开头加：
```javascript
const IS_TOUCH = ('ontouchstart' in window) || (navigator.maxTouchPoints > 0);
```

然后在 TitleScene、GameOverScene、WinScene 中用：
```javascript
const prompt = IS_TOUCH ? "点击屏幕开始" : "按 SPACE 开始";
```

### 3. 触屏事件优化
当前 `setupTouchControls()` 监听 `pointerdown/up`。需要也监听 `touchstart/touchend` 兜底：
```javascript
button.addEventListener("touchstart", (e) => { e.preventDefault(); setValue(true); }, { passive: false });
button.addEventListener("touchend", (e) => { e.preventDefault(); setValue(false); }, { passive: false });
```

### 4. 竖屏适配说明（不需要改代码）
当前 `Phaser.Scale.FIT` + `CENTER_BOTH` 已经自动处理横竖屏，保持 800×480 逻辑分辨率即可。

### 5. 页面防止缩放
```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
```

### 6. TitleScene 触摸启动
当前 TitleScene 只监听了 `keydown-SPACE`。加触摸/点击检测：
```javascript
this.input.once("pointerdown", () => {
  state.score = 0;
  state.lives = 3;
  this.scene.start("game");
});
```

Canvas 全屏可点，不需要精确点击按钮。

### 7. HUD 字号用场景比例
当前 HUD 字号写死 `16px` / `18px` 等。改成 `Math.round(GAME_WIDTH * 0.022)` 等相对值，确保在不同缩放下清晰。

---

## 不改的
- ❌ 不改任何游戏逻辑
- ❌ 不改物理参数
- ❌ 不改关卡数据
- ❌ 不改美术/纹理
- ❌ 不加新库

只改 CSS + 触屏检测 + 提示文字。
