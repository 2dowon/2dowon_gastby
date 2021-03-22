---
title: JavaScript30 - 3. CSS Variables
date: 2021-03-22 16:03:10
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

<img src="./image/css.gif"  width="800"/>

[CSS Variables 프로젝트](https://2dowon.github.io/JavaScript30/html/03.html)의 핵심은 다음과 같다.

1. css에 기본값 설정
2. JS로 css 값 변경하기 (spacing, blur, base color)

(+) 여기에 로컬 이미지를 업로드하는 기능을 추가했다.

</br>

## css에 기본값 설정

- 나는 js 기본 코드를 처음에 한 번 실행시켜줌으로써 기본 설정을 대체했었다.
- 하지만 Wes Bos는 아래처럼 css에 기본으로 적용되어야 하는 값을 설정해주고 시작했다.
- ✅ 처음에는 왜 이렇게 root값을 설정해서 css 기본 값을 설정하는지 그 이유를 몰랐는데, 아래에 왜 이렇게 작업했는지에 대한 이유가 나온다.

```css
:root {
  --base: #ffc600;
  --spacing: 10px;
  --blur: 10px;
}

img {
  padding: var(--spacing);
  background: var(--base);
  filter: blur(var(--blur));
}

.hl {
  color: var(--base);
}
```

## JS로 css 값 변경하기

### ✍️ My Code

- 처음에 나는 css에 변수를 만드는 작업을 하지 않았다.
- 따라서 css를 조작하고자 하는 모든 요소를 가져와서 하나씩 수정하는 작업을 거쳐야 했다.
- 이 프로젝트에서는 수정하고자 하는 값이 많지 않아서 이렇게 하더라도 크게 상관없어 보이지만, 만약 수정해야 하는 값이 100개라면...? 코드가 엄청 늘어나게 될 것이다. (클래스를 통해서 객체를 생성할 때와 아닐 때의 차이와 비슷한 것 같기도 🤔 )

```jsx
const img = document.querySelector("img");
const spacing = document.querySelector("#spacing");
const blur = document.querySelector("#blur");
const color = document.querySelector("#base");
const js = document.querySelector(".hl");

function updateSpacing() {
  const colorCode = color.value;
  js.style.color = colorCode;
  img.style.border = `solid ${colorCode} ${spacing.value}px`;
}

function updateBlur() {
  img.style.filter = `blur(${blur.value}px)`;
}

spacing.addEventListener("input", updateSpacing);
color.addEventListener("input", updateSpacing);
blur.addEventListener("input", updateBlur);
```

### 👍 Wes Bos Code

- 모든 input 태그들을 다 가져와서 inputs 유사배열에 넣고, forEach를 통해 change 또는 mousemove가 일어나는 input에만 handleUpdate 함수를 실행한다.
  - change ⇒ input 값이 변화했을 때
  - mousemove ⇒ mousemove 이벤트를 추가하지 않으면 좌우로 움직일 때는 값이 변하지 않기 때문에 효과가 적용되지 않는다. 따라서 이를 추가함으로써 range 값을 클릭해서 좌우로 움직일 때도 handleUpdaete 함수가 실행되어 그 값이 바로 바로 적용될 수 있도록 한다.
- blur나 border의 경우 px 단위가 필요하기 때문에 필요한 요소에는 data-sizing 옵션 값을 추가되어 있다.

  ```html
  <label for="spacing">Spacing:</label>
  <input
    id="spacing"
    type="range"
    name="spacing"
    min="10"
    max="200"
    value="10"
    data-sizing="px"
  />

  <label for="blur">Blur:</label>
  <input
    id="blur"
    type="range"
    name="blur"
    min="0"
    max="25"
    value="10"
    data-sizing="px"
  />
  ```

  base color의 경우 따로 필요하지 않으므로 해당 값이 없다. 따라서 suffix를 가져올 때 dataset.sizing이 있다면 가져오고, 없다면 아무 것도 가져오지 않으면 된다.

- 조작하고자 하는 css 값은 모두 root에 담아놨다. 이 때 조작하고자 하는 요소의 name과 css 변수 이름을 같게 설정해서 같은 이름의 css를 수정해주면 된다.
- `root` : 최상위 엘리먼트로 html 태그를 의미한다.

  `document.documentElement` 도 html을 가리키기 때문에 결국 여기서 root를 가리키는 것이다.

```jsx
const inputs = document.querySelectorAll(".controls input");

function handleUpdate() {
  const suffix = this.dataset.sizing || "";
  document.documentElement.style.setProperty(
    `--${this.name}`,
    this.value + suffix
  );
}

inputs.forEach((input) => input.addEventListener("change", handleUpdate));
inputs.forEach((input) => input.addEventListener("mousemove", handleUpdate));
```

## 로컬 이미지 업로드 기능

예전에 [그림판](https://github.com/2dowon/NomadCoders_paintJS) 만들었을 때 이 기능을 넣었던 것이 생각나서 이번에도 추가해봤다.

> HTML

```html
<div class="upload">
  <label for="jsUpload">
    Image
    <i class="far fa-image"></i>
  </label>
  <input type="file" accept="image/*" id="jsUpload" />
</div>
```

> JS

```jsx
const uploadBtn = document.querySelector("#jsUpload");

function readInputFile(event) {
  const file = event.target.files;
  const reader = new FileReader();
  reader.onload = function (e) {
    const newImg = new Image();
    newImg.onload = function () {
      img.src = newImg.src;
      img.style.height = "70vh";
    };
    newImg.src = e.target.result;
  };
  reader.readAsDataURL(file[0]);
}
uploadBtn.addEventListener("change", readInputFile);
```

# 최종 코드

> index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Scoped CSS Variables and JS</title>
    <link
      rel="stylesheet"
      href="https://use.fontawesome.com/releases/v5.11.2/css/all.css"
    />
    <link rel="stylesheet" href="../css/03.css" />
  </head>
  <body>
    <h2>Update CSS Variables with <span class="hl">JS</span></h2>

    <div class="controls">
      <div class="upload">
        <label for="jsUpload">
          Image
          <i class="far fa-image"></i>
        </label>
        <input type="file" accept="image/*" id="jsUpload" />
      </div>
      <label for="spacing">Spacing:</label>
      <input
        id="spacing"
        type="range"
        name="spacing"
        min="10"
        max="200"
        value="10"
        data-sizing="px"
      />

      <label for="blur">Blur:</label>
      <input
        id="blur"
        type="range"
        name="blur"
        min="0"
        max="25"
        value="10"
        data-sizing="px"
      />

      <label for="base">Base Color</label>
      <input id="base" type="color" name="base" value="#ffc600" />
    </div>

    <img src="https://source.unsplash.com/random/800x500" />

    <script>
      const inputs = document.querySelectorAll(".controls input");
      const uploadBtn = document.querySelector("#jsUpload");

      function handleUpdate() {
        const suffix = this.dataset.sizing || "";
        document.documentElement.style.setProperty(
          `--${this.name}`,
          this.value + suffix
        );
      }

      function readInputFile(event) {
        const file = event.target.files;
        const reader = new FileReader();
        reader.onload = function (e) {
          const newImg = new Image();
          newImg.onload = function () {
            img.src = newImg.src;
            img.style.height = "70vh";
          };
          newImg.src = e.target.result;
        };
        reader.readAsDataURL(file[0]);
      }

      inputs.forEach((input) => input.addEventListener("change", handleUpdate));
      inputs.forEach((input) =>
        input.addEventListener("mousemove", handleUpdate)
      );
      uploadBtn.addEventListener("change", readInputFile);
    </script>
  </body>
</html>
```

> style.css

```css
:root {
  --base: #ffc600;
  --spacing: 10px;
  --blur: 10px;
}

img {
  padding: var(--spacing);
  background: var(--base);
  filter: blur(var(--blur));
}

.hl {
  color: var(--base);
}

body {
  text-align: center;
  background: rgb(29, 29, 31);
  color: white;
  font-family: "helvetica neue", sans-serif;
  font-weight: 100;
  font-size: 40px;
}

.controls {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 50px;
}

label {
  margin-right: 10px;
}

input {
  width: 100px;
  margin-right: 30px;
}

#jsUpload {
  display: none;
}
.upload i {
  font-size: 40px;
  margin-right: 20px;
}
```

</br>

# Ref.

- [Woah! CSS Variables?! - #JavaScript30 3/30](https://www.youtube.com/watch?v=AHLNzv13c2I&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=3)

- [2dowon/JavaScript30](https://2dowon.github.io/JavaScript30/)

- [CSS Variables Project](https://2dowon.github.io/JavaScript30/html/03.html)
