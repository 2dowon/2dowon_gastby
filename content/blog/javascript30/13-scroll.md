---
title: JavaScript30 - 13. Slide in on Scroll
date: 2021-04-02 22:04:52
category: javascript30
thumbnail: { thumbnailSrc }
draft: false
---

<img src="./image/scroll.gif"  width="800"/>

[Slide in on Scroll](https://2dowon.github.io/JavaScript30/html/13.html) 프로젝트는 스크롤에 따라서 이미지가 양 옆에서 등장하도록 만드는 것이다.

- 제공되는 debounce 함수 이해하기
- 스크롤했을 때 이미지 슬라이드가 등장해야 하는 지점 찾기

## 제공되는 debounce 함수 이해하기

- debounce 함수는 starter templete에 제공된 함수이다.
- 아래에서 checkSlide 함수를 확인할 수 있는데, 이 함수는 scroll 이벤트가 발생할 때마다 실행되는 함수이다. 근데 일반적으로 스크롤을 하게되면 길이가 길지 않은 페이지임에도 엄청 많은 양의 스크롤을 하게 된다

  > 스크롤 횟수는 아래처럼 `console.count()`를 통해서 확인할 수 있다.

  ```jsx
  function checkSlide(e) {
    console.count(e);
  }

  window.addEventListener("scroll", checkSlide);
  ```

- 하지만 debounce 함수를 이용하게 되면 기본적으로 20ms 동안은 checkSlide 함수가 실행되지 않도록 해주기 때문에 훨씬 적은 양의 checkSlide 함수를 실행하게 된다.

  > 실제로 스크롤된 수를 비교해보면 약 10배정도 차이났다.

  ```jsx
  function debounce(func, wait = 20, immediate = true) {
    var timeout;
    return function () {
      var context = this,
        args = arguments;
      var later = function () {
        timeout = null;
        if (!immediate) func.apply(context, args);
      };
      var callNow = immediate && !timeout;
      clearTimeout(timeout);
      timeout = setTimeout(later, wait);
      if (callNow) func.apply(context, args);
    };
  }

  function checkSlide(e) {
    console.count(e);
  }

  window.addEventListener("scroll", debounce(checkSlide));
  ```

## 스크롤했을 때 이미지 슬라이드가 등장해야 하는 지점 찾기

### 슬라이드가 등장하는 애니메이션

- 슬라이드가 등장하는 애니메이션은 CSS로 구현되어져 있다.
- 기본적으로 모든 이미지는 좌우 위치에 따라 align-left 또는 align-right 클래스가 적용되어 있고, 공통적으로 slide-in 클래스가 적용되어 있다.
- 마지막으로 슬라이드가 등장해야 하는 타이밍에 맞춰 슬라이드에 active 클래스를 추가해주면 된다.

```css
.slide-in {
  opacity: 0;
  transition: all 0.5s;
}

.align-left.slide-in {
  transform: translateX(-30%) scale(0.95);
}

.align-right.slide-in {
  transform: translateX(30%) scale(0.95);
}

.slide-in.active {
  opacity: 1;
  transform: translateX(0%) scale(1);
}
```

### 슬라이드가 등장해야 하는 지점 찾기

- `slideInAt` ⇒ 현재 보여지는 위치가 이미지가 반쯤 보이는 위치로 슬라이드가 등장해야 하는 지점

  ⇒ `slideInAt` 은 현재까지 스크롤된 영역`window.scrollY` + 브라우저의 내부 창 사이즈`window.innerHeight` - 이미지 높이의 절반`sliderImage.height / 2`으로 계산할 수 있다. 스크롤된 영역 + 브라우저의 내부 창 사이즈를 함으로써 현재 스크롤된 화면의 총 길이를 알 수 있고, 이미지의 반 정도까지 스크롤되면 그 때 이미지가 등장하도록 만들고 싶기 때문에 이미지 크기의 절반을 빼주는 것이다.

- `imageBottom` ⇒ 이미지의 바닥 좌표로 현재 이미지의 가장 상단의 위치`sliderImage.offsetTop`에 이미지의 높이`sliderImage.height`만큼을 더하면 구할 수 있다.

  (+) `imageBottom` 은 절대값으로 변하지 않는 값이다.

- `isHalfShown` 은 `slideInAt` 이 `sliderImage.offsetTop` 보다 큰지를 판단한다. 만약 크다면 그 때는 이미지가 등장해야 하는 타이밍이다. 이미지의 위치에서 반이상 스크롤 됐다는 뜻이기 때문이다.
- `isNotScrolledPast` 은 `window.scrollY` 가 `imageBottom` 보다 작은지를 판단한다. 만약 작다면 아직 이미지를 다 지나는 만큼 스크롤이 되지 않았다는 뜻이다. 즉, 이 때는 이미지가 아직 사라지면 안된다.
- 마지막으로 `isHalfShown` 과 `isNotScrolledPast` 모두 true라면 이미지 슬라이드가 보여져야 하는 상황이므로 active 클래스를 추가해주고, 둘 중 하나라도 false라면 이미지가 사라져야 하기 때문에 active 클래스를 제거해준다. 만약 제거하지 않는다면 한 번 등장한 이미지는 계속 등장한 상태이기 때문에 스크롤을 할 때마다 애니메이션이 작동하지 않게 된다.

```jsx
const sliderImages = document.querySelectorAll(".slide-in");

function checkSlide(e) {
  sliderImages.forEach((sliderImage) => {
    const slideInAt =
      window.scrollY + window.innerHeight - sliderImage.height / 2;
    const imageBottom = sliderImage.offsetTop + sliderImage.height;
    const isHalfShown = slideInAt > sliderImage.offsetTop;
    const isNotScrolledPast = window.scrollY < imageBottom;
    if (isHalfShown && isNotScrolledPast) {
      sliderImage.classList.add("active");
    } else {
      sliderImage.classList.remove("active");
    }
  });
}

window.addEventListener("scroll", debounce(checkSlide));
```

# 최종 코드

> index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Document</title>
    <link rel="stylesheet" href="../css/13.css" />
  </head>
  <body>
    <div class="site-wrap">
      <h1>Slide in on Scroll</h1>
      <p>
        Consectetur adipisicing elit. Tempore tempora rerum, est autem
        cupiditate, corporis a qui libero ipsum delectus quidem dolor at nulla,
        adipisci veniam in reiciendis aut asperiores omnis blanditiis quod quas
        laborum nam! Fuga ad tempora in aspernatur pariaturlores sunt esse
        magni, ut, dignissimos.
      </p>
      <p>
        Lorem ipsum cupiditate, corporis a qui libero ipsum delectus quidem
        dolor at nulla, adipisci veniam in reiciendis aut asperiores omnis
        blanditiis quod quas laborum nam! Fuga ad tempora in aspernatur pariatur
        fugit quibusdam dolores sunt esse magni, ut, dignissimos.
      </p>
      <p>Adipisicing elit. Tempore tempora rerum..</p>
      <p>
        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Tempore
        tempora rerum, est autem cupiditate, corporis a qui libero ipsum
        delectus quidem dolor at nulla, adipisci veniam in reiciendis aut
        asperiores omnis blanditiis quod quas laborum nam! Fuga ad tempora in
        aspernatur pariatur fugit quibusdam dolores sunt esse magni, ut,
        dignissimos.
      </p>
      <p>
        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Tempore
        tempora rerum, est autem cupiditate, corporis a qui libero ipsum
        delectus quidem dolor at nulla, adipisci veniam in reiciendis aut
        asperiores omnis blanditiis quod quas laborum nam! Fuga ad tempora in
        aspernatur pariatur fugit quibusdam dolores sunt esse magni, ut,
        dignissimos.
      </p>
      <p>
        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Tempore
        tempora rerum, est autem cupiditate, corporis a qui libero ipsum
        delectus quidem dolor at nulla, adipisci veniam in reiciendis aut
        asperiores omnis blanditiis quod quas laborum nam! Fuga ad tempora in
        aspernatur pariatur fugit quibusdam dolores sunt esse magni, ut,
        dignissimos.
      </p>
      <img src="http://unsplash.it/400/400" class="align-left slide-in" />
      <p>
        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Voluptates,
        deserunt facilis et iste corrupti omnis tenetur est. Iste ut est dicta
        dolor itaque adipisci, dolorum minima, veritatis earum provident error
        molestias. Ratione magni illo sint vel velit ut excepturi consectetur
        suscipit, earum modi accusamus voluptatem nostrum, praesentium numquam,
        reiciendis voluptas sit id quisquam. Consequatur in quis reprehenderit
        modi perspiciatis necessitatibus saepe, quidem, suscipit iure natus
        dignissimos ipsam, eligendi deleniti accusantium, rerum quibusdam fugit
        perferendis et optio recusandae sed ratione. Culpa, dolorum
        reprehenderit harum ab voluptas fuga, nisi eligendi natus maiores illum
        quas quos et aperiam aut doloremque optio maxime fugiat doloribus. Eum
        dolorum expedita quam, nesciunt
      </p>
      <img src="http://unsplash.it/400/401" class="align-right slide-in" />
      <p>
        at provident praesentium atque quas rerum optio dignissimos repudiandae
        ullam illum quibusdam. Vel ad error quibusdam, illo ex totam placeat.
        Quos excepturi fuga, molestiae ea quisquam minus, ratione dicta
        consectetur officia omnis, doloribus voluptatibus? Veniam ipsum
        veritatis architecto, provident quas consequatur doloremque quam quidem
        earum expedita, ad delectus voluptatum, omnis praesentium nostrum qui
        aspernatur ea eaque adipisci et cumque ab? Ea voluptatum dolore itaque
        odio. Eius minima distinctio harum, officia ab nihil exercitationem.
        Tempora rem nemo nam temporibus molestias facilis minus ipsam quam
        doloribus consequatur debitis nesciunt tempore officiis aperiam
        quisquam, molestiae voluptates cum, fuga culpa. Distinctio accusamus
        quibusdam, tempore perspiciatis dolorum optio facere consequatur quidem
        ullam beatae architecto, ipsam sequi officiis dignissimos amet impedit
        natus necessitatibus tenetur repellendus dolor rem! Dicta dolorem, iure,
        facilis illo ex nihil ipsa amet officia, optio temporibus eum autem odit
        repellendus nisi. Possimus modi, corrupti error debitis doloribus dicta
        libero earum, sequi porro ut excepturi nostrum ea voluptatem nihil
        culpa? Ullam expedita eligendi obcaecati reiciendis velit provident
        omnis quas qui in corrupti est dolore facere ad hic, animi soluta
        assumenda consequuntur reprehenderit! Voluptate dolor nihil veniam
        laborum voluptas nisi pariatur sed optio accusantium quam consectetur,
        corrupti, sequi et consequuntur, excepturi doloremque. Tempore quis
        velit corporis neque fugit non sequi eaque rem hic. Facere, inventore,
        aspernatur. Accusantium modi atque, asperiores qui nobis soluta cumque
        suscipit excepturi possimus doloremque odit saepe perferendis temporibus
        molestiae nostrum voluptatum quis id sint quidem nesciunt culpa. Rerum
        labore dolor beatae blanditiis praesentium explicabo velit optio esse
        aperiam similique, voluptatem cum, maiores ipsa tempore. Reiciendis sed
        culpa atque inventore, nam ullam enim expedita consectetur id velit
        iusto alias vitae explicabo nemo neque odio reprehenderit soluta sint
        eaque. Aperiam, qui ut tenetur, voluptate doloremque officiis dicta
        quaerat voluptatem rerum natus magni. Eum amet autem dolor ullam.
      </p>
      <img src="http://unsplash.it/200/500" class="align-left slide-in" />
      <p>
        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Distinctio
        maiores adipisci quibusdam repudiandae dolor vero placeat esse sit!
        Quibusdam saepe aperiam explicabo placeat optio, consequuntur nihil
        voluptatibus expedita quia vero perferendis, deserunt et incidunt
        eveniet
        <img src="http://unsplash.it/200/200" class="align-right slide-in" />
        temporibus doloremque possimus facilis. Possimus labore, officia dolore!
        Eaque ratione saepe, alias harum laboriosam deserunt laudantium
        blanditiis eum explicabo placeat reiciendis labore iste sint.
        Consectetur expedita dignissimos, non quos distinctio, eos rerum facilis
        eligendi. Asperiores laudantium, rerum ratione consequatur, culpa
        consectetur possimus atque ab tempore illum non dolor nesciunt. Neque,
        rerum. A vel non incidunt, quod doloremque dignissimos necessitatibus
        aliquid laboriosam architecto at cupiditate commodi expedita in, quae
        blanditiis. Deserunt labore sequi, repellat laboriosam est, doloremque
        culpa reiciendis tempore excepturi. Enim nostrum fugit itaque vel
        corporis ullam sed tenetur ipsa qui rem quam error sint, libero.
        Laboriosam rem, ratione. Autem blanditiis
      </p>
      <p>
        laborum neque repudiandae quam, cumque, voluptate veritatis itaque,
        placeat veniam ad nisi. Expedita, laborum reprehenderit ratione soluta
        velit natus, odit mollitia. Corporis rerum minima fugiat in nostrum.
        Assumenda natus cupiditate hic quidem ex, quas, amet ipsum esse dolore
        facilis beatae maxime qui inventore, iste? Maiores dignissimos dolore
        culpa debitis voluptatem harum, excepturi enim reiciendis, tempora ab
        ipsam illum aspernatur quasi qui porro saepe iure sunt eligendi tenetur
        quaerat ducimus quas sequi omnis aperiam suscipit! Molestiae obcaecati
        officiis quo, ratione eveniet, provident pariatur. Veniam quasi expedita
        distinctio, itaque molestiae sequi, dolorum nisi repellendus quia
        facilis iusto dignissimos nam? Tenetur fugit quos autem nihil,
        perspiciatis expedita enim tempore, alias ab maiores quis necessitatibus
        distinctio molestias eum, quidem. Delectus impedit quidem laborum, fugit
        vel neque quo, ipsam, quasi aspernatur quas odio nihil? Veniam amet
        reiciendis blanditiis quis reprehenderit repudiandae neque, ab ducimus,
        odit excepturi voluptate saepe ipsam. Voluptatem eum error voluptas
        porro officiis, amet! Molestias, fugit, ut! Tempore non magnam, amet,
        facere ducimus accusantium eos veritatis neque.
      </p>
      <img src="http://unsplash.it/400/400" class="align-right slide-in" />
      <p>
        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Distinctio
        maiores adipisci quibusdam repudiandae dolor vero placeat esse sit!
        Quibusdam saepe aperiam explicabo placeat optio, consequuntur nihil
        voluptatibus expedita quia vero perferendis, deserunt et incidunt
        eveniet temporibus doloremque possimus facilis. Possimus labore, officia
        dolore! Eaque ratione saepe, alias harum laboriosam deserunt laudantium
        blanditiis eum explicabo placeat reiciendis labore iste sint.
        Consectetur expedita dignissimos, non quos distinctio, eos rerum facilis
        eligendi. Asperiores laudantium, rerum ratione consequatur, culpa
        consectetur possimus atque ab tempore illum non dolor nesciunt. Neque,
        rerum. A vel non incidunt, quod doloremque dignissimos necessitatibus
        aliquid laboriosam architecto at cupiditate commodi expedita in, quae
        blanditiis. Deserunt labore sequi, repellat laboriosam est, doloremque
        culpa reiciendis tempore excepturi. Enim nostrum fugit itaque vel
        corporis ullam sed tenetur ipsa qui rem quam error sint, libero.
        Laboriosam rem, ratione. Autem blanditiis laborum neque repudiandae
        quam, cumque, voluptate veritatis itaque, placeat veniam ad nisi.
        Expedita, laborum reprehenderit ratione soluta velit natus, odit
        mollitia. Corporis rerum minima fugiat in nostrum. Assumenda natus
        cupiditate hic quidem ex, quas, amet ipsum esse dolore facilis beatae
        maxime qui inventore, iste? Maiores dignissimos dolore culpa debitis
        voluptatem harum, excepturi enim reiciendis, tempora ab ipsam illum
        aspernatur quasi qui porro saepe iure sunt eligendi tenetur quaerat
        ducimus quas sequi omnis aperiam suscipit! Molestiae obcaecati officiis
        quo, ratione eveniet, provident pariatur. Veniam quasi expedita
        distinctio, itaque molestiae sequi, dolorum nisi repellendus quia
        facilis iusto dignissimos nam? Tenetur fugit quos autem nihil,
        perspiciatis expedita enim tempore, alias ab maiores quis necessitatibus
        distinctio molestias eum, quidem. Delectus impedit quidem laborum, fugit
        vel neque quo, ipsam, quasi aspernatur quas odio nihil? Veniam amet
        reiciendis blanditiis quis reprehenderit repudiandae neque, ab ducimus,
        odit excepturi voluptate saepe ipsam. Voluptatem eum error voluptas
        porro officiis, amet! Molestias, fugit, ut! Tempore non magnam, amet,
        facere ducimus accusantium eos veritatis neque.
      </p>
      <p>
        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Distinctio
        maiores adipisci quibusdam repudiandae dolor vero placeat esse sit!
        Quibusdam saepe aperiam explicabo placeat optio, consequuntur nihil
        voluptatibus expedita quia vero perferendis, deserunt et incidunt
        eveniet temporibus doloremque possimus facilis. Possimus labore, officia
        dolore! Eaque ratione saepe, alias harum laboriosam deserunt laudantium
        blanditiis eum explicabo placeat reiciendis labore iste sint.
        Consectetur expedita dignissimos, non quos distinctio, eos rerum facilis
        eligendi. Asperiores laudantium, rerum ratione consequatur, culpa
        consectetur possimus atque ab tempore illum non dolor nesciunt. Neque,
        rerum. A vel non incidunt, quod doloremque dignissimos necessitatibus
        aliquid laboriosam architecto at cupiditate commodi expedita in, quae
        blanditiis. Deserunt labore sequi, repellat laboriosam est, doloremque
        culpa reiciendis tempore excepturi. Enim nostrum fugit itaque vel
        corporis ullam sed tenetur ipsa qui rem quam error sint, libero.
        Laboriosam rem, ratione. Autem blanditiis laborum neque repudiandae
        quam, cumque, voluptate veritatis itaque, placeat veniam ad nisi.
        Expedita, laborum reprehenderit ratione soluta velit natus, odit
        mollitia. Corporis rerum minima fugiat in nostrum. Assumenda natus
        cupiditate hic quidem ex, quas, amet ipsum esse dolore facilis beatae
        maxime qui inventore, iste? Maiores dignissimos dolore culpa debitis
        voluptatem harum, excepturi enim reiciendis, tempora ab ipsam illum
        aspernatur quasi qui porro saepe iure sunt eligendi tenetur quaerat
        ducimus quas sequi omnis aperiam suscipit! Molestiae obcaecati officiis
        quo, ratione eveniet, provident pariatur. Veniam quasi expedita
        distinctio, itaque molestiae sequi, dolorum nisi repellendus quia
        facilis iusto dignissimos nam? Tenetur fugit quos autem nihil,
        perspiciatis expedita enim tempore, alias ab maiores quis necessitatibus
        distinctio molestias eum, quidem. Delectus impedit quidem laborum, fugit
        vel neque quo, ipsam, quasi aspernatur quas odio nihil? Veniam amet
        reiciendis blanditiis quis reprehenderit repudiandae neque, ab ducimus,
        odit excepturi voluptate saepe ipsam. Voluptatem eum error voluptas
        porro officiis, amet! Molestias, fugit, ut! Tempore non magnam, amet,
        facere ducimus accusantium eos veritatis neque.
      </p>
    </div>
    <script src="../js/13.js"></script>
  </body>
</html>
```

> style.css

```css
html {
  box-sizing: border-box;
  background: #ffc600;
  font-family: "helvetica neue";
  font-size: 20px;
  font-weight: 200;
}

body {
  margin: 0;
}

*,
*:before,
*:after {
  box-sizing: inherit;
}

h1 {
  margin-top: 0;
}

.site-wrap {
  max-width: 700px;
  margin: 100px auto;
  background: white;
  padding: 40px;
  text-align: justify;
}

.align-left {
  float: left;
  margin-right: 20px;
}

.align-right {
  float: right;
  margin-left: 20px;
}

.slide-in {
  opacity: 0;
  transition: all 0.5s;
}

.align-left.slide-in {
  transform: translateX(-30%) scale(0.95);
}

.align-right.slide-in {
  transform: translateX(30%) scale(0.95);
}

.slide-in.active {
  opacity: 1;
  transform: translateX(0%) scale(1);
}
```

> JS

```jsx
function debounce(func, wait = 20, immediate = true) {
  var timeout;
  return function () {
    var context = this,
      args = arguments;
    var later = function () {
      timeout = null;
      if (!immediate) func.apply(context, args);
    };
    var callNow = immediate && !timeout;
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
    if (callNow) func.apply(context, args);
  };
}

const sliderImages = document.querySelectorAll(".slide-in");

function checkSlide(e) {
  sliderImages.forEach((sliderImage) => {
    const slideInAt =
      window.scrollY + window.innerHeight - sliderImage.height / 2;
    const imageBottom = sliderImage.offsetTop + sliderImage.height;
    const isHalfShown = slideInAt > sliderImage.offsetTop;
    const isNotScrolledPast = window.scrollY < imageBottom;
    if (isHalfShown && isNotScrolledPast) {
      sliderImage.classList.add("active");
    } else {
      sliderImage.classList.remove("active");
    }
  });
}

window.addEventListener("scroll", debounce(checkSlide));
```

</br>

# Ref.

- [Vanilla JavaScript Slide In on Scroll - #JavaScript30 13/30](https://www.youtube.com/watch?v=uzRsENVD3W8&list=PLu8EoSxDXHP6CGK4YVJhL_VWetA865GOH&index=13)

- [JAVASCRIPT 30](https://2dowon.github.io/JavaScript30/)

- [https://mommoo.tistory.com/85](https://mommoo.tistory.com/85)
