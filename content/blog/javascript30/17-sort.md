---
title: JavaScript30 - 17. Sort Without Articles
date: 2021-04-09 18:04:11
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

[Sort Without Articles](https://2dowon.github.io/JavaScript30/html/17.html) 프로젝트의 핵심은 다음과 같다.

- bands 이름에서 Articles(관사) 제거하고, 정렬하기
- HTML에 표현하기

> bands 데이터

```jsx
const bands = [
  "The Plot in You",
  "The Devil Wears Prada",
  "Pierce the Veil",
  "Norma Jean",
  "The Bled",
  "Say Anything",
  "The Midway State",
  "We Came as Romans",
  "Counterparts",
  "Oh, Sleeper",
  "A Skylit Drive",
  "Anywhere But Here",
  "An Old Dog",
];
```

## bands 이름에서 Articles(관사) 제거하고, 정렬하기

### ✍️ My Code

- 관사 a, an, the를 제거하기 위해 `replace` 를 이용했다. replace 옵션 중 대소문자에 구분없이 제거하기 위해서 정규식의 `gi` 옵션을 사용했고, 줄의 시작과 일치하는 관사를 제거할 것이기 때문에 `^` 패턴을 사용했다.
- ⚠️ 이 때 주의해야 하는 점이 a, an, the 뒤에 공백을 한 칸씩 둬야 한다. 그래야 단어가 제거되고, 그러지 않을 경우 단어 안에 있는 a, an, the가 삭제될 수 있다. (빈칸이 없을 때 ex. Anywhere → nywhere)
- 정렬의 경우, a~z 순으로 정렬하는 것이므로 `sort()` 를 이용하면 정렬할 수 있다.
- 하지만, 내 코드의 경우 나중에 HTML에 추가할 때 관사가 제거된 밴드명이 추가된다는 문제점이 생긴다.

```jsx
const replaceArticles = bands.map((band) =>
  band.replace(/^(a |an |the )/gi, "")
);
replaceArticles.sort();
```

### 👍 Wes Bos Code

- 관사 제거하는 것을 함수로 만들어서 리턴하여, 리턴 값으로 정렬을 한다면 HTML에 추가할 때 관사가 제거된 밴드명이 추가되는 문제점을 해결할 수 있다
- a~z 순으로 정렬하는 것은 오름차순 정렬이므로 a보다 b가 클 경우 1(true)을 리턴하면 된다.

  `const sortedBands = bands.sort((a, b) => strip(a) - strip(b));` 오름차순 정렬이므로 이 식으로도 똑같은 결과가 나와야할 것 같은데, 이렇게는 정렬이 안된다. 🤔 왜 그러지

- `trim` 을 이용하면 양 옆 공백을 제거할 수 있다. 맨 앞에 있는 관사를 제거할 경우, 맨 앞에 공백이 생기기 때문에 trim을 이용해 제거해준다.

```jsx
function strip(bandName) {
  return bandName.replace(/^(a |the |an )/i, "").trim();
}

const sortedBands = bands.sort((a, b) => (strip(a) > strip(b) ? 1 : -1));
```

## HTML에 표현하기

### ✍️ My Code

```jsx
const container = document.querySelector("#bands");

function createElement(band) {
  const li = document.createElement("li");
  li.innerText = band;
  container.append(li);
}

sortedBands.forEach((band) => createElement(band));
```

### 👍 Wes Bos Code

```jsx
document.querySelector("#bands").innerHTML = sortedBands
  .map((band) => `<li>${band}</li>`)
  .join("");
```

# 최종 코드

> index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Sort Without Articles</title>
    <link rel="stylesheet" href="../css/17.css" />
  </head>
  <body>
    <ul id="bands"></ul>

    <script>
      const container = document.querySelector("#bands");

      const bands = [
        "The Plot in You",
        "The Devil Wears Prada",
        "Pierce the Veil",
        "Norma Jean",
        "The Bled",
        "Say Anything",
        "The Midway State",
        "We Came as Romans",
        "Counterparts",
        "Oh, Sleeper",
        "A Skylit Drive",
        "Anywhere But Here",
        "An Old Dog",
      ];

      function strip(bandName) {
        return bandName.replace(/^(a |the |an )/i, "").trim();
      }

      const sortedBands = bands.sort((a, b) => (strip(a) > strip(b) ? 1 : -1));

      function createElement(band) {
        const li = document.createElement("li");
        li.innerText = band;
        container.append(li);
      }

      sortedBands.forEach((band) => createElement(band));
    </script>
  </body>
</html>
```

> style.css

```css
body {
  margin: 0;
  font-family: sans-serif;
  background: url("https://source.unsplash.com/nDqA4d5NL0k/2000x2000");
  background-size: cover;
  display: flex;
  align-items: center;
  min-height: 100vh;
}

#bands {
  list-style: inside square;
  font-size: 20px;
  background: white;
  width: 500px;
  margin: 0 auto;
  margin-top: 100px;
  margin-bottom: 100px;
  padding: 0;
  box-shadow: 0 0 0 20px rgba(0, 0, 0, 0.05);
}

#bands li {
  border-bottom: 1px solid #efefef;
  padding: 20px;
}

#bands li:last-child {
  border-bottom: 0;
}

a {
  color: #ffc600;
  text-decoration: none;
}
```

</br>

# Ref.

- [JavaScript Practice: Sorting Band Names without articles - #JavaScript30​ 17/30](https://www.youtube.com/watch?v=PEEo-2mRQ7A&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=17)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)

- [[JavaScript] 자바스크립트에서 replace 를 replaceAll 처럼 사용하여 모든 문자 바꾸기 (feat.정규식)](https://elena90.tistory.com/entry/JavaScript-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%97%90%EC%84%9C-replace-%EB%A5%BC-replaceAll-%EC%B2%98%EB%9F%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%EB%AA%A8%EB%93%A0-%EB%AC%B8%EC%9E%90-%EB%B0%94%EA%BE%B8%EA%B8%B0-feat%EC%A0%95%EA%B7%9C%EC%8B%9D)

- [정규표현식, 이렇게 시작하자!](https://heropy.blog/2018/10/28/regexp/)
