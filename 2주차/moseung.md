# 모듈과 번들러
프론트엔드 개발을 하다보면 파일들을 나누고 재사용하는데 이 기술들이 어떻게 가능한가에 대해서 얘기를 나눠보고자 합니다.
## 모듈
모듈의 사전적 정의는 다음과 같습니다.``프로그램을 구성하는 구성 요소로, 관련된 데이터와 함수를 하나로 묶은 단위``<br>
우리가 왜 모듈을 사용하고 이렇게 분리하게 됐냐를 생각해보기 위해 근원을 찾고자 이 모듈이 어떻게 시작됐는지에 대해서 먼저 얘기해보려고 합니다.

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
### 8. 현대의 모듈 번들러
2010년대 후반 ~ 현재: Webpack, Rollup, Parcel 등 모듈 번들러의 등장으로, 다양한 모듈 시스템을 지원하고, 개발과 배포 과정에서 모듈을 효율적으로 관리할 수 있게 되었습니다.
<br> 모듈을 잘 보면 우리프로젝트에서 아래 사진처럼 됩니다.
<br>
![image](https://github.com/FrontendStudySeoul/coreJavascript/assets/103626175/51c6985d-1f18-4a56-89a6-2d7688d7f44b)


## 번들러
번들러는 여러개로 모듈화 된 파일을 하나로 합칩니다. 왜냐하면 위에서 모듈을 분리하는 기술이 있어도 브라우저는 이를 읽을 수 없기 때문입니다. <br>
웹팩은 애플리케이션의 모든 모듈과 그 의존 관계를 분석하여 모듈 그래프를 생성합니다. 이 과정은 애플리케이션의 규모가 커질수록 시간이 많이 소요됩니다.

### webpack

- 모듈 번들러: 웹팩은 자바스크립트 모듈을 하나의 번들 파일로 묶어주는 도구입니다. 이를 통해 파일 의존성을 관리하고 번들 크기를 최적화할 수 있습니다.
- 트리 쉐이킹(Tree Shaking): 사용되지 않는 코드를 제거하여 최종 번들의 크기를 줄입니다.
- 코드 스플리팅(Code Splitting): 애플리케이션의 코드가 필요할 때만 로드되도록 분할하여 초기 로드 시간을 단축합니다.
- 플러그인 시스템: 다양한 플러그인을 통해 기능을 확장할 수 있습니다. 예를 들어, 스타일 시트 로더, 이미지 최적화, 환경 변수 설정 등이 가능합니다.
  
### Rollup
- 모듈 번들러: Rollup은 특히 라이브러리 개발에 적합한 모듈 번들러입니다. ES6 모듈을 기반으로 한 번들링을 지원합니다. (다른것들에 비해서 더 최적화됨.)
- 트리 쉐이킹: Rollup은 초기부터 트리 쉐이킹을 지원하여 사용되지 않는 코드를 효과적으로 제거합니다.
- 플러그인 시스템: 다양한 플러그인을 통해 기능을 확장할 수 있습니다.
- 단일 파일 출력: 라이브러리는 종종 단일 파일로 배포되는 경우가 많습니다. Rollup은 단일 파일 번들링에 최적화되어 있어, 라이브러리 배포에 적합합니다.
  
### SWC
- 자바스크립트/타입스크립트 컴파일러: SWC는 자바스크립트와 타입스크립트를 컴파일하여 더 빠르고 최적화된 코드를 생성합니다.
- 초고속 빌드: Rust로 작성되어 매우 빠른 컴파일 속도를 자랑합니다.
- 바벨 대안: 바벨(Babel)과 유사한 기능을 제공하지만, 훨씬 더 빠르게 작동합니다.
- 모던 자바스크립트 지원: 최신 자바스크립트 문법과 기능을 지원하여 모던 웹 애플리케이션 개발에 적합합니다.
- 작업 분할: SWC는 코드 분석, 변환, 번들링 등의 작업을 여러 스레드(멀티스레드)로 분할하여 동시에 수행할 수 있습니다. 이를 통해 전체 처리 시간을 단축할 수 있습니다. 그리고 멀티스레드 환경에서의 동시성을 보장하는 방법도 잘 제공합니다.


