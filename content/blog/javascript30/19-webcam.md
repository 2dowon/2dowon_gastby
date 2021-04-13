---
title: JavaScript30 - 19. Webcam Fun
date: 2021-04-13 17:04:75
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

[Webcam Fun](https://2dowon.github.io/JavaScript30/html/19.html) 프로젝트의 핵심은 다음과 같다.

- 브라우저에 Webcam 화면 표시하기
- Webcam을 이용해 캡쳐하고, 저장하기
- 필터 효과 적용하기

## 브라우저에 Webcam 화면 표시하기

```jsx
const video = document.querySelector(".player");
const canvas = document.querySelector(".photo");
const ctx = canvas.getContext("2d");
const strip = document.querySelector(".strip");
const snap = document.querySelector(".snap");

function getVideo() {
  navigator.mediaDevices
    .getUserMedia({ video: true, audio: false })
    .then((localMediaStream) => {
      console.log(localMediaStream);

      video.srcObject = localMediaStream;
      video.play();
    })
    .catch((err) => {
      console.error(`OH NO!!!`, err);
    });
}

function paintToCanvas() {
  const width = video.videoWidth;
  const height = video.videoHeight;
  canvas.width = width;
  canvas.height = height;

  return setInterval(() => {
    ctx.drawImage(video, 0, 0, width, height);
  }, 16);
}

getVideo();

video.addEventListener("canplay", paintToCanvas);
```

## Webcam을 이용해 캡쳐하고, 저장하기

- snap은 html에서 만들었던 audio로 찰칵 하는 사운드이다.
- 이미지를 저장하는 것은 예전에 그림판을 만들면서 저장했던 기능이랑 거의 같은 코드이다. 먼저 캔버스의 이미지에 접근할 수 있도록 `canvas.toDataURL()` 을 이용한다. 그리고 `a` 태그를 만들어서 링크에 아까 만들었더너 url을 준다. 그리고 다운로드 속성을 이용하면 다운로드되는 파일의 이름을 수정할 수 있다.
- `innerHTML` 은 나중에 브라우저에서 캡쳐한 사진들을 따로 보여주기 위해서 사용하는 것이다.

```jsx
function takePhoto() {
  // played the sound
  snap.currentTime = 0;
  snap.play();

  // take the data out of the canvas
  const image = canvas.toDataURL();
  const link = document.createElement("a");
  link.href = image;
  link.download = "Selfie";
  link.innerHTML = `<img src="${image}" alt="Selfie" />`;
  strip.insertBefore(link, strip.firstChild);
}
```

## 필터 효과 적용하기

### redEffect

- 붉은 톤의 영상을 만들기 위한 함수
- `pixels.data` 는 R, G, B Alpha 값으로 이루어져 있기 때문에 Red 값은 올리고, 나머지 Green, Blue 값은 낮춰서 붉은 톤으로 만든다.

```jsx
function redEffect(pixels) {
  for (let i = 0; i < pixels.data.length; i += 4) {
    pixels.data[i + 0] = pixels.data[i + 0] + 200; // RED
    pixels.data[i + 1] = pixels.data[i + 1] - 50; // GREEN
    pixels.data[i + 2] = pixels.data[i + 2] * 0.5; // Blue
  }
  return pixels;
}
```

### rgbSplit

- R, G, B 레이어를 각각 분리하기 위한 함수
- 레이어 각각의 인덱스 값을 이용해 색상 레이어의 거리를 조절해서 레이어를 분리할 수 있다

```jsx
function rgbSplit(pixels) {
  for (let i = 0; i < pixels.data.length; i += 4) {
    pixels.data[i - 150] = pixels.data[i + 0]; // RED
    pixels.data[i + 500] = pixels.data[i + 1]; // GREEN
    pixels.data[i - 550] = pixels.data[i + 2]; // Blue
  }
  return pixels;
}
```

### greenScreen

- 특정 범위의 색상을 투명하게 만드는 함수
- 이 기능을 이용하려면 나중에 html 파일에서 주석 처리되어있는 슬라이더 부분 살리기!

```jsx
function greenScreen(pixels) {
  const levels = {};

  document.querySelectorAll(".rgb input").forEach((input) => {
    levels[input.name] = input.value;
  });

  for (i = 0; i < pixels.data.length; i = i + 4) {
    red = pixels.data[i + 0];
    green = pixels.data[i + 1];
    blue = pixels.data[i + 2];
    alpha = pixels.data[i + 3];

    if (
      red >= levels.rmin &&
      green >= levels.gmin &&
      blue >= levels.bmin &&
      red <= levels.rmax &&
      green <= levels.gmax &&
      blue <= levels.bmax
    ) {
      // take it out!
      pixels.data[i + 3] = 0;
    }
  }

  return pixels;
}
```

### 위 필터들을 적용한 paintToCanvas

> rgbSplit 적용

```jsx
function paintToCanvas() {
  const width = video.videoWidth;
  const height = video.videoHeight;
  canvas.width = width;
  canvas.height = height;

  return setInterval(() => {
    ctx.drawImage(video, 0, 0, width, height);

    // 필터 적용을 위해 추가된 부분
    let pixels = ctx.getImageData(0, 0, width, height);
    pixels = rgbSplit(pixels);
    // pixels = redEffect(pixels); // redEffect 적용
    // pixels = greenScreen(pixels); // greenScreen 적용

    ctx.putImageData(pixels, 0, 0);
  }, 16);
}
```

# 최종 코드

> index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Get User Media Code Along!</title>
    <link rel="stylesheet" href="../css/19.css" />
  </head>
  <body>
    <div class="photobooth">
      <div class="controls">
        <button onClick="takePhoto()">Take Photo</button>
        <!--       <div class="rgb">
        <label for="rmin">Red Min:</label>
        <input type="range" min=0 max=255 name="rmin">
        <label for="rmax">Red Max:</label>
        <input type="range" min=0 max=255 name="rmax">

        <br>

        <label for="gmin">Green Min:</label>
        <input type="range" min=0 max=255 name="gmin">
        <label for="gmax">Green Max:</label>
        <input type="range" min=0 max=255 name="gmax">

        <br>

        <label for="bmin">Blue Min:</label>
        <input type="range" min=0 max=255 name="bmin">
        <label for="bmax">Blue Max:</label>
        <input type="range" min=0 max=255 name="bmax">
      </div> -->
      </div>

      <canvas class="photo"></canvas>
      <video class="player"></video>
      <div class="strip"></div>
    </div>

    <audio class="snap" src="../sounds/snap.mp3" hidden></audio>

    <script src="../js/19.js"></script>
  </body>
</html>
```

> style.css

```css
html {
  box-sizing: border-box;
}

*,
*:before,
*:after {
  box-sizing: inherit;
}

html {
  font-size: 10px;
  background: #ffc600;
}

.photobooth {
  background: white;
  max-width: 150rem;
  margin: 2rem auto;
  border-radius: 2px;
}

/*clearfix*/
.photobooth:after {
  content: "";
  display: block;
  clear: both;
}

.photo {
  width: 100%;
  float: left;
}

.player {
  position: absolute;
  top: 20px;
  right: 20px;
  width: 200px;
}

/*
  Strip!
*/

.strip {
  padding: 2rem;
}

.strip img {
  width: 100px;
  overflow-x: scroll;
  padding: 0.8rem 0.8rem 2.5rem 0.8rem;
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.2);
  background: white;
}

.strip a:nth-child(5n + 1) img {
  transform: rotate(10deg);
}
.strip a:nth-child(5n + 2) img {
  transform: rotate(-2deg);
}
.strip a:nth-child(5n + 3) img {
  transform: rotate(8deg);
}
.strip a:nth-child(5n + 4) img {
  transform: rotate(-11deg);
}
.strip a:nth-child(5n + 5) img {
  transform: rotate(12deg);
}
```

> JS

```jsx
const video = document.querySelector(".player");
const canvas = document.querySelector(".photo");
const ctx = canvas.getContext("2d");
const strip = document.querySelector(".strip");
const snap = document.querySelector(".snap");

function getVideo() {
  navigator.mediaDevices
    .getUserMedia({ video: true, audio: false })
    .then((localMediaStream) => {
      console.log(localMediaStream);

      video.srcObject = localMediaStream;
      video.play();
    })
    .catch((err) => {
      console.error(`OH NO!!!`, err);
    });
}

function paintToCanvas() {
  const width = video.videoWidth;
  const height = video.videoHeight;
  canvas.width = width;
  canvas.height = height;

  return setInterval(() => {
    ctx.drawImage(video, 0, 0, width, height);
    // take the pixels out
    let pixels = ctx.getImageData(0, 0, width, height);
    // mess with them
    // pixels = redEffect(pixels);

    pixels = rgbSplit(pixels);
    // ctx.globalAlpha = 0.8;

    // pixels = greenScreen(pixels);
    // put them back
    ctx.putImageData(pixels, 0, 0);
  }, 16);
}

function takePhoto() {
  // played the sound
  snap.currentTime = 0;
  snap.play();

  // take the data out of the canvas
  const image = canvas.toDataURL();
  const link = document.createElement("a");
  link.href = image;
  link.download = "Selfie";
  link.innerHTML = `<img src="${image}" alt="Selfie" />`;
  strip.insertBefore(link, strip.firstChild);
}

function redEffect(pixels) {
  for (let i = 0; i < pixels.data.length; i += 4) {
    pixels.data[i + 0] = pixels.data[i + 0] + 200; // RED
    pixels.data[i + 1] = pixels.data[i + 1] - 50; // GREEN
    pixels.data[i + 2] = pixels.data[i + 2] * 0.5; // Blue
  }
  return pixels;
}

function rgbSplit(pixels) {
  for (let i = 0; i < pixels.data.length; i += 4) {
    pixels.data[i - 150] = pixels.data[i + 0]; // RED
    pixels.data[i + 500] = pixels.data[i + 1]; // GREEN
    pixels.data[i - 550] = pixels.data[i + 2]; // Blue
  }
  return pixels;
}

function greenScreen(pixels) {
  const levels = {};

  document.querySelectorAll(".rgb input").forEach((input) => {
    levels[input.name] = input.value;
  });

  for (i = 0; i < pixels.data.length; i = i + 4) {
    red = pixels.data[i + 0];
    green = pixels.data[i + 1];
    blue = pixels.data[i + 2];
    alpha = pixels.data[i + 3];

    if (
      red >= levels.rmin &&
      green >= levels.gmin &&
      blue >= levels.bmin &&
      red <= levels.rmax &&
      green <= levels.gmax &&
      blue <= levels.bmax
    ) {
      // take it out!
      pixels.data[i + 3] = 0;
    }
  }

  return pixels;
}

getVideo();

video.addEventListener("canplay", paintToCanvas);
```

</br>

# Ref.

- [Make a JavaScript Drum Kit in Vanilla JS! #JavaScript30 1/30](https://www.youtube.com/watch?v=VuN8qwZoego)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)

- [브라우저로 웹캠 화면 갖고 놀기](https://til-devsong.tistory.com/74)
