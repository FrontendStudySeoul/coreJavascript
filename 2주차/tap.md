# this

this의 결정시점: **`실행 컨텍스트가 생성`될 때 결정 === `함수를 호출`할 때 결정**

## 환경에 따른 전역 this

```js
// 브라우저 환경
console.log(this); // 브라우저 Window 객체
console.log(window); // 브라우저 Window 객체
console.log(window === this); // true
```

```js
// Node.js 환경
console.log(this); // Node.js global 객체
console.log(global); // Node.js global 객체
console.log(global === this); // true
```

MDN: [globalThis](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/globalThis)

### 전역 변수와 전역 객체

**자바스크립트의 `모든 변수`는 `특정 객체의 프로퍼티`로 동작한다.**

```js
var a = 1;
console.log(a); // 1
console.log(window.a); // 1
console.log(this.a); // 1
```

만약 이미 window에 존재하는 함수를 재정의한다면?

1. (1)과 (2)의 결과값은?
2. 다시 실행 후 (1)과 (2)의 결과값은?

```js
// (1)
console.log(setTimeout);

var setTimeout = function () {
  return "fake setTimeout";
};

// (2)
console.log(setTimeout); // ?
console.log(window.setTimeout); // ?
console.log(this.setTimeout); // ?
```

즉, **전역 변수를 선언하면 `자바스크립트 엔진`은 이를 `전역객체의 프로퍼티로 할당`한다.**

### delete 연산자의 전역 객체

```js
var a = 1;
delete window.a; // false (why?)
console.log(a, window.a, this.a); // 1 1 1
```

```js
var a = 1;
delete a; // false (why?)
console.log(a, window.a, this.a); // 1 1 1
```

```js
window.a = 1;
delete window.a; // true (why?)
// or
// delete a; // true
console.log(a, window.a, this.a); // Uncaught ReferenceError: a is not defined
```

처음부터 `전역객체의 프로퍼티로 할당`한 경우에는 삭제가 되지만 `전역변수로 선언`한 경우에는 삭제가 되지 않는다.

이는 자바스크립트 엔진이 [`delete`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/delete#return_value) 연산자 내부에 [`configurable`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty#%EC%84%A4%EB%AA%85) 속성을 `false`로 정의하게 한다.

## 함수 vs 메서드

- 둘의 차이는 `독립성`
- 함수를 호출할 때 함수 이름(프로퍼티명) 앞에 `객체가 명시`된 경우에는 `메서드`로써 호출, 나머지는 `함수`로써 호출
- 설계상 오류...
  > 실행 컨텍스트를 활성화할 당시 this가 지정되지 않는 경우 this는 `전역 객체`를 바라본다. 따라서 `함수`에서의 this는 `전역 객체`를 가리킨다.
- this 바인딩은 해당 함수를 호출하는 구문 앞에 `점(.)` 또는 `대괄호표기(['property'])`의 유무로 판단하면된다.

```js
var obj1 = {
  outer: function () {
    console.log(this); // (1)
    var innerFunc = function () {
      console.log(this); // (2) (3)
    };
    innerFunc();

    var obj2 = {
      innerMethod: innerFunc,
    };
    obj2.innerMethod();
  },
};
obj1.outer();
```

<details>
  <summary>정답</summary>
  (1): obj1
  (2): Window
  (3): obj2
</details>

### 콜백 함수 호출시 함수 내부에서의 this

```js
setTimeout(function () {
  console.log(this); // (1)
}, 300);
```

<details>
  <summary>정답</summary>
  3초 뒤, Window
</details>

```js
[(1, 2, 3, 4, 5)].forEach(function (x) {
  console.log(this, x); // (2)
});
```

<details>
  <summary>정답</summary>
  <div>Window, 1</div>
  <div>Window, 2</div>
  <div>Window, 3</div>
  <div>Window, 4</div>
  <div>Window, 5</div>
</details>

```html
<!DOCTYPE html>
<html>
  <body>
    <button id="btn">Click Me!</button>
    <p id="demo"></p>
    <script>
      let hello = "";

      hello = (event) => {
        document.getElementById("demo").innerHTML += this;
        console.log(window, this, event);
      };

      // (3)
      window.addEventListener("load", hello);
      // (4)
      document.getElementById("btn").addEventListener("click", hello);
      // (5)
      document
        .getElementById("btn")
        .addEventListener("click", function (event) {
          console.log("window:", window, "this:", this, ", event:", event);
        });
    </script>
  </body>
</html>
```

W3C: [this Keyword](https://www.w3schools.com/js/js_this.asp)

<details>
  <summary>정답</summary>
  <div>(3) load: Window, click: Window, event: Event(type: load)</div>
  <div>(4) window: Window, this: Window, event: PointEvent</div>
  <div>(5) window: Window, this:  버튼 Element, event: PointEvent</div>
</details>

즉, 콜백 함수는 내부 로직에 따라 실행되는데, this는 기본적으로 `전역객체`를 참조하지만, `제어권`을 받은 함수에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조한다.

다시 위 `addEventListener`의 콜백을 보면 `addEventListener`함수의 제어권을 `hello`라는 함수로 넘겨주었기 때문에 내부 로직에서 this가 될 대상이 전역이 아니라 제어권을 넘긴 `addEventListener`가 된다. (메서드로 보면됨)

### 생성자 함수 내부의 this

- 생성자는 `인스턴스`를 만들기 위한 `틀`이다.
- 어떤 함수가 생성자 함수로서 호출된 경우 내부에서는 this는 곧 새로 만들 구체적인 인스턴스 자신이 된다.
- 생성자 함수 호춠히 `prototype 프로퍼티`를 참조하는 `__proto__`라는 프로퍼티가 있는 객체(인스턴스)를 만들고, 미리 준비된 공통 속성 및 개성을 해당 객체(this)에 부여한다.

```js
var Cat = function (name, age) {
  this.bark = "야옹";
  this.name = name;
  this.age = age;
};
var nabi = new Cat("나비", 7);
console.log(nabi); // Cat { bark: '야옹', name: '나비', age: 5}
```

## 화살표 함수에서의 this

기본적으로 화살표 함수는 this 바인딩을 제공하지 않는다. 자신을 감싼 정적 범위이다.

```js
var obj = {
  bar: function () {
    var x = () => this;
    return x;
  },
};

// obj의 메소드로써 bar를 호출하고, this를 obj로 설정
// 반환된 함수로의 참조를 fn에 할당
var fn = obj.bar();

// this 설정 없이 fn을 호출하면,
// 기본값으로 global 객체 또는 엄격 모드에서는 undefined
console.log(fn()); // (1)
console.log(fn() === obj); // (2)

var fn2 = obj.bar;
// 화살표 함수의 this를 bar 메소드 내부에서 호출하면
// fn2의 this를 따르므로 window를 반환
console.log(fn2()()); // (3)
console.log(fn2()() == window); // (4)
```

<details>
  <summary>정답</summary>
  <div>(1) {bar: 𝑓}</div>
  <div>(2) true</div>
  <div>(3) Window</div>
  <div>(4) true</div>
</details>

MDN: [화살표 함수](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this#%ED%99%94%EC%82%B4%ED%91%9C_%ED%95%A8%EC%88%98)

### 객체의 프로토타입 체인에서 this

```js
var o = {
  f: function () {
    return this.a + this.b;
  },
};
var p = Object.create(o);
console.log(o, p); // (1)

p.a = 1;
p.b = 4;

console.log(o, p); // (2)
console.log(p.f()); // (3)
```

<details>
  <summary>정답</summary>
  <div>(1) {f: 𝑓}, {}</div>
  <div>(2) {f: 𝑓}, {a:1, b: 4}</div>
  <div>(3) 5</div>
</details>
<details>
  <summary>해설</summary>
  <p>
    
    객체의 프로토타입 체인에의해 메서드가 어떤 객체의 프로토타입 체인 위에 존재하면, this의 값은 그 객체가 메서드를 가진 것처럼 설정된다.

    이 예제에서, f 속성을 가지고 있지 않은 변수 p 가 할당된 객체는, 프로토타입으로 부터 상속받는다. 그러나 그것은 결국 o 에서 f 이름을 가진 멤버를 찾는 되는 문제가 되지 않는다; p.f 를 찾아 참조하기 시작하므로, 함수 내부의 this 는 p 처럼 나타나는 객체 값을 취한다.

    즉, f 는 p 의 메소드로서 호출된 이후로, this 는 p 를 나타낸다. 이것은 JavaScript 의 프로토타입 상속의 흥미로운 기능이다.

  </p>
</details>

### 접근자와 설정자의 this

```js
function sum() {
  return this.a + this.b + this.c;
}

var o = {
  a: 1,
  b: 2,
  c: 3,
  get average() {
    return (this.a + this.b + this.c) / 3;
  },
};

Object.defineProperty(o, "sum", {
  get: sum,
  enumerable: true,
  configurable: true,
});

console.log(o.average, o.sum); // (1)
```

<details>
  <summary>정답</summary>
  <div>(1) 2, 6</div>
</details>
<details>
  <summary>해설</summary>
  <p>
    
    다시 한 번 같은 개념으로, 함수를 접근자와 설정자에서 호출하더라도 동일합니다. 접근자나 설정자로 사용하는 함수의 this는 접근하거나 설정하는 속성을 가진 객체로 묶입니다.

  </p>
</details>
