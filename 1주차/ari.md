### 데이터 타입의 종류

- 기본형(Primitive Type)
  : number, boolean, null, undefined, Symbol(ES6부터 추가) 등이 있음

- 참조형(Reference Type)
  : object와 그 하위 분류로 array, function, Date, regExp, Map(ES6~), WeakMap(ES6~), Set(ES6~), WeakSet(ES6~) 등이 있음

### 기본형 vs 참조형의 차이

: 기본형은 값이 담긴 주솟값을 바로 복제, 참조형은 값이 담긴 주솟값들로 이루어진 묶음을 가리키는 주솟값을 복제함.

### 데이터 타입에 관한 배경지식

메모리와 데이터 <br>

- 비트(bit) : 0 또는 1만 표현할 수 있는 하나의 메모리 조각.

- 바이트 (byte) : 여러개의 비트를 묶은 단위. <br>

바이트로 묶게되면 검색 시간도 줄고, 표현할 수 있는 데이터가 많아지지만 낭비되는 공간이 많아짐.

메모리 용량이 작았던 시대에는 메모리 낭비를 최소화 하기위해서 데이터 타입별로 할당을 했지만,
but 비교적 메모리 용량이 커진 시대에 출현한 JS는 상대적으로 메모리 관리에 대한 압박에서 자유롭게 메모리 공간을 좀 더 넉넉하게 할당함. 개발자가 형변환을 걱정해야 하는 상황이 덜 발생하게 됨.

### 식별자와 변수

- 변수
  : 변할 수 있는 수. ('수'로 표현하는 이유는 수학용어를 차용했기 때문. 반드시 숫자인 것은 아니고 데이터를 말함)

- 식별자
  : 어떤 데이터를 식별하는 데 사용하는 이름. 즉 변수명

---

### 같이보면 좋을 내용

참조형 데이터는 가변값이라고 아마도 설명을 했을텐데요. 이를 설정에 따라 변경 불가능하게 하는 경우가 있습니다.
다양한 함수들이 있겠지만, 최근에 접해본 `Object.defineProperty`에 대해 설명하려고 합니다!

저희가 Object 중 key,value 값을 갖는 데이터를 선언할때, 대체로 아래와 같이 선언을 합니다.

```javascript
const test_object = {
  a: "test_a",
};

test_object.b = "test_b";
```

이렇게 했을 경우, 데이터를 변하지 않게하거나 추가적인 기능을 따로 구현해야할텐데요. 위에 말씀드린 데이터를 변하게 하지않게하거나, 추가적인 속성값을 넣거나 이를테면 값을 숨기게도 할 수 있는 기능을 제공하는 것이 바로 해당 메서드입니다!
Object.defineProperty()로 추가한 속성은 기본적으로 불변하며 열거 불가능합니다.

간단한 예시를 살펴보면, 아래와 같습니다.

[읽기 전용]

```javascript
const person = {};
Object.defineProperty(person, "name", {
  value: "John",
  writable: false, // 수정 불가
});
console.log(person.name); // John
person.name = "Doe"; // 무시됨
console.log(person.name); // John
```

[숨겨진 속성 정의]

```javascript
const person = {};
Object.defineProperty(person, "age", {
  value: 30,
  enumerable: false, // 열거 불가
});
console.log(Object.keys(person)); // []
console.log(person.age); // 30
```

[접근자 속성 정의하기]

```javascript
var bValue = 38;
Object.defineProperty(o, "b", {
  get() {
    return bValue;
  },
  set(newValue) {
    bValue = newValue;
  },
  enumerable: true,
  configurable: true,
});
o.b; // 38

// 'b' 속성이 o 객체에 존재하고 값은 38
// o.b를 재정의하지 않는 이상
// o.b의 값은 항상 bValue와 동일함
```

```javascript
// 두 가지를 혼용할 수는 없음
Object.defineProperty(o, "conflict", {
  value: 0x9f91102,
  get: function () {
    return 0xdeadbeef;
  },
});
// TypeError 발생
// value는 데이터 서술자에만,
// get은 접근자 서술자에만 나타날 수 있음
```

추가적인 내용은.. MDN 에서 https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty 확인해주세요 ~~

### 실제 사용 예제

해당 예제는 'javascript 상태관리 시스템 만들기' 내용이고, 준일님 블로그를 보며 공부하였습니다.

> Observer pattern은 객체의 상태 변화를 관찰하는 옵저버들의 목록을 객체에 등록해서 상태 변화가 발생할 때마다 메서드 등을 통해 객체가 직접 목록의 각 옵저버들에게 통지하도록 하는 디자인 패턴.

```javascript

// observable은 observe 에서 사용된다. observable(상태 변화 감지 객체)에 변화가 생기면, observe에 등록된 함수가 실행된다.

// state는 reducer 을 실행하고 반환되는 state 값을 받아서 observe에 등록 
export const observerable = (state) => {
  const stateKeys = Object.keys(state);

  for (const key of stateKeys) {
    let _value = state[key];

    const observers = new Set();

    Object.defineProperty(state, key, {
      get() {
        if (currentObserver) observers.add(currentObserver);
        return _value;
      },
      set(value) {
        if (_value === value) return;

        _value = value;
        observers.forEach((observer) => observer());
      },
    });
  }

  return state;
};

```

```javascript

export const createStore = (reducer) => {
  const state = observerable(reducer());

  const frozenState = {};

  Object.keys(state).forEach((key) => {
    Object.defineProperty(frozenState, key, {
      get: () => state[key],
    });
  });

  const dispatch = (action) => {
    const newState = reducer(state, action);

    for (const [key, value] of Object.entries(newState)) {
      if (!state[key]) continue;
      state[key] = value;
    }
  };

  return {
    dispatch,
    getState: () => frozenState,
  };
};

```

createStore 에는 state 등록, state 변경, state 조회 이렇게 3가지 기능이 수행이된다.
- state 등록(subscribe)은 reducer를 실행하고 반환된 state 값을 observerable로 만들어야한다.
- state 변경(dispatch)는 state 값을 변경해야 한다.
- state 조회(getState)는 state를 직접 반환하면 좋겠지만, state에는 getter, setter가 모두 포함되고 있기 때문에 직접 참조하여 수정이 가능하므로 getter만 가능하도록 frozenState를 따로 만드는 것이다. 이건 다른 방식도 있긴한데, 블로그에서 따로 확인하면 될 것 같다.
