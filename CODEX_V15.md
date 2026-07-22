# CODEX V15 — 方向修复 + 障碍物密度提升

先完整读 index.html 理解 V14 实现。

## Bug 修复

蓄力跳的方向被物理计算覆盖了。现在用户点了左边，但蓄力跳纯算 N+2 的落点速度，无视用户方向。

**修复要求**：`handleChargedJump()` 必须尊重用户的 `chargeDirection`，不要用 `dx / flightTime` 算 velocityX。用 `chargeDirection * CHARGE_JUMP_VX` 代替。

短跳（`handleJump`）的方向没问题，不要动。

## 障碍物密度

当前概率偏低。改成：
- 500m~2km: 3% → 8%
- 2km~10km: 5% → 12%
- 10km+: 8% → 15%
- `OBSTACLE_MIN_VERTICAL_GAP` 从 1950 缩到 1200，让障碍物出现更频繁

## 文件

只改 `/tmp/cat-tale/index.html`。
