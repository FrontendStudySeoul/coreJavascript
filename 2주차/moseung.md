# 모듈과 번들러
프론트엔드 개발을 하다보면 파일들을 나누고 재사용하는데 이 기술들이 어떻게 가능한가에 대해서 얘기를 나눠보고자 합니다.
## 모듈
모듈의 사전적 정의는 다음과 같습니다.``프로그램을 구성하는 구성 요소로, 관련된 데이터와 함수를 하나로 묶은 단위``

### 1. 초기 단계: 글로벌 스코프
1995년 ~ 2000년대 초반: 자바스크립트가 처음 등장했을 때, 모든 변수가 글로벌 스코프에 위치했습니다. 이는 작은 규모의 스크립트에는 적합했지만, 코드가 커지면서 변수 충돌과 스코프 오염 문제가 발생했습니다.
### 2. 즉시 실행 함수 표현식 (IIFE)
2000년대 초반: IIFE (Immediately Invoked Function Expressions)는 코드 스코프를 제한하기 위해 사용되었습니다. 이는 모듈화의 초창기 기법으로, 전역 변수를 줄이고 독립적인 코드 블록을 만들 수 있게 했습니다.

```jsx
(function() {
  var privateVariable = 'I am private';
  console.log(privateVariable);
})();
```

### 3. 모듈 패턴
2000년대 중반: 모듈 패턴이 도입되면서 자바스크립트에서 보다 구조화된 모듈화를 할 수 있게 되었습니다. 모듈 패턴은 IIFE를 사용하여 공개(public) 인터페이스와 비공개(private) 인터페이스를 정의했습니다.

```jsx
var Module = (function() {
  var privateVariable = 'I am private';
  
  return {
    publicMethod: function() {
      console.log(privateVariable);
    }
  };
})();

Module.publicMethod(); // I am private
```

### 4. CommonJS(CJS)
2009년: 서버사이드 자바스크립트(Node.js)의 등장으로 CommonJS 모듈 시스템이 도입되었습니다. CommonJS는 require와 module.exports를 사용하여 모듈을 가져오고 내보내는 방식입니다.

```jsx
// math.js
module.exports.add = function(a, b) {
  return a + b;
};

// main.js
var math = require('./math');
console.log(math.add(2, 3)); // 5
```

### 5. Asynchronous Module Definition (AMD)
2010년: AMD는 비동기적으로 모듈을 로드할 수 있게 하는 방식으로, 주로 브라우저 환경에서 사용되었습니다. 대표적인 라이브러리로 RequireJS가 있습니다.

```jsx
// math.js
define(function() {
  return {
    add: function(a, b) {
      return a + b;
    }
  };
});

// main.js
require(['math'], function(math) {
  console.log(math.add(2, 3)); // 5
});
```

### 6. Universal Module Definition (UMD)
2011년: UMD는 CommonJS와 AMD를 모두 지원하는 모듈 형식으로, 다양한 환경에서 모듈을 사용할 수 있게 했습니다.

### 7. ECMAScript 2015 (ES6) 모듈
2015년: ECMAScript 2015(ES6) 표준에서 자바스크립트 모듈이 공식적으로 도입되었습니다. import와 export 키워드를 사용하여 모듈을 정의하고 사용할 수 있습니다.

```jsx

// math.js
export function add(a, b) {
  return a + b;
}

// main.js
import { add } from './math';
console.log(add(2, 3)); // 5
```
8. 현대의 모듈 번들러
2010년대 후반 ~ 현재: Webpack, Rollup, Parcel 등 모듈 번들러의 등장으로, 다양한 모듈 시스템을 지원하고, 개발과 배포 과정에서 모듈을 효율적으로 관리할 수 있게 되었습니다.

## 번들러
### webpack

### Rollup

### SWC
