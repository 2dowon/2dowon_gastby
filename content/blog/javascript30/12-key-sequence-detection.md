---
title: JavaScript30 - 12. Key Sequence Detection
date: 2021-04-01 21:04:65
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

<img src="./image/keyDetection.gif"  width="800"/>

[Key Sequence Detection](https://2dowon.github.io/JavaScript30/html/12.html) 프로젝트는 secretCode를 찾는 것이다. secretCode를 찾아서 입력하게 되면 화면에 유니콘이 생긴다!

- 키보드 입력받기
- 모든 키보드의 입력은 pressed 배열에 넣는데, pressed 배열의 길이는 secretCode의 길이로 유지하도록 한다. 그래야 정확한 secretCode를 입력했을 때만 유니콘을 생성할 수 있다.

## 키보드 입력받기

### ✍️ My Code

- 처음에는 문제가 키보드의 입력을 받아서 받은 입력으로 배열을 만드는 것이라 생각했다.
- 그래서 작성한 코드가 키보드의 입력된 코드값을 받는데, 입력 받은 코드값은 아스키 코드이므로 이를 다시 문자로 전환해서 keys 배열에 저장했다.
- 키보드의 입력을 받기 위해서는 keydown이나 keyup 이벤트를 등록하면 된다.
- `e.keyCode` ⇒ keydown이나 keyup 이벤트의 keyCode를 출력하면 내가 현재 입력한 키의 코드 값을 알 수 있다. (아스키 코드)
- `String.fromCharCode(e.keyCode)` ⇒ 아스키 코드를 문자로 전환할 수 있다.
- 방향키의 경우, 등록된 코드 값과 이에 해당하는 아스키 코드값이 달라서 다른 결과가 나와 조건문으로 처리했다.

```jsx
const keys = [];

function getKeys(e) {
  // console.log(e.keyCode);

  if (e.keyCode == 37) {
    // left
    console.log("ArrowLeft");
    keys.push("ArrowLeft");
  } else if (e.keyCode == 38) {
    // up
    console.log("ArrowUp");
    keys.push("ArrowUp");
  } else if (e.keyCode == 39) {
    // right
    console.log("ArrowRight");
    keys.push("ArrowRight");
  } else if (e.keyCode == 40) {
    // down
    console.log("ArrowDown");
    keys.push("ArrowDown");
  }

  const key = String.fromCharCode(e.keyCode);
  keys.push(key);
  console.log(key);
  console.log(keys);
}

window.addEventListener("keydown", getKeys);
```

### 👍 Wes Bos Code

- 나는 키보드의 입력값을 알기 위해 keyCode를 이용했는데, 그러다보니 소문자, 대문자, 한글의 구분없이 입력을 받는 문제가 존재했다. 방향키의 경우에도 방향키 대신 특수문자가 출력되는 문제가 있었다.
- 하지만 keyCode 대신 `key`를 이용할 경우, 입력된 키의 값을 정확히 알 수 있다.

```jsx
const pressed = [];

window.addEventListener("keyup", (e) => {
  console.log(e.key);
  pressed.push(e.key);
  console.log(pressed);
});
```

## secretCode를 입력하면 유니콘 생성하기

- secretCode를 입력하면 유니콘을 생성하는 것이 이 프로젝트의 최종 목표이다.
- 유니콘의 생성은 [cornify의 JS 파일](https://www.cornify.com/js/cornify.js)에서 `cornify_add()` 함수를 이용한다.
- 모든 키보드의 입력은 pressed 배열에 넣는데, pressed 배열의 길이는 secretCode의 길이로 유지하도록 한다. 그래야 정확한 secretCode를 입력했을 때만 유니콘을 생성할 수 있다.

```jsx
const pressed = [];
const secretCode = "wesbos";

window.addEventListener("keyup", (e) => {
  console.log(e.key);
  pressed.push(e.key);
  pressed.splice(-secretCode.length - 1, pressed.length - secretCode.length);
  if (pressed.join("").includes(secretCode)) {
    console.log("DING DING!");
    cornify_add();
  }
  console.log(pressed);
});
```

### pressed 배열의 길이는 항상 secretCode의 길이로 유지하기

- secretCode의 길이만큼은 pressed 배열에 입력 값을 저장하다가 pressed 배열의 길이가 secretCode의 길이 이상이 되면 가장 오래된 입력인 맨 처음 입력 값을 제거해줌으로써 항상 길이를 secretCode의 길이로 유지한다
- `splice(제거하기 시작할 인덱스, 삭제하고 싶은 개수)`
- 인덱스가 마이너스라면 뒤에서부터 센다는 뜻이다. 예를 들어, 인덱스가 -1 이라면 배열의 맨 뒤 요소를 의미한다. 따라서 `-secretCode.length - 1` 값이 -7이라면 뒤에서부터 7번째 요소이다. 이는 pressed 배열의 길이가 7일 때 맨 앞 요소를 뜻한다.

  🤔 근데 생각해보니까 맨 앞 요소를 삭제하는 것이기 때문에 인덱스가 0인 요소를 삭제해도 된다.

- 처음에는 pressed 배열의 길이가 secretCode의 길이보다 작기 때문에 `pressed.length - secretCode.length` 값이 음수거나 0이여서 아무것도 삭제되지 않는다. 그러다가 pressed 배열의 길이가 secretCode의 길이보다 많아지는 순간, 이 값은 1이 된다. 즉, 1개를 삭제하게 될 것이라는 뜻이다.
- 결국 아래 코드는 pressed 배열의 길이가 secretCode의 길이보다 많아지는 순간마다 pressed 배열의 가장 오래된 요소인 맨 앞 요소를 1개 삭제한다는 뜻이다. ⇒ 많아지는 순간마다 삭제하기 때문에 2개 이상 삭제할 일은 없다.

```jsx
pressed.splice(-secretCode.length - 1, pressed.length - secretCode.length);
// pressed.splice(0, pressed.length - secretCode.length);
```

### pressed 배열 안에 secretCode가 있다면 유니콘 생성하기

- pressed 배열 안의 모든 입력 값은 한 단어로 합친 다음 이 안에 secretCode가 있는지 확인하다. 근데 사실 pressed 배열의 길이는 secretCode의 길이와 똑같기 때문에 결국 pressed 배열을 한 단어로 합친 것이 secretCode와 똑같은지를 묻는 것과 동일하다.
- 그래서 똑같다면 `cornify_add()` 함수를 이용해 유니콘을 생성해주면 된다.

```jsx
if (pressed.join("").includes(secretCode)) {
  console.log("DING DING!");
  cornify_add();
}
```

# 최종 코드

> index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Key Detection</title>
    <script
      type="text/javascript"
      src="https://www.cornify.com/js/cornify.js"
    ></script>
  </head>
  <body>
    <script>
      const pressed = [];
      const secretCode = "wesbos";

      window.addEventListener("keyup", (e) => {
        console.log(e.key);
        pressed.push(e.key);
        pressed.splice(0, pressed.length - secretCode.length);
        // pressed.splice(-secretCode.length - 1, pressed.length - secretCode.length);
        if (pressed.join("").includes(secretCode)) {
          console.log("DING DING!");
          cornify_add();
        }
        console.log(pressed);
      });
    </script>
  </body>
</html>
```

</br>

# Ref.

- [JavaScript KONAMI CODE! #JavaScript30 12/30](https://www.youtube.com/watch?v=_A5eVOIqGLU&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=12)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)

- [[Javascript] 자바스크립트 키보드 관련 이벤트(onkeydown, onkeyup, onkeypress)](https://sas-study.tistory.com/143)
