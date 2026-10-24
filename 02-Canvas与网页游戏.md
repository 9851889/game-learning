# 02 · Canvas 与网页游戏

> 目标：能独立写出「一个会动、能用键盘控制、能碰到东西」的画面。

---

## 1. Canvas 是什么

Canvas（画布）就是一块空白像素板，你在上面画什么它就显示什么。

```html
<canvas id="game" width="400" height="300"></canvas>
```

拿它的「笔」（绘图上下文）：

```js
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
```

`ctx` 提供所有画图方法：画矩形、圆、文字、线、图片……

---

## 2. 画东西

```js
// 矩形
ctx.fillStyle = 'skyblue';
ctx.fillRect(50, 50, 100, 80);      // x, y, 宽, 高

// 圆
ctx.beginPath();
ctx.arc(200, 150, 40, 0, Math.PI * 2);   // 圆心x, 圆心y, 半径, 起始角, 结束角
ctx.fillStyle = 'orange';
ctx.fill();

// 文字
ctx.font = '20px sans-serif';
ctx.fillText('得分：0', 10, 25);
```

⚠️ **每帧开始要清空画布**，否则画面会叠加成一团糊：

```js
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  // ... 在这里画
}
```

---

## 3. 让它动起来

```js
const ball = { x: 200, y: 150, r: 20, vx: 120, vy: 80 };

function update(dt) {
  ball.x += ball.vx * dt;
  ball.y += ball.vy * dt;

  // 撞墙反弹
  if (ball.x < ball.r || ball.x > canvas.width - ball.r) ball.vx = -ball.vx;
  if (ball.y < ball.r || ball.y > canvas.height - ball.r) ball.vy = -ball.vy;
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.beginPath();
  ctx.arc(ball.x, ball.y, ball.r, 0, Math.PI * 2);
  ctx.fillStyle = 'orange';
  ctx.fill();
}
```

---

## 4. 键盘输入

用一张表记住当前哪些键按着：

```js
const keys = {};

window.addEventListener('keydown', e => {
  keys[e.key] = true;
  if (e.key === ' ') e.preventDefault();   // 空格会滚动页面，先挡掉
});
window.addEventListener('keyup', e => {
  keys[e.key] = false;
});
```

在 `update` 里读：

```js
if (keys['ArrowLeft']  || keys['a']) player.vx -= 600 * dt;
if (keys['ArrowRight'] || keys['d']) player.vx += 600 * dt;
```

> 💡 `e.key` 是按键的**字符**：`'a'`、`'ArrowLeft'`、`' '`（空格）。想用字母码就得用 `e.code`（`'KeyA'`、`'Space'`）。

---

## 5. 碰撞检测

### 矩形 vs 矩形（AABB）

两个矩形重叠的判断：**四个边都不互相穿过**。

```js
function hitRect(a, b) {
  return a.x < b.x + b.w &&   // a 的左边在 b 的右边之外
         a.x + a.w > b.x &&   // a 的右边在 b 的左边之外
         a.y < b.y + b.h &&
         a.y + a.h > b.y;
}
```

### 圆 vs 圆

两圆心距离小于半径和就碰到了：

```js
function hitCircle(a, b) {
  const dx = a.x - b.x;
  const dy = a.y - b.y;
  const r = a.r + b.r;
  return dx * dx + dy * dy <= r * r;   // 不用开方，省性能
}
```

---

## 6. 随机数

```js
Math.random();                      // 0 ~ 1 之间的小数
Math.floor(Math.random() * 10);     // 0 ~ 9 的整数
Math.random() * (max - min) + min;  // min ~ max 之间
```

---

## 7. 一个能跑的最小骨架

把上面所有东西拼起来，就是一个完整游戏的骨架：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>我的第一个游戏</title>
  <style>body { background:#1e1e2e; }</style>
</head>
<body>
  <canvas id="game" width="400" height="300"></canvas>
  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const keys = {};

    window.addEventListener('keydown', e => { keys[e.key] = true; });
    window.addEventListener('keyup',   e => { keys[e.key] = false; });

    const ball = { x: 200, y: 150, r: 20, vx: 120, vy: 80 };

    function update(dt) {
      ball.x += ball.vx * dt;
      ball.y += ball.vy * dt;
      if (ball.x < ball.r || ball.x > canvas.width - ball.r) ball.vx = -ball.vx;
      if (ball.y < ball.r || ball.y > canvas.height - ball.r) ball.vy = -ball.vy;
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.beginPath();
      ctx.arc(ball.x, ball.y, ball.r, 0, Math.PI * 2);
      ctx.fillStyle = 'orange';
      ctx.fill();
    }

    let last = performance.now();
    function loop(now) {
      const dt = (now - last) / 1000;
      last = now;
      update(dt);
      draw();
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  </script>
</body>
</html>
```

把这个存成 `.html`，双击用浏览器打开，小球就在弹了。🎉

---

## 本节要点自测

1. `ctx.fillRect(50, 50, 100, 80)` 里四个数字各是什么？
2. 为什么每帧要 `clearRect`？
3. `keys` 这张表是怎么记录按键状态的？
4. 圆和圆碰撞的判断条件用一句话怎么说？

---

**上一节** → [01 · 游戏开发基础](01-游戏开发基础.md)
**下一节** → [03 · 第一个游戏实战](03-第一个游戏实战.md)：把这些拼成一个真正能玩的游戏。
