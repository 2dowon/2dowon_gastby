---
title: JavaScript30 - 20. Speech Detection
date: 2021-04-17 10:04:08
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

[Speech Detection](https://2dowon.github.io/JavaScript30/html/20.html) 프로젝트의 핵심은 다음과 같다.

- 브라우저에서 음성 인식하기
- 인식한 음성을 웹에 표시하기
- 특정 단어 인식하기

## 브라우저에서 음성 인식하기

### SpeechRecognition

- 브라우저에서 음성을 인식하기 위해서 Web API 중 하나인 [SpeechRecognition](https://developer.mozilla.org/en-US/docs/Web/API/SpeechRecognition) 을 사용하면 된다.
- `SpeechRecognition` 는 Chrome과 Edge 브라우저에서만 사용할 수 있다.
- `interimResults` 는 임시 결과의 반환 여부를 제어할 수 있는 속성이다. 이 값을 true로 설정하면 실시간으로 인식된 값을 모두 확인할 수 있고, false로 설정하면 최종적으로 인식된 문장만 확인할 수 있다.
- `lang` 은 인식하고자 하는 음성의 언어를 선택하면 된다.

```jsx
window.SpeechRecognition =
  window.SpeechRecognition || window.webkitSpeechRecognition;

const recognition = new SpeechRecognition();
recognition.interimResults = true;
recognition.lang = "en-US";

recognition.addEventListener("result", (e) => {
  const transcript = Array.from(e.results)
    .map((result) => result[0])
    .map((result) => result.transcript)
    .join("");
});

recognition.start();
```

## 인식한 음성을 웹에 표시하기

- 인식한 음성을 웹에 표시하기 위해서 `p` 태그를 생성하고, 이를 `words` div 안에 넣는다.
- 잠시 멈췄다가 말했을 경우 새로운 문단으로 생성해주기 위해서 `isFinal` 속성 값이 true라면 새로운 `p` 태그를 생성할 수 있도록 한다.
- 그리고 잠시 멈췄다가 말했을 때도 계속 인식할 수 있도록 하기 위해서 `recognition` 이 `end` 라면 다시 시작하도록 만든다.

```jsx
let p = document.createElement("p");
const words = document.querySelector(".words");
words.appendChild(p);

recognition.addEventListener("result", (e) => {
  const transcript = Array.from(e.results)
    .map((result) => result[0])
    .map((result) => result.transcript)
    .join("");

  // 잠시 멈췄다가 말했을 경우 새로운 문단으로 생성해주기 위해서
  if (e.results[0].isFinal) {
    p = document.createElement("p");
    words.appendChild(p);
  }
});

// 잠시 멈췄다가 말했을 때도 계속 인식할 수 있도록 하기 위해서
recognition.addEventListener("end", recognition.start);
```

## 특정 단어 인식하기

- 특정 단어를 인식하면 해당 단어를 다른 것으로 대체할 수 있다.
- 예를 들어서 poop, poo, shit, dump와 같은 단어를 인식하면 해당 단어는 그대로 작성되는 것이 아닌 💩 이모지로 대체된다.

```jsx
recognition.addEventListener("result", (e) => {
  const transcript = Array.from(e.results)
    .map((result) => result[0])
    .map((result) => result.transcript)
    .join("");

  const poopScript = transcript.replace(/poop|poo|shit|dump/gi, "💩");
  p.textContent = poopScript;

  if (e.results[0].isFinal) {
    p = document.createElement("p");
    words.appendChild(p);
  }
});
```

# 최종 코드

> index.html

- div에 `contenteditable` 옵션을 주면 div 안에 있는 내용을 편집할 수 있다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Speech Detection</title>
    <link rel="stylesheet" href="../css/20.css" />
  </head>
  <body>
    <div class="words" contenteditable></div>

    <script>
      window.SpeechRecognition =
        window.SpeechRecognition || window.webkitSpeechRecognition;

      const recognition = new SpeechRecognition();
      recognition.interimResults = true;
      recognition.lang = "en-US";

      let p = document.createElement("p");
      const words = document.querySelector(".words");
      words.appendChild(p);

      recognition.addEventListener("result", (e) => {
        const transcript = Array.from(e.results)
          .map((result) => result[0])
          .map((result) => result.transcript)
          .join("");

        const poopScript = transcript.replace(/poop|poo|shit|dump/gi, "💩");
        p.textContent = poopScript;

        if (e.results[0].isFinal) {
          p = document.createElement("p");
          words.appendChild(p);
        }
      });

      recognition.addEventListener("end", recognition.start);

      recognition.start();
    </script>
  </body>
</html>
```

> style.css

```css
html {
  font-size: 10px;
}

body {
  background: #ffc600;
  font-family: "helvetica neue";
  font-weight: 200;
  font-size: 20px;
}

.words {
  max-width: 500px;
  margin: 50px auto;
  background: white;
  border-radius: 5px;
  box-shadow: 10px 10px 0 rgba(0, 0, 0, 0.1);
  padding: 1rem 2rem 1rem 5rem;
  background: -webkit-gradient(
      linear,
      0 0,
      0 100%,
      from(#d9eaf3),
      color-stop(4%, #fff)
    ) 0 4px;
  background-size: 100% 3rem;
  position: relative;
  line-height: 3rem;
}

p {
  margin: 0 0 3rem;
}

.words:before {
  content: "";
  position: absolute;
  width: 4px;
  top: 0;
  left: 30px;
  bottom: 0;
  border: 1px solid;
  border-color: transparent #efe4e4;
}
```

</br>

# Ref.

- [JavaScript Speech Recognition #JavaScript30 20/30](https://www.youtube.com/watch?v=0mJC0A72Fnw&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=20)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)

- [음성을 인식하는 웹 메모장 만들기](https://til-devsong.tistory.com/77?category=775075)

- [[SpeechRecognition] Web/Chrome 음성 인식](https://mizzo-dev.tistory.com/entry/SpeechRecognition-WebChrome-%EC%9D%8C%EC%84%B1-%EC%9D%B8%EC%8B%9D)
