---
title: JavaScript30​ - 11. Custom Video Player
date: 2021-03-31 17:03:66
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

<img src="./image/videoPlayer.gif"  width="800"/>

Custom Video Player 프로젝트의 핵심은 다음과 같다.

- Video를 클릭하면 재생되거나 일시정지
- 10초 전, 25초 후로 스킵하는 버튼
- Video 볼륨 및 재생속도 조절
- 진행바에서 Video 재생 시점 선택

> html

```html
<div class="player">
  <video class="player__video viewer" src="../img/11.mp4"></video>
  <div class="player__controls">
    <div class="progress">
      <div class="progress__filled"></div>
    </div>
    <button class="player__button toggle" title="Toggle Play">►</button>
    <input
      type="range"
      name="volume"
      class="player__slider"
      min="0"
      max="1"
      step="0.05"
      value="1"
    />
    <input
      type="range"
      name="playbackRate"
      class="player__slider"
      min="0.5"
      max="2"
      step="0.1"
      value="1"
    />
    <button data-skip="-10" class="player__button">« 10s</button>
    <button data-skip="25" class="player__button">25s »</button>
  </div>
</div>
```

## Video를 클릭하면 재생되거나 일시정지

- Video를 클릭하거나 toggle 버튼을 클릭하면 `togglePlay` 함수를 실행
- `togglePlay` ⇒ 비디오가 정지 상태이면 재생, 재생 상태이면 정지
- 비디오가 재생 상태이면 toggle 버튼 아이콘이 ❚ ❚ 이런 모양이어야 하고, 정지 상태이면 ► 이어야 한다.
- 따라서 비디오의 상태에 따라 `updateButton` 함수를 통해 아이콘 바꿔주기

```jsx
const player = document.querySelector(".player");
const video = player.querySelector(".viewer");
const toggle = player.querySelector(".toggle");

function togglePlay() {
  const method = video.paused ? "play" : "pause";
  video[method]();

  //   if (video.paused) {
  //     video.play();
  //   } else {
  //     video.pause();
  //   }
}

function updateButton() {
  const icon = this.paused ? "►" : "❚ ❚";
  toggle.textContent = icon;
  console.log("update btn");
}

video.addEventListener("click", togglePlay);
video.addEventListener("play", updateButton);
video.addEventListener("pause", updateButton);
toggle.addEventListener("click", togglePlay);
```

## 10초 전, 25초 후로 스킵하는 버튼

- skipButtons에는 10초 전으로 스킵하는 버튼과 25초 후로 스킵하는 버튼 2가지가 있다.
- skipButtons 안에 `data-skip` 속성으로 데이터가 저장되어 있다
- 따라서 스킵 버튼을 누를 때마다 `dataset.skip` 값만큼 `video.currentTime`에 더해주면 된다.
- 이때 currentTime의 타입은 number이므로 더하고자 하는 값의 타입도 nubmer로 형변환해야 한다.

```jsx
const skipButtons = player.querySelectorAll("[data-skip]");

function skip() {
  video.currentTime += Number(this.dataset.skip);
}

skipButtons.forEach((button) => button.addEventListener("click", skip));
```

## Video 볼륨 및 재생속도 조절

- ranges에는 볼륨을 조절하는 range와 재생속도를 조절하는 range 2가지가 있다
- `video[this.name] = this.value;`
  - 각각의 range는 name 속성의 값이 [video에서 사용해야 하는 속성](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)의 이름과 같다. ⇒ `this.name`
    - [HTMLMediaElement.playbackRate](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/playbackRate) : 재생 속도 조절
    - [HTMLMediaElement.volume](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/volume) : 볼륨 조절
  - video 속성에서 조절한 값은 각각의 range가 갖는 값(`this.value`)으로 설정해주면 된다.
- range에 change 이벤트가 발생했을 때와 mousemove 이벤트가 발생했을 때 handleRangeUpdate 함수를 실행해준다. mousemove 이벤트가 발생했을 때 실행해줘야 range를 움직일 때 실시간으로 변경된 값이 적용된다.

```jsx
const ranges = player.querySelectorAll(".player__slider");

function handleRangeUpdate() {
  video[this.name] = this.value;
}

ranges.forEach((range) => range.addEventListener("change", handleRangeUpdate));
ranges.forEach((range) =>
  range.addEventListener("mousemove", handleRangeUpdate)
);
```

## 진행바에서 Video 재생 시점 선택

### Video 재생 시간에 따른 진행바 조절

- Video가 현재 얼마나 재생되었는지 알기 위해서는 `현재 재생된 시간 / 비디오의 총 시간 * 100` 을 이용하면 현재 재생된 %를 알 수 있다. ⇒ `(video.currentTime / video.duration) * 100`
- 진행바의 진행된 영역은 progressBar의 flexBasis에서 조절할 수 있다
- Video가 재생되는 시간에 맞춰서 진행바가 채워져야 하므로 시간이 업데이트될 때마다 진행바의 영역을 채워주는 `handleProgress` 함수를 실행한다.

### 진행바에서 선택한 시점부터 Video 재생

- 진행바가 클릭되면 진행바에서 선택한 시점부터 Video를 재생할 수 있는 `scrub` 함수를 실행한다.
- 진행바에서 클릭된 곳부터 Video가 재생되기 위해서는 진행바의 클릭된 부분이 동영상에서 차지하는 비율을 구해야 한다. 따라서 `선택된 진행바의 길이 / 총 진행바의 길이 * 비디오의 총 시간` 을 통해 구할 수 있다.

  ⇒ `(e.offsetX / progress.offsetWidth) * video.duration`

  그리고 마지막으로 Video의 현재 시각`video.currentTime`을 구한 값으로 바꿔주면 해당 시점부터 비디오가 재생된다.

- 진행바를 드래그해서 움직일 때마다 Video의 재생시점의 변화가 실시간으로 반영되기 위해서는 mousemove 이벤트일 때도 scrub 함수를 실행해줘야 한다. 하지만 이 경우 진행바 위에서 마우스가 움직일 때마다 영상의 재생시점에 변화가 생겨서 불편하다. 즉, 진행바를 클릭해서 움직이는 경우에만 실시간으로 반영되어야 한다. 따라서 mousedown이라는 변수를 통해 mousedown이벤트에서는 해당 변수를 true, mouseup 이벤트에서는 해당 변수를 false값으로 만들어 현재 클릭되어 있는지를 알 수 있다.
- 따라서 mousemove 이벤트가 발생하더라도 mousedwon 변수의 값이 true일 때만 (클릭된 상태) scrub 함수를 실행한다. ⇒ `progress.addEventListener("mousemove", (e) => mousedown && scrub(e));`

  ✅ `A && B` 이면 A가 true일 때만 B를 확인한다. A가 false이면 B의 값에 상관없이 무조건 false이므로

```jsx
const progress = player.querySelector(".progress");
const progressBar = player.querySelector(".progress__filled");

function handleProgress() {
  const percent = (video.currentTime / video.duration) * 100;
  progressBar.style.flexBasis = `${percent}%`;
}

function scrub(e) {
  const scrubTime = (e.offsetX / progress.offsetWidth) * video.duration;
  video.currentTime = scrubTime;
}

video.addEventListener("timeupdate", handleProgress);

let mousedown = false;
progress.addEventListener("click", scrub);
progress.addEventListener("mousemove", (e) => mousedown && scrub(e));
progress.addEventListener("mousedown", () => (mousedown = true));
progress.addEventListener("mouseup", () => (mousedown = false));
```

# 최종 코드

> index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>HTML Video Player</title>
    <link rel="stylesheet" href="../css/11.css" />
  </head>
  <body>
    <div class="player">
      <video class="player__video viewer" src="../img/11.mp4"></video>

      <div class="player__controls">
        <div class="progress">
          <div class="progress__filled"></div>
        </div>
        <button class="player__button toggle" title="Toggle Play">►</button>
        <input
          type="range"
          name="volume"
          class="player__slider"
          min="0"
          max="1"
          step="0.05"
          value="1"
        />
        <input
          type="range"
          name="playbackRate"
          class="player__slider"
          min="0.5"
          max="2"
          step="0.1"
          value="1"
        />
        <button data-skip="-10" class="player__button">« 10s</button>
        <button data-skip="25" class="player__button">25s »</button>
      </div>
    </div>

    <script src="../js/11.js"></script>
  </body>
</html>
```

> 11.css

```css
html {
  box-sizing: border-box;
}

*,
*:before,
*:after {
  box-sizing: inherit;
}

body {
  margin: 0;
  padding: 0;
  display: flex;
  background: #7a419b;
  min-height: 100vh;
  background: linear-gradient(135deg, #7c1599 0%, #921099 48%, #7e4ae8 100%);
  background-size: cover;
  align-items: center;
  justify-content: center;
}

.player {
  max-width: 750px;
  border: 5px solid rgba(0, 0, 0, 0.2);
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
  position: relative;
  font-size: 0;
  overflow: hidden;
}

/* This css is only applied when fullscreen is active. */
.player:fullscreen {
  max-width: none;
  width: 100%;
}

.player:-webkit-full-screen {
  max-width: none;
  width: 100%;
}

.player__video {
  width: 100%;
}

.player__button {
  background: none;
  border: 0;
  line-height: 1;
  color: white;
  text-align: center;
  outline: 0;
  padding: 0;
  cursor: pointer;
  max-width: 50px;
}

.player__button:focus {
  border-color: #ffc600;
}

.player__slider {
  width: 10px;
  height: 30px;
}

.player__controls {
  display: flex;
  position: absolute;
  bottom: 0;
  width: 100%;
  transform: translateY(100%) translateY(-5px);
  transition: all 0.3s;
  flex-wrap: wrap;
  background: rgba(0, 0, 0, 0.1);
}

.player:hover .player__controls {
  transform: translateY(0);
}

.player:hover .progress {
  height: 15px;
}

.player__controls > * {
  flex: 1;
}

.progress {
  flex: 10;
  position: relative;
  display: flex;
  flex-basis: 100%;
  height: 5px;
  transition: height 0.3s;
  background: rgba(0, 0, 0, 0.5);
  cursor: ew-resize;
}

.progress__filled {
  width: 50%;
  background: #ffc600;
  flex: 0;
  flex-basis: 50%;
}

/* unholy css to style input type="range" */

input[type="range"] {
  -webkit-appearance: none;
  background: transparent;
  width: 100%;
  margin: 0 5px;
}

input[type="range"]:focus {
  outline: none;
}

input[type="range"]::-webkit-slider-runnable-track {
  width: 100%;
  height: 8.4px;
  cursor: pointer;
  box-shadow: 1px 1px 1px rgba(0, 0, 0, 0), 0 0 1px rgba(13, 13, 13, 0);
  background: rgba(255, 255, 255, 0.8);
  border-radius: 1.3px;
  border: 0.2px solid rgba(1, 1, 1, 0);
}

input[type="range"]::-webkit-slider-thumb {
  height: 15px;
  width: 15px;
  border-radius: 50px;
  background: #ffc600;
  cursor: pointer;
  -webkit-appearance: none;
  margin-top: -3.5px;
  box-shadow: 0 0 2px rgba(0, 0, 0, 0.2);
}

input[type="range"]:focus::-webkit-slider-runnable-track {
  background: #bada55;
}

input[type="range"]::-moz-range-track {
  width: 100%;
  height: 8.4px;
  cursor: pointer;
  box-shadow: 1px 1px 1px rgba(0, 0, 0, 0), 0 0 1px rgba(13, 13, 13, 0);
  background: #ffffff;
  border-radius: 1.3px;
  border: 0.2px solid rgba(1, 1, 1, 0);
}

input[type="range"]::-moz-range-thumb {
  box-shadow: 0 0 0 rgba(0, 0, 0, 0), 0 0 0 rgba(13, 13, 13, 0);
  height: 15px;
  width: 15px;
  border-radius: 50px;
  background: #ffc600;
  cursor: pointer;
}
```

> 11.js

```jsx
/* Get Our Elements */
const player = document.querySelector(".player");
const video = player.querySelector(".viewer");
const progress = player.querySelector(".progress");
const progressBar = player.querySelector(".progress__filled");
const toggle = player.querySelector(".toggle");
const skipButtons = player.querySelectorAll("[data-skip]");
const ranges = player.querySelectorAll(".player__slider");

/* Build out functions */
function togglePlay() {
  const method = video.paused ? "play" : "pause";
  video[method]();
}

function updateButton() {
  const icon = this.paused ? "►" : "❚ ❚";
  toggle.textContent = icon;
}

function skip() {
  video.currentTime += Number(this.dataset.skip);
}

function handleRangeUpdate() {
  video[this.name] = this.value;
}

function handleProgress() {
  const percent = (video.currentTime / video.duration) * 100;
  progressBar.style.flexBasis = `${percent}%`;
}

function scrub(e) {
  console.log(e);
  const scrubTime = (e.offsetX / progress.offsetWidth) * video.duration;
  video.currentTime = scrubTime;
}

/* Hook up the event listeners */
video.addEventListener("click", togglePlay);
video.addEventListener("play", updateButton);
video.addEventListener("pause", updateButton);
video.addEventListener("timeupdate", handleProgress);

toggle.addEventListener("click", togglePlay);

skipButtons.forEach((button) => button.addEventListener("click", skip));
ranges.forEach((range) => range.addEventListener("change", handleRangeUpdate));
ranges.forEach((range) =>
  range.addEventListener("mousemove", handleRangeUpdate)
);

let mousedown = false;
progress.addEventListener("click", scrub);
progress.addEventListener("mousemove", (e) => mousedown && scrub(e));
progress.addEventListener("mousedown", () => (mousedown = true));
progress.addEventListener("mouseup", () => (mousedown = false));
```

</br>

# Ref.

- [Custom HTML5 Video Player - #JavaScript30​ 11/30](https://www.youtube.com/watch?v=yx-HYerClEA&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=11)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)

- [HTMLMediaElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)
