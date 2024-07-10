# 4장. 콜백함수

- 콜백함수: 다른 코드에 함수 또는 메서드를 인자로 넘겨줘, 제어권도 함께 위임한 함수


<br/>

## this

<img width="273" alt="image" src="https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/76d7fd8d-d6a0-4940-b10f-9b11784f27b6">

- 제어권을 넘겨받는 코드에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조하게 된다.
<br/>

## 콜백 함수는 함수다

```jsx
var obj = {
  vals: [1, 2, 3],
  logValues: function(v, i) {
    console.log(this, v, i);
  },
};

obj.logValues(1, 2);
// 메서드로 호출 - { vals: [1, 2, 3], logValues: f } 1 2

[4, 5, 6].forEach(obj.logValues);
// 함수로 호출 - Window { ... } 4 0 Window { ... } 5 1 Window { ... } 6 2
```

- 어떤 함수의 인자에 객체의 메서드를 전달하더라도 이는 결국 **메서드가 아닌 “함수”다**
- 따라서 this 사용에 유의할 것

<br/>

## this에 다른 값 바인딩하기

### 1.  변수에 저장하기

```jsx
var obj1 = {
  name: 'obj1',
  func: function() {
    var self = this; // 변수에 this(현재 obj)를 담기
    return function() {
      console.log(self.name);
    };
  },
};
var callback = obj1.func(); // obj1
```
<br/>

### 2. this에 할당하기 - call

```jsx
var obj3 = { name: 'obj3' };
var callback3 = obj1.func.call(obj3); // 또는 apply
```

> [**`Function.prototype.call()`**](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
> 
> 
> this 값 및 각각 전달된 인수와 함께 함수 호출
> 
<br/>

### 3. bind 활용하기

```jsx
setTimeout(obj1.func.bind(obj2), 1500);
```

- call, apply - 함수 즉시 호출, 컨텍스트 수정
- bind - 나중에 호출될 함수를 설정

- call vs apply vs bind
    
    ```jsx
    const obj = {
      name: 'Alice',
      greet: function() {
        console.log('Hello, ' + this.name);
      }
    };
    
    const greetAlice = obj.greet.bind(obj); // 함수를 반환
    greetAlice(); // Hello, Alice
    ```
    
    ```jsx
    const obj = {
      name: 'Alice'
    };
    
    function greet() {
      console.log('Hello, ' + this.name);
    }
    
    greet.call(obj); // Hello, Alice
    ```
    
<br/>

## 콜백 지옥과 비동기 제어

<img width="473" alt="image" src="https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/0df00e1b-2081-4555-94b5-47636d7707a9">
<br/>

### 1. Promise

```jsx
function fn() {
  new Promise((resolve, reject) => {
    console.log('하나');
    resolve(); 
  })
  .then(() => {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        console.log('둘');
        resolve();
      }, 0);
    });
  })
  .then(() => {
    console.log('셋');
  })
  .catch((err) => {
    console.log('err ', err);
  });
}
```

<img width="662" alt="image" src="https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/8b9dcb93-5cc9-4a57-a8f5-bc89b7157fb8">

- `new Promise()`
    
    대기(pending): 이행하지도, 거부하지도 않은 초기 상태 
    
- `new Promise(function(resolve, reject) { **resolve**() })`
    
    이행(fulfilled): 연산이 성공적으로 완료됨. then으로 결과값을 받을 수 있음
    
- `new Promise(function(resolve, reject) { **reject**() })`
    
    거부(rejected): 연산이 실패함. catch로 값을 받을 수 있음
    
- ref
    - [https://joshua1988.github.io/web-development/javascript/promise-for-beginners/#프로미스의-3가지-상태states](https://joshua1988.github.io/web-development/javascript/promise-for-beginners/#%ED%94%84%EB%A1%9C%EB%AF%B8%EC%8A%A4%EC%9D%98-3%EA%B0%80%EC%A7%80-%EC%83%81%ED%83%9Cstates)
<br/>

### 2. Generator

- [**iteration protocols**](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Iteration_protocols)
    1. **iterable**
        
        ```jsx
        const str = 'hello';
        str[Symbol.iterator]; // [Function]
        ```
        
        - 객체의 `Symbol.iterator` 속성에 특정 형태의 함수가 들어있다면, 이를 반복 가능한 객체(iterable object) 혹은 줄여서 **iterable**이라 부르고, "해당 객체는 **iterable protocol을 만족한다**” 고 한다.
        - String, Array, TypedArray, Map, Set 등 또는 **generator 함수**를 사용해 만들 수 있다
            
          > ![스크린샷 2024-07-10 오후 1 27 02](https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/bf1b034e-5f78-4089-b5f4-f4b1020434cd)
            > obj는 iterable이 아니여서 for…of 사용이 불가능
            
        - for…of 루프와 spread 연산자, 분해대입 등 사용 가능
            
            ```jsx
            // `for...of`
            for (let c of 'hello') {
              console.log(c);
            }
            
            // spread 연산자
            const characters = [...'hello'];
            
            // 분해대입
            const [c1, c2] = 'hello';
            
            // `Array.from`은 iterable 혹은 array-like 객체를 인수로 받습니다.
            Array.from('hello');
            ```
            
    2. **iterator**
        
        ```jsx
        str[Symbol.iterator]; // [Function]
        ```
        
        - iterable에 `Symbol.iterator` (또는 `@@iterator`) 메소드 호출했을때 **반환되는 값**
        - iterator는 next라는 메소드를 가지고 있어, 이를 통해 iterable의 요소를 순회할 수 있다.
    3. generator 함수
        
        ```jsx
        function* gen() {
          // ...
        }
        ```
        
        - iterable protocol과 iterator protocol을 동시에 만족 → **iterator 생성 없이 next() 사용 가능**
        - return 키워드 사용시, 함수 종료
        - 인수를 주면, 함수가 멈췄던 부분의 yield 결과값이 인수로 출력
            
            ```jsx
            function* gen() {
              const received = yield 1;
              console.log(received); // 중지된 부분 실행 - 'hello' 출력
            }
            
            const iter = gen();
            iter.next();
            
            // 'hello'가 출력됩니다.
            iter.next('hello');
            ```
            
- **Code**
    
    ```jsx
    function* generateSequence() {
      yield 1;
      return 2; // 수행중인 이터레이터를 종료
      yield 3;
    }
    
    let generator = generateSequence();
    iter.next(); // { value: 1, done: false }
    iter.next(); // { value: 2, done: true }
    iter.next(); // { value: undefined, done: true }
    ```
    
    1. `function*` - 제너레이터 함수. **제너레이터 객체**를 반환
    2. `yield` - 제너레이터 함수를 멈추거나 다시 시작하는데 사용되는 키워드 / **함수 호출자에게 함수 실행의 제어권을 양도한다!**
    3. `next()` - yield 문을 만날때까지 실행 지속
    
    - 기타 예시
        
        ```jsx
        function* map(iterable, mapper) {
          for (let item of iterable) {
            yield mapper(item);
          }
        }
        
        const numbers = map([1,2,3], v=>v+1)
        
        numbers; // map {<suspended>}
        [...numbers]; // [2,3,4]
        ```
        
- 콜백 지옥 해결
    
    ```jsx
    
    function* fetchStarCount() {
      const starCount = {};
    
      // 1. Github에 공개되어있는 저장소 중, 언어가 JavaScript이고 별표를 가장 많이 받은 저장소를 불러온다.
      const topRepoRes = yield axios.get(`${API_URL}/search/repositories?q=language:javascript&sort=stars&per_page=1`);
    
      // 2. 위 저장소에 가장 많이 기여한 기여자 5명의 정보를 불러온다.
      const topMemberRes = yield axios.get(`${API_URL}/repos/${topRepoRes.data.items[0].full_name}/contributors?per_page=5`);
    
      // 3. 해당 기여자들이 최근에 Github에서 별표를 한 저장소를 각각 10개씩 불러온다.
      const ps = topMemberRes.data.map(user => axios.get(`${API_URL}/users/${user.login}/starred?per_page=10`));
      const starredReposRess = yield Promise.all(ps);
      const starredReposData = starredReposRess.map(r => r.data)
    
      // 4. 불러온 저장소를 모두 모아, 개수를 센 후 저장소의 이름을 개수와 함께 출력한다.
      for (let repoArr of starredReposData) {
        for (let repo of repoArr) {
          if (repo.full_name in starCount) {
            starCount[repo.full_name]++;
          } else {
            starCount[repo.full_name] = 1;
          }
        }
      }
      return starCount;
    }
    ```
    
- ref
    - https://helloworldjavascript.net/pages/260-iteration.html
    - [https://velog.io/@dev_boku/누구나-자바스크립트-제너레이터-함수가-필요한-이유는-무엇인가요](https://velog.io/@dev_boku/%EB%88%84%EA%B5%AC%EB%82%98-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%A0%9C%EB%84%88%EB%A0%88%EC%9D%B4%ED%84%B0-%ED%95%A8%EC%88%98%EA%B0%80-%ED%95%84%EC%9A%94%ED%95%9C-%EC%9D%B4%EC%9C%A0%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94)
    - https://seo-tory.tistory.com/77
<br/>

### 3. async + await

```jsx
const axios = require('axios');
const API_URL = 'https://api.github.com';

async function fetchStarCount() {
  const starCount = {};

  // 1. Github에 공개되어있는 저장소 중, 언어가 JavaScript이고 별표를 가장 많이 받은 저장소를 불러온다.
  const topRepoRes = await axios.get(`${API_URL}/search/repositories?q=language:javascript&sort=stars&per_page=1`);

  // 2. 위 저장소에 가장 많이 기여한 기여자 5명의 정보를 불러온다.
  const topMemberRes = await axios.get(`${API_URL}/repos/${topRepoRes.data.items[0].full_name}/contributors?per_page=5`);

  // 3. 해당 기여자들이 최근에 Github에서 별표를 한 저장소를 각각 10개씩 불러온다.
  const ps = topMemberRes.data.map(user => axios.get(`${API_URL}/users/${user.login}/starred?per_page=10`));
  const starredReposRess = await Promise.all(ps);
  const starredReposData = starredReposRess.map(r => r.data)

  // 4. 불러온 저장소를 모두 모아, 개수를 센 후 저장소의 이름을 개수와 함께 출력한다.
  for (let repoArr of starredReposData) {
    for (let repo of repoArr) {
      if (repo.full_name in starCount) {
        starCount[repo.full_name]++;
      } else {
        starCount[repo.full_name] = 1;
      }
    }
  }
  return starCount;
}

fetchStarCount().then(console.log);
```

<br/>
<br/>

> 😮 다들 어떤 방식을 선호하시나요?
