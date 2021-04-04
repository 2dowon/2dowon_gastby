---
title: JavaScript30 - 14. JavaScript References VS Copying
date: 2021-04-04 22:04:80
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

[JavaScript References VS Copying](https://2dowon.github.io/JavaScript30/html/14.html) 프로젝트는 복사(cloning)에 관한 것이다.

- primitive types 원시값 복사와 Object 참조값 복사
- 얕은 복사(shallow-cloning)와 깊은 복사(deep-cloning)

## primitive types ⇒ 원시값 복사

- primitive types = number, string, boolean, null, undefined
- primitive types인 원시형 데이터를 변수에 할당하게 되면 데이터 자체가 변수에 할당된다.
- 따라서 기존 변수를 복사한 후 수정하게 되면 기존 변수와 복사된 변수는 서로에게 영향을 미치지 않는다.

> number 변수 복사

```jsx
let age = 100;
let age2 = age;
console.log(age, age2); // 100 100
age = 200;
console.log(age, age2); // 200 100
```

> string 변수 복사

```jsx
let name = "Wes";
let name2 = name;
console.log(name, name2); // Wes Wes
name = "wesley";
console.log(name, name2); // wesley Wes
```

## Object, Array ⇒ 참조값 복사

- Object, Array를 변수에 할당하면 변수에는 Object, Array가 메모리에 들어있는 주소인, 참조값이 할당된다.
- 따라서 기존 변수나 복사된 변수를 수정하게 되면 둘은 같은 참조값을 가지고 있기 때문에 서로에게 영향을 미치게 된다.

> array 변수 복사

```jsx
const players = ["Wes", "Sarah", "Ryan", "Poppy"];
const team = players;
team[3] = "Lux";

console.log(players);
// ["Wes", "Sarah", "Ryan", "Lux"]
console.log(team);
// ["Wes", "Sarah", "Ryan", "Lux"]
```

## 얕은 복사(shallow-cloning)

- 얕은 복사(shallow-cloning)를 하면 기존 변수의 값을 복사해서 새로운 변수를 만들게 된다.
- 따라서 기존 변수와 복사된 변수는 서로 영향을 미치지 않게 되어 앞에서 발생했던 문제인 서로 같은 참조값을 공유하여 하나의 변수를 수정할 때 다른 변수도 수정되는 문제를 해결할 수 있다.
- 하지만 얕은 복사는 중첩된 구조를 복사할 수 없다는 문제점이 있다.

### 1. slice를 이용한 얕은 복사

```jsx
const team2 = players.slice();
```

### 2. concat을 이용한 얕은 복사

```jsx
const team3 = [].concat(players);
```

### 3. spread syntax를 이용한 얕은 복사 👍

```jsx
const team4 = [...players];
```

### 4. Array.from을 이용한 얕은 복사

```jsx
const team5 = Array.from(players);
```

### 5. Object.assign을 이용한 얕은 복사

```jsx
const cap = Object.assign({}, person);
```

### ✅ 얕은 복사를 한 후 복사한 변수를 수정

> array 변수

```jsx
const players = ["Wes", "Sarah", "Ryan", "Poppy"];
const team4 = [...players];
team4[3] = "heeee hawww";
console.log(players);
// ["Wes", "Sarah", "Ryan", "Poppy"]
console.log(team4);
// ["Wes", "Sarah", "Ryan", "heeee hawww"]

const person = {
  name: "Wes Bos",
  age: 80,
};
const cap = { ...person };
cap.age = 99;
console.log(person);
// {name: "Wes Bos", age: 80}
console.log(cap);
// {name: "Wes Bos", age: 99}
```

> object 변수

```jsx
const person = {
  name: "Wes Bos",
  age: 80,
};

const cap = { ...person };
cap.age = 99;
console.log(person);
// {name: "Wes Bos", age: 80}
console.log(cap);
// {name: "Wes Bos", age: 99}

const cap2 = Object.assign({}, person, { number: 99, age: 12 });
console.log(person);
// {name: "Wes Bos", age: 80}
console.log(cap2);
// {name: "Wes Bos", age: 12, number: 99}
```

## 깊은 복사 (deep-cloning)

- 중첩된 구조를 복사하기 위해서는 깊은 복사(deep-cloning)를 이용해야 한다.
- 깊은 복사를 하는 방법에는 json을 이용한 방법이 있다.

> 얕은 복사를 통해 name, age 값은 복사할 수 있지만, social 값은 복사할 수 없다

```jsx
const wes = {
  name: "Wes",
  age: 100,
  social: {
    twitter: "@wesbos",
    facebook: "wesbos.developer",
  },
};
```

> 얕은 복사 후 중첩 구조의 값을 수정하면 기존 변수의 값도 같이 수정된다.

```jsx
const dev = Object.assign({}, wes);
dev.social.twitter = "@coolman";
console.log(wes.social);
// {twitter: "@coolman", facebook: "wesbos.developer"}
console.log(dev.social);
// {twitter: "@coolman", facebook: "wesbos.developer"}
```

### json을 이용한 깊은 복사

- `JSON.stringify` 를 통해 object의 모든 값을 string으로 변환해서 복사한다.
- 그 다음 `JSON.parse` 를 이용하면 다시 json 형식으로 변환되어 object의 모든 값이 복사되는 것과 같다.

```jsx
const dev2 = JSON.parse(JSON.stringify(wes));
dev2.social.twitter = "@coolman";
console.log(wes.social);
// {twitter: "@wesbos", facebook: "wesbos.developer"}
console.log(dev.social);
// {twitter: "@coolman", facebook: "wesbos.developer"}
```

# 최종 코드

> index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>JS Reference VS Copy</title>
  </head>
  <body>
    <script>
      // start with strings, numbers and booleans
      let age = 100;
      let age2 = age;
      console.log(age, age2);
      age = 200;
      console.log(age, age2);

      let name = "Wes";
      let name2 = name;
      console.log(name, name2);
      name = "wesley";
      console.log(name, name2);

      // Let's say we have an array
      const players = ["Wes", "Sarah", "Ryan", "Poppy"];

      // and we want to make a copy of it.
      const team = players;

      console.log(players, team);
      // You might think we can just do something like this:
      team[3] = "Lux";
      console.log(players, team);

      // however what happens when we update that array?

      // now here is the problem!

      // oh no - we have edited the original array too!

      // Why? It's because that is an array reference, not an array copy. They both point to the same array!

      // So, how do we fix this? We take a copy instead!
      const team2 = players.slice();

      // one way

      // or create a new array and concat the old one in
      const team3 = [].concat(players);

      // or use the new ES6 Spread
      const team4 = [...players];
      team4[3] = "heeee hawww";
      console.log(players, team4);

      players[0] = "won";
      console.log(players, team4);

      const team5 = Array.from(players);

      // now when we update it, the original one isn't changed

      // The same thing goes for objects, let's say we have a person object

      // with Objects
      const person = {
        name: "Wes Bos",
        age: 80,
      };

      // and think we make a copy:
      // const captain = person;
      // captain.number = 99;
      // console.log(person, captain);

      // how do we take a copy instead?
      const cap2 = Object.assign({}, person, { number: 99, age: 12 });
      console.log(person);
      console.log(cap2);

      // We will hopefully soon see the object ...spread
      const cap3 = { ...person };
      cap3.age = 99;
      console.log(person);
      console.log(cap3);

      // Things to note - this is only 1 level deep - both for Arrays and Objects. lodash has a cloneDeep method, but you should think twice before using it.

      const wes = {
        name: "Wes",
        age: 100,
        social: {
          twitter: "@wesbos",
          facebook: "wesbos.developer",
        },
      };

      //console.clear();
      console.log(wes);

      const dev = Object.assign({}, wes);
      console.log(dev);
      // dev.social.twitter = "@coolman";
      // console.log(wes.social);
      // console.log(dev.social);

      const dev2 = JSON.parse(JSON.stringify(wes));
      console.log(dev2);
      dev2.social.twitter = "@coolman";
      console.log(wes.social);
      console.log(dev2.social);
    </script>
  </body>
</html>
```

</br>

# Ref.

- [JavaScript Fundamentals: Reference VS Copy #JavaScript30 14/30](https://www.youtube.com/watch?v=YnfwDQ5XYF4&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=14)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)

- [깊은 복사와 얕은 복사에 대한 심도있는 이야기](https://medium.com/watcha/%EA%B9%8A%EC%9D%80-%EB%B3%B5%EC%82%AC%EC%99%80-%EC%96%95%EC%9D%80-%EB%B3%B5%EC%82%AC%EC%97%90-%EB%8C%80%ED%95%9C-%EC%8B%AC%EB%8F%84%EC%9E%88%EB%8A%94-%EC%9D%B4%EC%95%BC%EA%B8%B0-2f7d797e008a)
