---
title: JavaScript30 - 1. JavaScript Drum Kit
date: 2021-03-20 15:03:82
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

<img src="./image/drum.gif"  width="800"/>

JavaScript Drum Kit 프로젝트의 핵심은 다음과 같다.

1. Keyboard Event를 통해 어떤 key가 눌렸는지 알아내서 해당 object 찾기
2. 해당 key에 맞는 음악 재생하기
3. 해당 key에 맞는 CSS 효과 적용하기

</br>

## Keyboard Event를 통해 어떤 key가 눌렸는지 알아내서 해당 object 찾기

Keyboard Event 중 [keydown event](https://developer.mozilla.org/en-US/docs/Web/API/Document/keydown_event)의 keyCode를 이용하면 어떤 key가 눌렸는지 알 수 있다.

### ✍️ My Code

- 강의 듣기 전에는 HTML에서 key, audio를 다 찾아서 forEach를 통해 dataset.key가 keydown의 keyCode와 같은지 확인했다

```jsx
const key = document.querySelectorAll(".key");
const audios = document.querySelectorAll("audio");

function playSound(char) {
  audios.forEach((audio) => {
    if (audio.dataset.key == char) {
      // 오디오 재생
    }
  });
}

function addEffect(char) {
  key.forEach((k) => {
    if (k.dataset.key == char) {
      // playing 클래스 추가해서 CSS 적용
    }
  });
}

document.addEventListener("keydown", (e) => {
  playSound(e.keyCode);
  addEffect(e.keyCode);
});
```

### 👍 Wes Bos Code

- 강의에서는 keydown Event가 발생하면 playSound 함수에서 audio 태그와 div 태그 중 data-key 값이 keyCode와 같은 요소를 가져왔다.
- 만약 엉뚱한 키를 눌렀다면 audio, key는 null 값이 되므로 함수를 early return한다.

```jsx
function playSound(e) {
  const audio = document.querySelector(`audio[data-key="${e.keyCode}"]`);
  const key = document.querySelector(`div[data-key="${e.keyCode}"]`);
  if (!audio) return;
}

window.addEventListener("keydown", playSound);
```

## 해당 key에 맞는 음악 재생하기

음악을 재생하기 위해서는 Audio 객체를 생성하거나 이미 HTML 파일에 Audio 객체가 있다면 이를 가져와서 `play()` 메서드를 이용하면 된다.

### ✍️ My Code

- Audio 객체를 생성해서 음악을 재생
- key가 눌리 때마다 Audio 객체를 생성하기 때문에 반복해서 클릭해도 음악이 제대로 재생된다.

```jsx
const audios = document.querySelectorAll("audio");

function playSound(char) {
  audios.forEach((audio) => {
    if (audio.dataset.key == char) {
      const sound = new Audio(audio.src);
      sound.play();
    }
  });
}

document.addEventListener("keydown", (e) => {
  playSound(e.keyCode);
});
```

### 👍 Wes Bos Code

- 제공된 HTML 파일에는 이미 Audio 객체가 있었기 때문에 해당 객체를 가져와서 play() 메서드를 이용하면 재생할 수 있다.
- 하지만 이 경우 반복해서 클릭하면 이미 재생되고 있는 효과음이 끝난 다음에 재생되기 때문에 클릭할 때마다 currentTime을 0으로 만들어서 처음부터 재생될 수 있도록 만들어줘야 한다.

```jsx
function playSound(e) {
  const audio = document.querySelector(`audio[data-key="${e.keyCode}"]`);
  const key = document.querySelector(`div[data-key="${e.keyCode}"]`);
  if (!audio) return;

  audio.currentTime = 0;
  audio.play();
}

window.addEventListener("keydown", playSound);
```

## 해당 key에 맞는 CSS 효과 적용하기

key가 눌릴 때마다 적용되는 css는 playing 이라는 이름의 클래스로 만들어져 있다. 따라서 key가 눌리면 해당 key에 playing 클래스를 추가하고, key가 눌린 후에는 해당 효과가 없어져야 하므로 다시 playing 클래스를 없애줘야 한다.

### ✍️ My Code

- key가 눌렸을 때 해당 key에 playing 클래스를 추가
- 잠깐 적용되었다가 다시 해당 효과가 없어져야 하기 때문에 setTimeout을 이용해 클래스를 추가하고 0.1초 후에 클래스를 없애는 함수를 실행한다.

  ⚠️ 아래 Wes Bos Code를 통해 이 경우 왜 setTimeout을 이용하면 안되는지 설명!

```jsx
const key = document.querySelectorAll(".key");

function addEffect(char) {
  key.forEach((k) => {
    if (k.dataset.key == char) {
      k.classList.add("playing");
      setTimeout(removeEffect, 100);
    }
  });
}

function removeEffect() {
  key.forEach((k) => {
    if (k.classList.contains("playing")) {
      k.classList.remove("playing");
    }
  });
}

document.addEventListener("keydown", (e) => {
  addEffect(e.keyCode);
});
```

### 👍 Wes Bos Code

- 키가 눌렀을 때 효과를 주기 위해서 setTimeout을 사용하는 것은 좋지 않다. 만약 setTimeout을 이용할 경우 trasition이 적용되는 시간마다 setTimeout의 시간도 수정해줘야 하기 때문이다. 예를 들어서 현재는 key의 trasition이 0.07초가 걸리기 때문에 setTimeout을 0.1초 후에 작동시키면 0.03초만 보이게 되어서 원하는 효과가 적용된 것처럼 보이지만 만약 trasition을 2초로 늘리게 되면 trasition이 보이기 전에 setTimeout이 실행되어서 아무 효과가 적용되지 않은 것처럼 보인다. 즉, trasition을 수정할 때마다 setTimeout도 수정해줘야 하는 문제가 생긴다.
- 그래서 setTimeout 대신 addEventListener에 transitionend를 주면 css에서 transitionend된 요소들이 모두 나오게 되고, 이 때 우리가 키가 눌렀을 때만 적용되는 변화에만 transform이 있기 때문에 해당 propertyName에 transform이 있는 경우에만 playing 클래스를 없애주면 된다

```jsx
function removeTransition(e) {
  if (e.propertyName !== "transform") return;
  e.target.classList.remove("playing");
}

function playSound(e) {
  const audio = document.querySelector(`audio[data-key="${e.keyCode}"]`);
  const key = document.querySelector(`div[data-key="${e.keyCode}"]`);
  if (!audio) return;

  key.classList.add("playing");
}

const keys = Array.from(document.querySelectorAll(".key"));
keys.forEach((key) => key.addEventListener("transitionend", removeTransition));
window.addEventListener("keydown", playSound);
```

# 최종 코드

> index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>JS Drum Kit</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <div class="keys">
      <div data-key="65" class="key">
        <kbd>A</kbd>
        <span class="sound">clap</span>
      </div>
      <div data-key="83" class="key">
        <kbd>S</kbd>
        <span class="sound">hihat</span>
      </div>
      <div data-key="68" class="key">
        <kbd>D</kbd>
        <span class="sound">kick</span>
      </div>
      <div data-key="70" class="key">
        <kbd>F</kbd>
        <span class="sound">openhat</span>
      </div>
      <div data-key="71" class="key">
        <kbd>G</kbd>
        <span class="sound">boom</span>
      </div>
      <div data-key="72" class="key">
        <kbd>H</kbd>
        <span class="sound">ride</span>
      </div>
      <div data-key="74" class="key">
        <kbd>J</kbd>
        <span class="sound">snare</span>
      </div>
      <div data-key="75" class="key">
        <kbd>K</kbd>
        <span class="sound">tom</span>
      </div>
      <div data-key="76" class="key">
        <kbd>L</kbd>
        <span class="sound">tink</span>
      </div>
    </div>

    <audio data-key="65" src="sounds/clap.wav"></audio>
    <audio data-key="83" src="sounds/hihat.wav"></audio>
    <audio data-key="68" src="sounds/kick.wav"></audio>
    <audio data-key="70" src="sounds/openhat.wav"></audio>
    <audio data-key="71" src="sounds/boom.wav"></audio>
    <audio data-key="72" src="sounds/ride.wav"></audio>
    <audio data-key="74" src="sounds/snare.wav"></audio>
    <audio data-key="75" src="sounds/tom.wav"></audio>
    <audio data-key="76" src="sounds/tink.wav"></audio>

    <script>
      function removeTransition(e) {
        if (e.propertyName !== "transform") return;
        e.target.classList.remove("playing");
      }

      function playSound(e) {
        const audio = document.querySelector(`audio[data-key="${e.keyCode}"]`);
        const key = document.querySelector(`div[data-key="${e.keyCode}"]`);
        if (!audio) return;

        key.classList.add("playing");
        audio.currentTime = 0;
        audio.play();
      }

      const keys = Array.from(document.querySelectorAll(".key"));
      keys.forEach((key) =>
        key.addEventListener("transitionend", removeTransition)
      );
      window.addEventListener("keydown", playSound);
    </script>
  </body>
</html>
```

> style.css

```css
html {
  font-size: 10px;
  background: url("./background.jpg") bottom center;
  background-size: cover;
}

body,
html {
  margin: 0;
  padding: 0;
  font-family: sans-serif;
}

.keys {
  display: flex;
  flex: 1;
  min-height: 100vh;
  align-items: center;
  justify-content: center;
}

.key {
  border: 0.4rem solid black;
  border-radius: 0.5rem;
  margin: 1rem;
  font-size: 1.5rem;
  padding: 1rem 0.5rem;
  transition: all 0.07s ease;
  width: 10rem;
  text-align: center;
  color: white;
  background: rgba(0, 0, 0, 0.4);
  text-shadow: 0 0 0.5rem black;
}

.playing {
  transform: scale(1.1);
  border-color: #ffc600;
  box-shadow: 0 0 1rem #ffc600;
}

kbd {
  display: block;
  font-size: 4rem;
}

.sound {
  font-size: 1.2rem;
  text-transform: uppercase;
  letter-spacing: 0.1rem;
  color: #ffc600;
}
```

</br>

# Ref.

- [Make a JavaScript Drum Kit in Vanilla JS! #JavaScript30 1/30](https://www.youtube.com/watch?v=VuN8qwZoego)

- [2dowon/JavaScript30](https://github.com/2dowon/JavaScript30/tree/main/01%20-%20JavaScript%20Drum%20Kit)
