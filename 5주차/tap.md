# 6장 프로토타입

자바스크립트는 프로토타입 기반 언어이다. 클래스 기반 언어에서는 '상속'을 사용하지만 프로토타입 기반 언어에서는 어떤 객체를 원형으로 삼고 이를 복제(참조)함으로써 상속과 비슷한 효과를 얻는다.

## constructor, prototype, instance

![image](https://github.com/user-attachments/assets/de5c2828-47ae-4b29-b8e0-83eb247fa406)

- 어떤 생성자 함수(Constructor)를 new 연산자와 함께 호출하면
- Constructor에서 정의된 내용을 바탕으로 새로운 인스턴스가 생성된다.
- 이때 instance에는 `__proto__` 라는 프로퍼티가 자동을 부여되는데
- 이 프로퍼티는 Constructor의 prototype라는 프로퍼티를 참조한다.

prototype와 이를 참조하는 `__proto__`는 모두 `객체`이다. prototype 객체 내부에는 인스턴스가 사용할 메서드를 저장한다.

> 참고) [`__proto__`가 deprecated된 이유](https://www.inflearn.com/questions/361135/comment/151689)

---

> Note

- ES5.1 명세서에선 `__proto__`가 아니라 `[[prototype]]`으로 명시되어있고, `__proto__`는 `[[prototype]]`의 구현체일뿐이다.
- `instance.__proto__`로 직접 접근은 허용하지 않고 오직 `Object.getPrototypeOf(instacne)` 또는 `Reflect.getPrototypeOf(instance)`를 통해서만 접근하도록 정의했다. (아직까지 권장하지 않지만 많은 사람들이 직접 접근을 사용해서 "호환성 유지"를 위해 레거시로 정식 인정하게됨)
- 실무에선 가급적 `__proto__` 사용하지 않고 [`Object.getPrototypeOf()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf) 또는 [`Object.create`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/create)를 이용 권장.

cf) [`handler.getPrototypeOf()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/getPrototypeOf)라는 메서드가 있는데 이 메서드는 [[prototype]] 객체를 가지는것이 아닌 [[GetPrototypeOf]] [내부 메서드](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy#object_internal_methods)에 대한 트랩입니다.

<details>
  <summary>example - handler.getPrototypeOf()</summary>

    const monster1 = {
        eyeCount: 4,
    };

    const monsterPrototype = {
        eyeCount: 2,
    };

    const handler = {
        getPrototypeOf(target) {
            return monsterPrototype;
        },
    };

    const proxy1 = new Proxy(monster1, handler);

    console.log(Object.getPrototypeOf(proxy1) , monsterPrototype)
    console.log(Object.getPrototypeOf(proxy1) === monsterPrototype);
    // Expected output: true

    console.log(Object.getPrototypeOf(proxy1).eyeCount);
    // Expected output: 2

</details>

<details>
  <summary>example - Object.create()</summary>

    // Shape - 상위클래스
    function Shape() {
        this.x = 0;
        this.y = 0;
    }

    // 상위클래스 메서드
    Shape.prototype.move = function (x, y) {
        this.x += x;
        this.y += y;
        console.info("Shape moved.", this.x, this.y);
    };

    // Rectangle - 하위클래스
    function Rectangle() {
        Shape.call(this); // super 생성자 호출.
    }

    // 하위클래스는 상위클래스를 확장
    Rectangle.prototype = Object.create(Shape.prototype);
    console.log("1 Rectangle.prototype", Rectangle.prototype)
    Rectangle.prototype.constructor = Rectangle;
    console.log("2 Rectangle.prototype", Rectangle.prototype)


    var rect = new Rectangle();

    console.log("Is rect an instance of Rectangle?", rect instanceof Rectangle); // true
    console.log("Is rect an instance of Shape?", rect instanceof Shape); // true
    rect.move(1, 1); // Outputs, 'Shape moved.'

</details>

[2주차 - 객체의 프로토타입 체인에서 this](https://github.com/FrontendStudySeoul/coreJavascript/blob/main/2%EC%A3%BC%EC%B0%A8/tap.md#%EA%B0%9D%EC%B2%B4%EC%9D%98-%ED%94%84%EB%A1%9C%ED%86%A0%ED%83%80%EC%9E%85-%EC%B2%B4%EC%9D%B8%EC%97%90%EC%84%9C-this)

---

### 프로토타입 생성자 함수

Person이라는 생성자 함수의 prototype에 getName이라는 메서드를 지정

```js
let Person = function (name) {
  this._name = name;
};

Person.prototype.getName = function () {
  return this._name;
};

let suzi = new Person("Suzi");
suzi.__proto__.getName(); // undefined

Person.prototype === suzi.__proto__; // true

suzi.getName(); // Suzi
```

- undefined가 출력되고`에러가 발생하지 않았다`. 즉, `호출 가능한 함수`이다.
- Person의 인스턴스는 proto 프로퍼티로 getName 호출이 가능하다
- `undefined`가 되는 이유는 함수를 메서드로써 호출하면 앞의 객체가 `this`이기때문에 `__proto__` 객체에는 name 프로퍼티가 없기 때문에 undefined가 반환된다. (찾고자하는 식별자가 없다면 Error 대신에 undefined를 반환하는 것이 자바스크립트 규약)
- suzi의 name 프로퍼티를 접근하고 싶다면 `__proto__`를 생략 (원래 생략 가능한 스펙)

이런 특성 덕분에 `생성자 함수`의 prototype에 어떤 _메서드나 프로퍼티_ 가 있다면, `인스턴스`에서도 마치 자신의 것처럼 해 당 _메서드나 프로퍼티에 접근_ 할 수 있게 된다.

한편 prototype 프로퍼티 내부에 있지 않은 메서드들은 인스턴스가 직접 호출할 수 없기 때문에 생성자 함수에서 직접 접근해야 실행이 가능하다.

```js
var arr = [1, 2];
arr.forEach(function () {}); // (0)
Array.isArray(arr); // (0) true
arr.isArray(); // (x) TypeError: arr.isArray is not a function
```

## constructor 프로퍼티

생성자 함수의 프로퍼티인 Prototype 내부에는 consturctor라는 프로퍼티가 있다. 인스턴스의 **proto** 객체에도 마찬가지이다. 원래의 생성자 함수(자기 자신)을 참조하는데, 인스턴스로부터 그 원형을 알 수 있는 수단이기 때문이다.

```js
let arr = [1, 2];
Array.prototype.constructor == Array; // true
arr.__proto__.constructor == Array; // true
arr.constructor == Array; // true

let arr2 = new arr.constructor(3, 4);
console.log(arr2); // [3, 4]
```

constructor는 읽기 전용 속성(기본형 리터럴 변수 - number, string, boolean)이 부여된 예외적인 경우를 제외하고는 값을 바꿀 수 있다.

```js
let NewConstructor = function () {
  console.log("this is new constructor!");
};
let dataTypes = [
  1, // 🔴 Number & false
  "test", // 🔴 String & false
  true, // 🔴 Boolean & false
  {}, // NewConstructor & false
  [], // NewConstructor & false
  function () {}, // NewConstructor & false
  /test/, // NewConstructor & false
  new Number(), // NewConstructor & false
  new String(), // NewConstructor & false
  new Boolean(), // NewConstructor & false
  new Object(), // NewConstructor & false
  new Array(), // NewConstructor & false
  new Function(), // NewConstructor & false
  new RegExp(), // NewConstructor & false
  new Date(), // NewConstructor & false
  new Error(), // NewConstructor & false
];

dataTypes.forEach(function (d) {
  d.constructor = NewConstructor;
  console.log(d.constructor.name, "&", d instanceof NewConstructor);
});
```

모든 데이터가 d instanceof NewConstructor 명령어에 대해 false를 반환하는데, **constructor를 변경하더라도 `참조하는 대상이 변경`될 뿐 이미 만들어진 `인스턴스의 원형`이 바뀐다거나 `데이터 타입`이 변하는 것은 아님**을 알 수 있다.
