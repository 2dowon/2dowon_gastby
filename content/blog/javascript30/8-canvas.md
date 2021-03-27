---
title: JavaScript30 - 8. Fun with HTML5 Canvas
date: 2021-03-27 17:03:84
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

<img src="./image/canvas.gif"  width="800"/>

[Fun with HTML5 Canvas 프로젝트](https://2dowon.github.io/JavaScript30/html/08.html)의 핵심은 다음과 같다.

- canvas에 선 그리기
- 선 색상이 무지개색으로 변하기
- 선 두께가 두꺼워졌다가 얇아졌다가를 반복하기

## canvas에 선 그리기

- canvas에서 마우스가 클릭된채로 움직이면 그리는 상태이고, 아니면 그리지 않는 상태로 만들어준다.
- moveTo를 이용해 현재 그리고자 하는 곳까지 x, y를 이동하고, lineTo를 이용해 선을 그려준다

### ✍️ My Code

- 예전에 [노마드코더의 바닐라 JS로 그림판 만들기 강의](https://nomadcoders.co/javascript-for-beginners-2)를 들은 적이 있어 그 때 코드를 참고했다.
- resize 함수를 만들어, 윈도우가 resize될 때마다 캔버스 크기에 반영될 수 있도록 한다
- mousemove 이벤트를 등록해, onMouseMove 함수에서 마우스가 움직일 때마다 x, y 위치를 체크한다. 이때 painting이 true면 x, y까지 선을 그리고, false면 x, y까지 위치를 이동시킨다.

```jsx
const canvas = document.querySelector("#draw");
const ctx = canvas.getContext("2d");

ctx.strokeStyle = "#2c2c2c";
let painting = false;

function resize() {
  stageWidth = document.body.clientWidth;
  stageHeight = document.body.clientHeight;

  canvas.width = stageWidth;
  canvas.height = stageHeight;
}

function startPainting() {
  painting = true;
}

function stopPainting() {
  painting = false;
}

function onMouseMove(event) {
  const x = event.offsetX;
  const y = event.offsetY;
  if (!painting) {
    ctx.beginPath();
    ctx.moveTo(x, y);
  } else {
    ctx.lineTo(x, y);
    ctx.stroke();
  }
}

window.addEventListener("resize", resize);
canvas.addEventListener("mousemove", onMouseMove);
canvas.addEventListener("mousedown", startPainting);
canvas.addEventListener("mouseup", stopPainting);
canvas.addEventListener("mouseout", stopPainting);
resize();
```

### 👍 Wes Bos Code

- 마우스가 움직일 때마다 x, y 위치를 확인해서 움직이는 것이 아니라 클릭됐을 때만 x, y 위치를 찾고 마우스가 움직이면 그 때 그 위치로 움직여서 선을 그리는 방식이다.
- 아래 코드에서는 처음에만 canvas 크기를 설정하기 때문에 중간에 창의 사이즈가 변하면 반영되지 않는다.
- lineCap과 lineJoin을 둘 다 round로 해서 선의 끝이 동그랗게 그려지고, 선이 부드럽게 이어지도록 만든다.

```jsx
const canvas = document.querySelector("#draw");
const ctx = canvas.getContext("2d");
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;
ctx.strokeStyle = "#BADA55";
ctx.lineJoin = "round";
ctx.lineCap = "round";
ctx.lineWidth = 100;

let isDrawing = false;
let lastX = 0;
let lastY = 0;

function draw(e) {
  if (!isDrawing) return;
  ctx.beginPath();
  ctx.moveTo(lastX, lastY);
  ctx.lineTo(e.offsetX, e.offsetY);
  ctx.stroke();
  [lastX, lastY] = [e.offsetX, e.offsetY];
}

canvas.addEventListener("mousedown", (e) => {
  isDrawing = true;
  [lastX, lastY] = [e.offsetX, e.offsetY];
});

canvas.addEventListener("mousemove", draw);
canvas.addEventListener("mouseup", () => (isDrawing = false));
canvas.addEventListener("mouseout", () => (isDrawing = false));
```

## 선 색상이 무지개색으로 변하기

- [HSL](http://jun.hansung.ac.kr/CWP/htmls/html%20hsl%20colors.html)(hue 색조, saturation 채도, lightness 명도)를 이용하면 색상을 쉽게 무지개색처럼 바꿀 수 있다.
- saturation = 100%, lightness = 50%로 고정인 상황에서 hue만 0~360으로 이동하면 색이 빨주노초파남보를 거쳐서 다시 빨간색이 된다.

### ✍️ My Code

- 처음에는 어떻게 색상이 바뀌게 해야 하는지 전혀 감이 안와서 빨주노초파남보 색상을 colors 배열로 만든 다음 선을 그릴 때마다 색상을 하나씩 정해주도록 만들었다.
- 그리고 그 결과는 당연히 실패였다. 그려질 때마다 색상이 변하는 것이 아니라 선 하나의 전체 색상이 계속 변했기 때문이다.

```jsx
colors = [
  "#FF0000",
  "#FF7F00",
  "#FFDB00",
  "#4CFF00",
  "#00F5FF",
  "#0006FF",
  "#A900FF",
];
colorNum = 0;

function onMouseMove(event) {
  const x = event.offsetX;
  const y = event.offsetY;
  if (!painting) {
    ctx.beginPath();
    ctx.moveTo(x, y);
  } else {
    if (colorNum == 7) {
      colorNum = 0;
    }
    ctx.strokeStyle = colors[colorNum];
    ctx.lineTo(x, y);
    ctx.stroke();
    colorNum++;
  }
}
```

### 👍 Wes Bos Code

```jsx
let hue = 0;

function draw(e) {
  hue++;
  if (hue >= 360) {
    hue = 0;
  }
}
```

## 선 두께가 두꺼워졌다가 얇아졌다가를 반복하기

### ✍️ My Code

- ctx.lineWidth가 100보다 작으면 1씩 늘어나고, 100보다 크면 1씩 줄어들게 만들면 된다는 생각에 아래처럼 코드를 작성했고, 처음에 1부터 시작해 100까지 1씩 늘다가 그 다음부터는 100과 101을 번갈아 움직이는 코드가 완성되었다... 😂
- 너무 심각한 상태의 코드라 부끄러워서 사실 적을까 말까 고민했는데, 나중에 돌아보면 이땐 이렇게 밖에 못 생각하던 순간이 있었지.. 라는 기록으로 일단 남겨두기로 했다.

```jsx
function draw(e) {
  if (ctx.lineWidth >= 100) {
    ctx.lineWidth--;
  } else {
    ctx.lineWidth++;
  }
}
```

### 👍 Wes Bos Code

- ctx.lineWidth가 1부터 100까지 증가하다가 100이 되면 다시 1까지 감소하도록 만들기 위해서는 증가와 감소의 방향을 알려줄 변수가 하나 필요하다.
- 따라서 direction이라는 변수를 하나 만들어 100이 되는 순간과 1이 되는 순간에 direction을 반대로 해주어서 증가에서 감소로, 감소에서 증가가 되도록 만들어줘야 한다.

```jsx
let direction = true;

function draw(e) {
  if (ctx.lineWidth >= 100 || ctx.lineWidth <= 1) {
    direction = !direction;
  }

  if (direction) {
    ctx.lineWidth++;
  } else {
    ctx.lineWidth--;
  }
}
```

## (+)

### [globalCompositeOperation](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation)

```jsx
ctx.globalCompositeOperation = type;
```

- 포토샵의 블렌드 모드처럼 색이 겹쳐질 때마다 어떻게 표현할지 type 중에 고를 수 있다.
- `type` : source-over, source-in, source-out, source-atop, destination-over, destination-in, destination-out, destination-atop, lighter, copy, xor, multipy, screen, overlay, darken, lighten, color-dodge, color-burn, hard-light, soft-light, difference, exclusion, hu, saturation, color, luminosity

### resize

- resize를 이용해서 화면이 리사이즈 될 때마다 캔버스의 사이즈를 조정해주려고 했는데, 이상하게 resize 이벤트가 발생할 때마다 lineCap, lineJoin이 제대로 적용되지 않아서 아래 최종 코드에서는 제거했다.

```jsx
function resize() {
  stageWidth = document.body.clientWidth;
  stageHeight = document.body.clientHeight;

  canvas.width = stageWidth;
  canvas.height = stageHeight;
}
window.addEventListener("resize", resize);
```

# 최종 코드

> index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>HTML5 Canvas</title>
    <link rel="stylesheet" href="../css/08.css" />
  </head>
  <body>
    <h1 class="title">🎨</h1>
    <canvas class="canvas" width="800" height="800"></canvas>
    <script>
      const canvas = document.querySelector(".canvas");
      const ctx = canvas.getContext("2d");

      const canvasWidth = window.innerWidth - 200;
      const canvasHeight = window.innerHeight - 200;
      canvas.width = canvasWidth;
      canvas.height = canvasHeight;
      canvas.style.width = `${canvasWidth}px`;
      canvas.style.height = `${canvasHeight}px`;

      ctx.strokeStyle = "#BADA55";
      ctx.lineJoin = "round";
      ctx.lineCap = "round";
      ctx.lineWidth = 100;

      let isDrawing = false;
      let lastX = 0;
      let lastY = 0;
      let hue = 0;
      let direction = true;

      function draw(e) {
        if (!isDrawing) return;
        ctx.strokeStyle = `hsl(${hue}, 100%, 50%)`;
        ctx.beginPath();
        ctx.moveTo(lastX, lastY);
        ctx.lineTo(e.offsetX, e.offsetY);
        ctx.stroke();
        [lastX, lastY] = [e.offsetX, e.offsetY];

        hue++;
        if (hue >= 360) {
          hue = 0;
        }
        if (ctx.lineWidth >= 100 || ctx.lineWidth <= 1) {
          direction = !direction;
        }

        if (direction) {
          ctx.lineWidth++;
        } else {
          ctx.lineWidth--;
        }
      }

      canvas.addEventListener("mousedown", (e) => {
        isDrawing = true;
        [lastX, lastY] = [e.offsetX, e.offsetY];
      });

      canvas.addEventListener("mousemove", draw);
      canvas.addEventListener("mouseup", () => (isDrawing = false));
      canvas.addEventListener("mouseout", () => (isDrawing = false));
    </script>
  </body>
</html>
```

> style.css

```css
* {
  box-sizing: border-box;
}

body {
  background-color: #1d1d20;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.title {
  font-size: 50px;
}

.canvas {
  background-color: white;
  border-radius: 15px;
}
```

</br>

# Ref.

- [Let's build something fun with HTML5 Canvas - #JavaScript30 8/30](https://www.youtube.com/watch?v=8ZGAzJ0drl0&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=8)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)
