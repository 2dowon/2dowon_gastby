---
title: JavaScrip30 - 7. Array Cardio Day 2
date: 2021-03-26 16:03:41
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

[Array Cardio Day 2 프로젝트](https://2dowon.github.io/JavaScript30/html/07.html)는 **Array의 다양한 메서드 연습하기!** 이다. 오늘 연습할 Array의 메서드는 다음과 같다.

- [some](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/some)
- [every](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/every)
- [find](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/find)
- [findIndex](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)
- [splice](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)

> 사용할 데이터

```jsx
const people = [
  { name: "Wes", year: 1988 },
  { name: "Kait", year: 1986 },
  { name: "Irv", year: 1970 },
  { name: "Lux", year: 2015 },
];

const comments = [
  { text: "Love this!", id: 523423 },
  { text: "Super good", id: 823423 },
  { text: "You are the best", id: 2039842 },
  { text: "Ramen is my fav food ever", id: 123523 },
  { text: "Nice Nice Nice!", id: 542328 },
];
```

## some

- 배열의 요소 중에서 callback함수의 값이 true가 있는지 없는지를 체크해서 true/false로 알려준다
- 요소 중 하나라도 조건에 맞으면 true가 리턴된다
- 빈 배열에서 호출하면 무조건 false를 리턴한다
- `some` 메서드의 기본 문법

  ```jsx
  arr.some(callback[, thisArg])
  ```

### is at least one person 19 or older?

```jsx
const currentYear = new Date().getFullYear();
const isAdult = people.some((person) => currentYear - person.year >= 19);

console.log({ isAdult });
// {isAdult: true}
```

## every

- 배열에 들어있는 모든 요소들이 callback 함수의 조건을 충족해야지만 true가 리턴된다
- `every` 메서드의 기본 문법

  ```jsx
  arr.every(callback[, thisArg])
  ```

### is everyone 19 or older?

```jsx
const currentYear = new Date().getFullYear();
const allAdults = people.every((person) => currentYear - person.year >= 19);

console.log({ allAdults });
// {allAdults: false}
```

## find

- 주어진 조건을 만족하는 **첫 번째** 요소의 값을 리턴한다
- 해당 조건을 만족하는 값이 없다면 `undefined`를 리턴한다
- `find` 메서드의 기본 문법

  ```jsx
  arr.find(callback[, thisArg])
  ```

### find the comment with the ID of 823423

```jsx
const findComment = comments.find((comment) => comment.id === 823423);

console.log(findComment);
// {text: "Super good", id: 823423}
```

## findIndex

- 주어진 조건을 만족하는 첫 번째 요소의 인덱스를 리턴한다
- 해당 조건을 만족하는 요소가 없다면 `-1`을 리턴한다
- `findIndex` 메서드의 기본 문법

  ```jsx
  arr.findIndex(callback(element[, index[, array]])[, thisArg])
  ```

### Find the comment with this ID

```jsx
const index = comments.findIndex((comment) => comment.id === 823423);
console.log(index); // 1
```

## splice

- remove an item by index position ⇒ 지정된 위치의 item을 제거하는 것
- `splice` 함수의 기본 문법

  ```jsx
  array.splice(start[, deleteCount[, item1[, item2[, ...]]]])
  ```

  - start : 지우기 시작하고 싶은 index의 위치
  - deleteCount : 지우고 싶은 개수 (만약 적지 않으면 끝까지 다 지워버린다)
  - item~ : splice를 한 다음 그 위치에 item을 추가로 넣을 수도 있다

- 🤔 delete와 splice의 차이

  - `delete` : 해당 요소를 삭제해도 기존의 key들은 유지된다.

  - `splice` : delete와 다르게 해당 요소를 삭제하면 기존의 key들이 알아서 하나씩 줄어들어 유지된다

  - ex. 1번 인덱스를 삭제할 때, delete를 이용하면 남은 key가 0, 2, 3, 4이다. 하지만 splice를 이용하면 남은 key는 0, 1, 2, 3이 된다.

### delete the comment with the ID of 823423

findIndex 메서드로 index를 알아낸 후, splice로 삭제

> comments 배열에서 바로 splice 메서드를 사용하면 comment 배열이 수정된다

```jsx
comments.splice(index, 1);
```

> 만약 기존 배열은 유지한채로 원하는 요소를 삭제한 새로운 배열을 만들고 싶다면

```jsx
const newComments = [...comments.slice(0, index), ...comments.slice(index + 1)];
```

</br>

# Ref.

- [.some(), .every(), .find() and [...SPREADS] - Array Cardio Day 2 - #JavaScript30 7/30](https://www.youtube.com/watch?v=QNmRfyNg1lw&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=7)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)
