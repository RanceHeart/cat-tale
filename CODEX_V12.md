# CODEX V12 — 6区高度视觉进化系统

**核心改动：从 2 个房间交替 → 6 个高度分区，每区有独特视觉主题、抓取点外观、背景色和装饰元素。**

## 1. 新增：ZONE 常量和辅助函数

在 `ROOM_DARK_CSS` 常量之后添加：

```js
// 高度分区定义 (基于 world y 坐标，height = (500 - y) / 10)
function getZoneByY(y) {
  const h = (500 - y) / 10; // 米
  if (h < 60)  return 0; // 地下室
  if (h < 200) return 1; // 住宅区
  if (h < 400) return 2; // 阁楼/天台
  if (h < 600) return 3; // 云层区
  if (h < 800) return 4; // 高空区
  return 5;               // 平流层
}

const ZONE_CONFIG = [
  { name: "basement", bg1: 0x3d3d3d, bg2: 0x454545, lineColor: 0x555555, brickColor: 0x505050, grabColor: 0x8b5a2b, grabStroke: 0x4f331d, depthBg: -20, depthDecor: -19 },
  { name: "residential", bg1: 0xfff8e7, bg2: 0xe8f4fd, lineColor: 0xd8c39a, brickColor: 0xb9863c, grabColor: 0xd4af37, grabStroke: 0x8b6f18, depthBg: -20, depthDecor: -19 },
  { name: "attic", bg1: 0x5c4033, bg2: 0x4a3528, lineColor: 0x8b7355, brickColor: 0x7a6548, grabColor: 0xcd853f, grabStroke: 0x8b6914, depthBg: -20, depthDecor: -18 },
  { name: "cloud", bg1: 0xb0d4f1, bg2: 0xd4e8f7, lineColor: 0xffffff, brickColor: 0x87ceeb, grabColor: 0x40e0d0, grabStroke: 0x20b2aa, depthBg: -20, depthDecor: -18 },
  { name: "highalt", bg1: 0x1a237e, bg2: 0x0d47a1, lineColor: 0xffffff, brickColor: 0x42a5f5, grabColor: 0xffd700, grabStroke: 0xffab00, depthBg: -20, depthDecor: -18 },
  { name: "stratosphere", bg1: 0x0d0221, bg2: 0x1a0533, lineColor: 0x7c4dff, brickColor: 0x651fff, grabColor: 0xe040fb, grabStroke: 0xaa00ff, depthBg: -20, depthDecor: -18 }
];
```

## 2. 重写 createRooms()

替换整个 `createRooms()` 方法。保持 `roomHeight = 720` 和原有循环结构，但根据 `getZoneByY(y)` 选择不同的绘制逻辑。

```js
createRooms() {
  const roomHeight = 720;
  const topRoom = Math.floor(Math.abs(WORLD_TOP) / roomHeight) + 1;

  // 最底层背景（depth -30，使用起始 zone 颜色）
  this.add.rectangle(GAME_WIDTH / 2, WORLD_TOP + WORLD_HEIGHT / 2, GAME_WIDTH, WORLD_HEIGHT, 0x3d3d3d).setDepth(-30);

  for (let i = 0; i <= topRoom + 1; i++) {
    const y = 690 - (i + 1) * roomHeight;
    const zone = getZoneByY(y);
    const cfg = ZONE_CONFIG[zone];
    // 在每个 720px 的房间内交替两个变体
    const variant = i % 2;
    const bgColor = variant === 0 ? cfg.bg1 : cfg.bg2;

    // 房间底色
    this.add.rectangle(195, y + roomHeight / 2, GAME_WIDTH, roomHeight, bgColor).setDepth(cfg.depthBg);

    // 各区特色装饰
    this.drawZoneDecor(zone, y, roomHeight, variant);
  }
}
```

## 3. drawZoneDecor(zone, y, roomHeight, variant)

新增方法，为每个 zone 绘制专属装饰。

### Zone 0 — 地下室 (Basement)
老样子保持：
- 水平砖缝：`this.add.rectangle(195, lineY, GAME_WIDTH, 2, 0x555555)` 每 46px
- 垂直砖块：`this.add.rectangle(x, lineY, 2, 24, 0x505050)` 每 92px x 96px

### Zone 1 — 住宅区 (Residential)  
变体 0（暖黄墙）：
- 窗户：左侧一个窗户（x=50, y+170, 38x50 蓝色，边框金色）
- 书架：右侧（x=280, y+300, 80x120 棕色带书分隔线）
- 钟表：右侧上方圆形（x=300, y+80, 15 浅金）

变体 1（浅蓝墙）：
- 画框：右侧（x=280, y+150, 60x50 深棕框，浅色内）
- 台灯：左侧（x=50, y+400, 30x60 浅黄灯罩+细杆）
- 暖气片：底部（x=100, y+650, 180x30 灰白条纹）

### Zone 2 — 阁楼/天台 (Attic)
变体 0（深木色）：
- 木梁斜顶：从两边向中间倾斜的三角形轮廓（x=0→195, y→y+80 和 x=390→195, y→y+80 画细线）
- 旧箱子：底部（x=50, y+550, 80x50 棕色）
- 吊灯：中间（x=195, y+50, 向下的线条+小灯）

变体 1（更深的木色）：
- 老虎窗：中间上方（x=140, y+80, 80x80 斜顶小窗）
- 木地板线：水平线每 50px （0x7a6548）
- 工具箱：角落（x=280, y+600, 60x50 暗红）

### Zone 3 — 云层区 (Cloud)
变体 0（天蓝色）：
- 浮云：3 朵白色/浅灰圆角云朵，分别在不同位置
- 每朵云：2-3 个叠加的圆角矩形（椭圆 shape，如 40x20, 30x20, 20x20 叠加）
- 云朵颜色 `0xffffff` alpha 0.3-0.5

变体 1（淡蓝）：
- 飞鸟剪影：V 形小三角形（2-3个），棕黑色 `0x5D4037`
- 小云：更稀疏，更高处

### Zone 4 — 高空区 (High Altitude)
变体 0（深蓝）：
- 星星：10-15 个小圆点 `0xffffff` alpha 0.5-1.0，随机散布
- 弦月：右上角白色弧形（两个重叠圆做月牙）

变体 1（深蓝偏紫）：
- 更多星星 + 流星尾迹（短的白色斜线）
- 薄雾带：水平半透明条 alpha 0.1

### Zone 5 — 平流层 (Stratosphere)
变体 0（紫黑）：
- 极光：3 条垂直弯曲的彩色光带，半透明渐变
  - 使用 add.rectangle + alpha 模拟，颜色 `0x7c4dff` `0x00bcd4` `0xe040fb`，alpha 0.15-0.3
  - 从左侧 30px 到右侧不同位置，高度 120-200px
- 明亮星星：带光晕的 `0xffffff` 星点

变体 1（深紫）：
- 极光另一方向弯曲
- 地球弧线：底部显示棕色/蓝色弧线暗示地平线
- 超亮星星

## 4. 重写 drawGrabPoints()

根据 `getZoneByY(point.y)` 选择不同抓取点外观。

```js
drawGrabPoints() {
  this.pointGroup = this.add.group();
  this.grabPoints.forEach((point) => {
    const zone = getZoneByY(point.y);
    // 不同 zone 不同样式
    let elements = [];
    switch (zone) {
      case 0: // 地下室 — 锈水管+螺栓
        elements = this.makeGrabPipe(point);
        break;
      case 1: // 住宅 — 金色窗帘杆钩
        elements = this.makeGrabCurtain(point);
        break;
      case 2: // 阁楼 — 铜钩+木梁
        elements = this.makeGrabHook(point);
        break;
      case 3: // 云层 — 水晶钩（青蓝）
        elements = this.makeGrabCrystal(point, 0x40e0d0, 0x20b2aa);
        break;
      case 4: // 高空 — 星形光钩（金-青）
        elements = this.makeGrabCrystal(point, 0xffd700, 0xffab00);
        break;
      case 5: // 平流层 — 极光水晶（紫粉）
        elements = this.makeGrabCrystal(point, 0xe040fb, 0xaa00ff);
        break;
    }
    elements.forEach(el => this.pointGroup.add(el));
  });
}

// 辅助方法

makeGrabPipe(point) {
  const pipe = this.add.rectangle(point.x, point.y, 28, 9, 0x8b5a2b).setStrokeStyle(2, 0x4f331d);
  const boltA = this.add.circle(point.x - 8, point.y, 2, 0xd2b48c);
  const boltB = this.add.circle(point.x + 8, point.y, 2, 0xd2b48c);
  return [pipe, boltA, boltB];
}

makeGrabCurtain(point) {
  const rod = this.add.rectangle(point.x, point.y, 24, 5, 0xd4af37).setStrokeStyle(1, 0x8b6f18);
  const ringL = this.add.circle(point.x - 8, point.y, 4, 0xffe58f);
  const ringR = this.add.circle(point.x + 8, point.y, 4, 0xffe58f);
  return [rod, ringL, ringR];
}

makeGrabHook(point) {
  const hook = this.add.rectangle(point.x, point.y, 18, 8, 0xcd853f).setStrokeStyle(2, 0x8b6914);
  const glow = this.add.circle(point.x, point.y - 4, 3, 0xffd4a3, 0.6);
  return [hook, glow];
}

makeGrabCrystal(point, color, strokeColor) {
  const crystal = this.add.rectangle(point.x, point.y, 16, 12, color, 0.85).setStrokeStyle(2, strokeColor);
  const glowA = this.add.circle(point.x - 5, point.y - 3, 2, 0xffffff, 0.5);
  const glowB = this.add.circle(point.x + 5, point.y + 2, 2, 0xffffff, 0.4);
  return [crystal, glowA, glowB];
}
```

## 5. 删除旧字段

- 删除 `point.roomId` 字段（generateGrabPoints 中）
- `roomId` 不再使用，改用 `getZoneByY(point.y)`

## 6. 动态装饰（update 中简单动画）

在 `update()` 方法末尾（return 之前）添加：

```js
// 根据猫当前高度更新动态装饰
const catZone = getZoneByY(this.cat.y);
this.updateAtmosphere(catZone);
```

新增 `updateAtmosphere(zone)` 方法：

```js
updateAtmosphere(zone) {
  // 云层区：云朵呼吸动画（轻微缩放/上下浮动）
  // 高空区：星星闪烁（alpha 脉动）
  // 平流层：极光飘动（水平偏移）
  // 这些由 Phaser tween 在 create 时预置好，这里只触发
}
```

实际上这些在 create 时通过 tween 更好，不需要 update 循环。简化：在 drawZoneDecor 中创建时直接启动 tween。

```js
// 在 drawZoneDecor 中 zone===3 时：
if (!this._cloudsTweened) {
  this.tweens.add({
    targets: cloudElements,
    y: '+=8',  // 小幅上下浮动
    duration: 3000 + Math.random() * 2000,
    yoyo: true,
    repeat: -1,
    ease: 'Sine.easeInOut'
  });
}
```

## 最终校验清单

- [ ] getZoneByY 正确映射 y 坐标到 6 个 zone
- [ ] createRooms 为每个房间正确调用 drawZoneDecor
- [ ] drawZoneDecor 在所有 zone 都有装饰（不会跳过或报错）
- [ ] drawGrabPoints 用 getZoneByY 替代 roomId
- [ ] 所有旧 roomId 引用被移除
- [ ] 动态装饰（云浮动、星星闪烁）视觉上顺畅

## 文件

只修改 `/tmp/cat-tale/index.html`。
