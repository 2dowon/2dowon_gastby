---
title: JavaScript30 - 18. Adding Up Times with Reduce
date: 2021-04-10 16:04:62
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

[Adding Up Times with Reduce](https://2dowon.github.io/JavaScript30/html/18.html) 프로젝트의 핵심은 다음과 같다.

- `분:초` 형식으로 저장되어 있는 동영상의 길이를 초로 변환해서 모든 동영상들의 길이의 합이 몇 초인지 구하기
- 동영상의 길이를 `시간 분 초` 형식으로 바꾸기

## `분:초` 형식으로 저장되어 있는 동영상의 길이를 초로 변환해서 모든 동영상들의 길이의 합이 몇 초인지 구하기

### ✍️ My Code

- `분:초` 형식으로 동영상의 길이가 저장되어 있으므로 `:` 를 기준으로 0번째 요소가 분, 1번째 요소가 초이다.
- string을 split 함수를 이용해 구분했으므로 모든 요소는 string이다. 따라서 이를 계산하기 위해서는 `parseInt` 를 이용해 숫자로 형변환을 해줘야 한다.
- 1분은 60초이므로 분의 요소에는 60을 곱해서 더해주고, 초는 그대로 더해주면 된다.
- 그렇게 모든 동영상의 요소들을 반복하면 총 시간의 길이를 알 수 있다

```jsx
const timeNodes = document.querySelectorAll("[data-time]");

let seconds = 0;

function calcSec(timeNode) {
  seconds += parseInt(timeNode.split(":")[0]) * 60;
  seconds += parseInt(timeNode.split(":")[1]);
}
timeNodes.forEach((timeNode) => calcSec(timeNode.dataset.time));
```

### 👍 Wes Bos Code

- 나는 forEach를 이용해서 timeNodes의 모든 요소를 반복했는데, Wes Bos는 map을 이용했다.
- timeNodes의 요소마다 초를 구해서 계속 더하는 것이므로 `reduce` 함수를 이용했다.

```jsx
const timeNodes = Array.from(document.querySelectorAll("[data-time]"));

const seconds = timeNodes
  .map((node) => node.dataset.time)
  .map((timeCode) => {
    const [mins, secs] = timeCode.split(":").map(parseFloat);
    return mins * 60 + secs;
  })
  .reduce((total, vidSeconds) => total + vidSeconds);
```

## 동영상의 길이를 `시간 분 초` 형식으로 바꾸기

### ✍️ My Code

나눴을 때 몫을 구하기 위해 `parseInt` 를 이용했다. 나머지는 `%` 를 이용하면 된다.

```jsx
const timeNodes = document.querySelectorAll("[data-time]");

let seconds = 0;

function calcSec(timeNode) {
  seconds += parseInt(timeNode.split(":")[0]) * 60;
  seconds += parseInt(timeNode.split(":")[1]);
}
timeNodes.forEach((timeNode) => calcSec(timeNode.dataset.time));

let secondsLeft = seconds;

const hour = parseInt(secondsLeft / 3600);
const min = parseInt((secondsLeft - hour * 3600) / 60);
const sec = (secondsLeft - hour * 3600) % 60;

console.log(hour, min, sec);
```

### 👍 Wes Bos Code

나눴을 때 몫을 구하기 위해 내림을 계산하는 `Math.floor()` 를 이용했다.

```jsx
const timeNodes = Array.from(document.querySelectorAll("[data-time]"));

const seconds = timeNodes
  .map((node) => node.dataset.time)
  .map((timeCode) => {
    const [mins, secs] = timeCode.split(":").map(parseFloat);
    return mins * 60 + secs;
  })
  .reduce((total, vidSeconds) => total + vidSeconds);
let secondsLeft = seconds;
const hours = Math.floor(secondsLeft / 3600);
secondsLeft = secondsLeft % 3600;
const mins = Math.floor(secondsLeft / 60);
secondsLeft = secondsLeft % 60;
console.log(hours, mins, secondsLeft);
```

# 최종 코드

> My Code

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Videos</title>
  </head>
  <body>
    <ul class="videos">
      <li data-time="5:43">Video 1</li>
      <li data-time="2:33">Video 2</li>
      <li data-time="3:45">Video 3</li>
      <li data-time="0:47">Video 4</li>
      <li data-time="5:21">Video 5</li>
      <li data-time="6:56">Video 6</li>
      <li data-time="3:46">Video 7</li>
      <li data-time="5:25">Video 8</li>
      <li data-time="3:14">Video 9</li>
      <li data-time="3:31">Video 10</li>
      <li data-time="5:59">Video 11</li>
      <li data-time="3:07">Video 12</li>
      <li data-time="11:29">Video 13</li>
      <li data-time="8:57">Video 14</li>
      <li data-time="5:49">Video 15</li>
      <li data-time="5:52">Video 16</li>
      <li data-time="5:50">Video 17</li>
      <li data-time="9:13">Video 18</li>
      <li data-time="11:51">Video 19</li>
      <li data-time="7:58">Video 20</li>
      <li data-time="4:40">Video 21</li>
      <li data-time="4:45">Video 22</li>
      <li data-time="6:46">Video 23</li>
      <li data-time="7:24">Video 24</li>
      <li data-time="7:12">Video 25</li>
      <li data-time="5:23">Video 26</li>
      <li data-time="3:34">Video 27</li>
      <li data-time="8:22">Video 28</li>
      <li data-time="5:17">Video 29</li>
      <li data-time="3:10">Video 30</li>
      <li data-time="4:43">Video 31</li>
      <li data-time="19:43">Video 32</li>
      <li data-time="0:47">Video 33</li>
      <li data-time="0:47">Video 34</li>
      <li data-time="3:14">Video 35</li>
      <li data-time="3:59">Video 36</li>
      <li data-time="2:43">Video 37</li>
      <li data-time="4:17">Video 38</li>
      <li data-time="6:56">Video 39</li>
      <li data-time="3:05">Video 40</li>
      <li data-time="2:06">Video 41</li>
      <li data-time="1:59">Video 42</li>
      <li data-time="1:49">Video 43</li>
      <li data-time="3:36">Video 44</li>
      <li data-time="7:10">Video 45</li>
      <li data-time="3:44">Video 46</li>
      <li data-time="3:44">Video 47</li>
      <li data-time="4:36">Video 48</li>
      <li data-time="3:16">Video 49</li>
      <li data-time="1:10">Video 50</li>
      <li data-time="6:10">Video 51</li>
      <li data-time="2:14">Video 52</li>
      <li data-time="3:44">Video 53</li>
      <li data-time="5:05">Video 54</li>
      <li data-time="6:03">Video 55</li>
      <li data-time="12:39">Video 56</li>
      <li data-time="1:56">Video 57</li>
      <li data-time="4:04">Video 58</li>
    </ul>
    <script>
      const timeNodes = document.querySelectorAll("[data-time]");

      let seconds = 0;

      function calcSec(timeNode) {
        seconds += parseInt(timeNode.split(":")[0]) * 60;
        seconds += parseInt(timeNode.split(":")[1]);
      }
      timeNodes.forEach((timeNode) => calcSec(timeNode.dataset.time));

      let secondsLeft = seconds;

      const hour = parseInt(secondsLeft / 3600);
      const min = parseInt((secondsLeft - hour * 3600) / 60);
      const sec = (secondsLeft - hour * 3600) % 60;

      console.log(hour, min, sec);
    </script>
  </body>
</html>
```

> Wes Bos Code

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Videos</title>
  </head>
  <body>
    <ul class="videos">
      <li data-time="5:43">Video 1</li>
      <li data-time="2:33">Video 2</li>
      <li data-time="3:45">Video 3</li>
      <li data-time="0:47">Video 4</li>
      <li data-time="5:21">Video 5</li>
      <li data-time="6:56">Video 6</li>
      <li data-time="3:46">Video 7</li>
      <li data-time="5:25">Video 8</li>
      <li data-time="3:14">Video 9</li>
      <li data-time="3:31">Video 10</li>
      <li data-time="5:59">Video 11</li>
      <li data-time="3:07">Video 12</li>
      <li data-time="11:29">Video 13</li>
      <li data-time="8:57">Video 14</li>
      <li data-time="5:49">Video 15</li>
      <li data-time="5:52">Video 16</li>
      <li data-time="5:50">Video 17</li>
      <li data-time="9:13">Video 18</li>
      <li data-time="11:51">Video 19</li>
      <li data-time="7:58">Video 20</li>
      <li data-time="4:40">Video 21</li>
      <li data-time="4:45">Video 22</li>
      <li data-time="6:46">Video 23</li>
      <li data-time="7:24">Video 24</li>
      <li data-time="7:12">Video 25</li>
      <li data-time="5:23">Video 26</li>
      <li data-time="3:34">Video 27</li>
      <li data-time="8:22">Video 28</li>
      <li data-time="5:17">Video 29</li>
      <li data-time="3:10">Video 30</li>
      <li data-time="4:43">Video 31</li>
      <li data-time="19:43">Video 32</li>
      <li data-time="0:47">Video 33</li>
      <li data-time="0:47">Video 34</li>
      <li data-time="3:14">Video 35</li>
      <li data-time="3:59">Video 36</li>
      <li data-time="2:43">Video 37</li>
      <li data-time="4:17">Video 38</li>
      <li data-time="6:56">Video 39</li>
      <li data-time="3:05">Video 40</li>
      <li data-time="2:06">Video 41</li>
      <li data-time="1:59">Video 42</li>
      <li data-time="1:49">Video 43</li>
      <li data-time="3:36">Video 44</li>
      <li data-time="7:10">Video 45</li>
      <li data-time="3:44">Video 46</li>
      <li data-time="3:44">Video 47</li>
      <li data-time="4:36">Video 48</li>
      <li data-time="3:16">Video 49</li>
      <li data-time="1:10">Video 50</li>
      <li data-time="6:10">Video 51</li>
      <li data-time="2:14">Video 52</li>
      <li data-time="3:44">Video 53</li>
      <li data-time="5:05">Video 54</li>
      <li data-time="6:03">Video 55</li>
      <li data-time="12:39">Video 56</li>
      <li data-time="1:56">Video 57</li>
      <li data-time="4:04">Video 58</li>
    </ul>
    <script>
      const timeNodes = Array.from(document.querySelectorAll("[data-time]"));

      const seconds = timeNodes
        .map((node) => node.dataset.time)
        .map((timeCode) => {
          const [mins, secs] = timeCode.split(":").map(parseFloat);
          return mins * 60 + secs;
        })
        .reduce((total, vidSeconds) => total + vidSeconds);
      let secondsLeft = seconds;
      const hours = Math.floor(secondsLeft / 3600);
      secondsLeft = secondsLeft % 3600;
      const mins = Math.floor(secondsLeft / 60);
      secondsLeft = secondsLeft % 60;
      console.log(hours, mins, secondsLeft);
    </script>
  </body>
</html>
```

</br>

# Ref.

- [How JavaScript's Array Reduce Works - #JavaScript30 18/30](https://www.youtube.com/watch?v=SadWPo2KZWg&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=18)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)
