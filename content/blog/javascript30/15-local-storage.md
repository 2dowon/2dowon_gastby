---
title: JavaScript30 - 15. LocalStorage and Event Delegation
date: 2021-04-06 10:04:88
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

[LocalStorage](https://2dowon.github.io/JavaScript30/html/15.html) 프로젝트의 핵심은 다음과 같다.

- to do list 만들기
- LocalStorage에 저장하기

## to do list 만들기

### ✍️ My Code

- to do list는 이전에도 몇 번 만든 적이 있어서 크게 어렵지 않게 만들 수 있었다.
- 하지만 처음 시작할 때 submit이 제출되는 것을 막는 것을 깜박해 console.log를 확인할 수 없어서 헤맸는데, 이 경우 `e.preventDefault();` 를 이용하면 이벤트가 발생하는 것을 막아주기 때문에 결국 form에 적용하게 되면 submit되는 것을 막을 수 있다.
- addItem 함수를 통해 input에 입력된 값을 가져와서 이 값을 이용해 todo를 만들면 된다. 그리고 그 후에는 input에 입력된 값을 초기화해준다.

```jsx
const addItems = document.querySelector(".add-items");
const inputItem = document.querySelector(".input-item");
const itemsList = document.querySelector(".plates");

const items = [];

function addItem(e) {
  e.preventDefault();
  const currentValue = inputItem.value;
  creatElement(currentValue);
  inputItem.value = "";
}

function creatElement(text) {
  const li = document.createElement("li");
  const newId = items.length;

  const input = document.createElement("input");
  input.setAttribute("type", "checkbox");
  input.setAttribute("data-index", newId);
  input.setAttribute("id", `item${newId}`);

  const label = document.createElement("label");
  label.setAttribute("for", `item${newId}`);
  label.innerText = text;

  li.append(input);
  li.append(label);
  itemsList.append(li);

  const itemObj = {
    text,
    id: `item${newId}`,
  };
  items.push(itemObj);
}

addItems.addEventListener("submit", addItem);
```

### 👍 Wes Bos Code

- 나는 creatElement 함수에서 하나씩 요소들을 생성하면서 만들었는데, 템플릿 리터럴을 이용해서 innerHTML을 통해 요소를 만들게 되면 훨씬 가독성이 좋고, 쉽게 만들 수 있다.

  그리고 map 함수를 이용해서 id 값을 만들 수 있다.

- 나는 위에서 토글 값의 함수를 따로 만들지 않았는데, 이는 나중에 local storage에 저장하기 위해서는 꼭 필요하다. 체크가 되었는지의 여부도 저장해야 되기 때문에. 나는 이 함수를 만들지 않았기 때문에 나중에 local storage에 저장할 때에도 새로고침하면 체크 여부는 저장되지 않았었다.
- toggleDone 함수는 itemsList에 이벤트를 등록함으로써 **이벤트 위임(Event Delegation)**을 할 수 있다. 그렇기 때문에 클릭한 요소가 input이 아닐 경우에는 early return을 통해 빨리 함수를 종료해줘야 한다.

```jsx
const addItems = document.querySelector(".add-items");
const itemsList = document.querySelector(".plates");
const items = [];

function addItem(e) {
  e.preventDefault();
  const text = this.querySelector("[name=item]").value;
  const item = {
    text,
    done: false,
  };

  items.push(item);
  creatElement(items, itemsList);
  this.reset();
}

function creatElement(plates = [], platesList) {
  platesList.innerHTML = plates
    .map((plate, i) => {
      return `
        <li>
          <input type="checkbox" data-index=${i} id="item${i}" ${
        plate.done ? "checked" : ""
      } />
          <label for="item${i}">${plate.text}</label>
        </li>
      `;
    })
    .join("");
}

function toggleDone(e) {
  if (!e.target.matches("input")) return; // skip this unless it's an input
  const el = e.target;
  const index = el.dataset.index;
  items[index].done = !items[index].done;
  creatElement(items, itemsList);
}

addItems.addEventListener("submit", addItem);
itemsList.addEventListener("click", toggleDone);

populateList(items, itemsList);
```

## LocalStorage에 저장하기

- 새로고침했을 때도 정보가 초기화되지 않기 위해서는 [local storage](https://developer.mozilla.org/ko/docs/Web/API/Window/localStorage) 를 이용해야 한다.
- [local storage](https://developer.mozilla.org/ko/docs/Web/API/Window/localStorage)
  - `localStorage.setItem()` localStorage에 저장할 때
  - `localStorage.getItem()` localStorage에 저장한 항목을 읽을 때
  - `localStorage.removeItem()` localStorage에 저장한 항목을 삭제할 때
  - `localStorage.clear()` localStorage의 전체를 초기화할 때

> 아래 saveItems 함수는 아이템을 새로 만들 때와 토글 버튼을 클릭해 토글의 상태에 변화가 생겼을 때마다 실행줘야 한다.

```jsx
const ITEMS_LS = "items";
const items = JSON.parse(localStorage.getItem(ITEMS_LS)) || [];

function saveItems() {
  localStorage.setItem(ITEMS_LS, JSON.stringify(items));
}
```

- 위에서 to do list를 만들 때 생성된 to do를 items 배열에 push했다.
- `saveItems` 함수 ⇒ items는 object로 구성된 배열인데, 이를 `JSON.stringify()` 를 통해 string으로 만들어 `setItem` 함수를 이용해 localStorage에 저장한다.
- localStorage에 저장한 항목들은 `getItem` 을 이용해 읽을 수 있고, 읽어온 항목들을 `JSON.parse()` 를 이용해 다시 object로 변환해 items 배열에 저장한다. 단, localStorage에 저장된 항목들이 있는 경우에만 읽어오면 되기 때문에, 만약 localStorage에 저장된 값이 없다면 items는 빈 배열이 된다.

# 최종 코드

> index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>LocalStorage</title>
    <link rel="stylesheet" href="../css/15.css" />
  </head>
  <body>
    <svg
      xmlns="http://www.w3.org/2000/svg"
      xmlns:xlink="http://www.w3.org/1999/xlink"
      version="1.1"
      x="0px"
      y="0px"
      viewBox="0 0 512 512"
      enable-background="new 0 0 512 512"
      xml:space="preserve"
    >
      <g>
        <path
          d="M495.9,425.3H16.1c-5.2,0-10.1,2.9-12.5,7.6c-2.4,4.7-2.1,10.3,0.9,14.6l39,56.4c2.6,3.8,7,6.1,11.6,6.1h401.7   c4.6,0,9-2.3,11.6-6.1l39-56.4c3-4.3,3.3-9.9,0.9-14.6C506,428.2,501.1,425.3,495.9,425.3z M449.4,481.8H62.6L43,453.6H469   L449.4,481.8z"
        />
        <path
          d="M158.3,122c7.8,0,14.1-6.3,14.1-14.1V43.4c0-7.8-6.3-14.1-14.1-14.1c-7.8,0-14.1,6.3-14.1,14.1v64.5   C144.2,115.7,150.5,122,158.3,122z"
        />
        <path
          d="M245.1,94.7c7.8,0,14.1-6.3,14.1-14.1V16.1c0-7.8-6.3-14.1-14.1-14.1C237.3,2,231,8.3,231,16.1v64.5   C231,88.4,237.3,94.7,245.1,94.7z"
        />
        <path
          d="M331.9,122c7.8,0,14.1-6.3,14.1-14.1V43.4c0-7.8-6.3-14.1-14.1-14.1s-14.1,6.3-14.1,14.1v64.5   C317.8,115.7,324.1,122,331.9,122z"
        />
        <path
          d="M9.6,385.2c5.3,2.8,11.8,1.9,16.2-2.2l50.6-47.7c56.7,46.5,126.6,71.9,198.3,71.9c0,0,0,0,0,0   c87.5,0,169.7-36.6,231.4-103.2c5-5.4,5-13.8,0-19.2c-61.8-66.5-144-103.2-231.4-103.2c-72,0-142.2,25.6-199,72.5l-50-47.1   c-4.4-4.1-10.9-5-16.2-2.2c-5.3,2.8-8.3,8.7-7.4,14.6l11.6,75L2.2,370.6C1.3,376.5,4.2,382.4,9.6,385.2z M380.9,230.8   c34.9,14.3,67.2,35.7,95.3,63.6c-10.1,10-20.8,19.2-31.9,27.5c-22.4-3.3-29.6-8.8-30.7-9.7c-4-5.7-11.8-7.7-18.1-4.4   c-6.9,3.6-9.5,12.2-5.9,19.1c1.9,3.5,7.3,10.3,22.4,16c-10.1,5.7-20.5,10.7-31.1,15.1C352.4,320.2,352.4,268.6,380.9,230.8z    M36.3,255.6l29.4,27.7c5.3,5,13.6,5.1,19.1,0.3c53.2-47.6,120.7-73.7,190-73.7c26.9,0,53.2,3.9,78.5,11.3   c-29.3,44.6-29.3,102,0,146.6c-25.3,7.4-51.6,11.3-78.5,11.3c-69,0-136.3-26-189.4-73.2c-2.7-2.4-13.4-6.3-19.1,0.3l-30.1,28.3   l5.7-40C42.2,293,36.3,255.6,36.3,255.6z"
        />
        <circle cx="398.8" cy="273.8" r="14.1" />
      </g>
    </svg>

    <div class="wrapper">
      <h2>LOCAL TAPAS</h2>
      <p></p>
      <ul class="plates">
        <!-- <li>Loading Tapas...</li> -->
      </ul>
      <form class="add-items">
        <input
          type="text"
          name="item"
          placeholder="Item Name"
          class="input-item"
          required
        />
        <input type="submit" value="+ Add Item" />
      </form>
    </div>

    <script src="../js/15.js"></script>
  </body>
</html>
```

> styles.css

```css
html {
  box-sizing: border-box;
  background: url("../img/oh-la-la.jpeg") center no-repeat;
  background-size: cover;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  font-family: Futura, "Trebuchet MS", Arial, sans-serif;
}

*,
*:before,
*:after {
  box-sizing: inherit;
}

svg {
  fill: white;
  background: rgba(0, 0, 0, 0.1);
  padding: 20px;
  border-radius: 50%;
  width: 200px;
  margin-bottom: 50px;
}

.wrapper {
  padding: 20px;
  max-width: 350px;
  background: rgba(255, 255, 255, 0.95);
  box-shadow: 0 0 0 10px rgba(0, 0, 0, 0.1);
}

h2 {
  text-align: center;
  margin: 0;
  font-weight: 200;
}

.plates {
  margin: 0;
  padding: 0;
  text-align: left;
  list-style: none;
}

.plates li {
  border-bottom: 1px solid rgba(0, 0, 0, 0.2);
  padding: 10px 0;
  font-weight: 100;
  display: flex;
}

.plates label {
  flex: 1;
  cursor: pointer;
}

.plates input {
  display: none;
}

.plates input + label:before {
  content: "⬜️";
  margin-right: 10px;
}

.plates input:checked + label:before {
  content: "🌮";
}

.add-items {
  margin-top: 20px;
}

.add-items input {
  padding: 10px;
  outline: 0;
  border: 1px solid rgba(0, 0, 0, 0.1);
}
```

> JS

```jsx
const addItems = document.querySelector(".add-items");
const inputItem = document.querySelector(".input-item");
const itemsList = document.querySelector(".plates");

const ITEMS_LS = "items";
const items = JSON.parse(localStorage.getItem(ITEMS_LS)) || [];

function addItem(e) {
  e.preventDefault();
  const text = inputItem.value;

  const item = {
    text,
    done: false,
  };

  items.push(item);
  creatElement(items, itemsList);
  saveItems();

  inputItem.value = "";
}

function saveItems() {
  localStorage.setItem(ITEMS_LS, JSON.stringify(items));
}

function creatElement(plates = [], platesList) {
  platesList.innerHTML = plates
    .map((plate, i) => {
      return `
      <li>
        <input type="checkbox" data-index=${i} id="item${i}" ${
        plate.done ? "checked" : ""
      } />
        <label for="item${i}">${plate.text}</label>
      </li>
    `;
    })
    .join("");
}

function toggleDone(e) {
  if (!e.target.matches("input")) return;
  // skip this unless it's an input
  const el = e.target;
  const index = el.dataset.index;
  console.log(items);
  console.log(items[index]);
  items[index].done = !items[index].done;
  saveItems();
  creatElement(items, itemsList);
}

function init() {
  creatElement(items, itemsList);
  addItems.addEventListener("submit", addItem);
  itemsList.addEventListener("click", toggleDone);
}
init();
```

<br>

# Ref.

- [How LocalStorage and Event Delegation work. #JavaScript30 15/30](https://www.youtube.com/watch?v=YL1F4dCUlLc&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=15)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)
