## 05. 불변 객체

- 불변 객체 : 객체가 생성된 이후 내부 상태를 변경할 수 없는 객체.
    - 라이브러리나 프레임워크에서뿐만 아니라 함수형 프로그래밍, 디자인 패턴 등에서도 매우 중요한 기초 개념 → **왜?**
    
    ```
    1. 상태 관리의 용이성 : 객체 상태 변경할 수 없기에 프로그램 상태를 추적하고 관리하기 쉬움
    2. 동시성 문제 해결 : 여러 스레드가 동시 접근해도 상태 변경X
    3. 함수형 프로그래밍의 핵심 : 상태 변경을 피하고 순수 함수를 사용하는 것이 중요 -> 예측 가능한 코드 작성
    4. 참조 투명성 : 동일한 입력에 대해 항상 동일한 출력을 반환하는 성질
    5. 설계의 단순화 : 객체 상태 변화에 따른 부작용을 최소화하여 설계가 단순, 오류가능성 줄여줌
    6. 프레임 워크의 사용 : UI업데이트가 빈번하게 일어남 -> 불변 객체를 사용하면 상태 변화 추적이 쉬워지고,
    효율적인 리렌더링이 가능해짐. -> 
    
    **[React에서 불변성이 중요한 이유 👀]
    https://medium.com/@choi-hyunho/react-%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4%EC%84%9C-state-%EB%A5%BC-%EB%B3%80%EA%B2%BD-%ED%96%88%EC%A7%80%EB%A7%8C-%EB%A6%AC%EC%95%A1%ED%8A%B8%EA%B0%80-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EA%B0%90%EC%A7%80%ED%95%98%EC%A7%80-%EB%AA%BB%ED%95%98%EC%97%AC-%EB%B3%80%ED%99%94%EA%B0%80-%EC%9D%BC%EC%96%B4%EB%82%98%EC%A7%80-%EC%95%8A%EC%95%84-%EC%9E%90%EB%A3%8C%EB%A5%BC-%EC%B0%BE%EC%95%84%EB%B3%B4%EB%8B%A4%EA%B0%80-5bd91e150f1**
    
    ```
    

- 참조형 데이터의 ‘가변’은 데이터 자체가 아닌 **내부 프로퍼티를 변경할 때만 성립**한다(mean, 새로운 객체를 할당함으로 값을 직접 변경하는 경우가 아닌 obj.name으로 프로퍼티에 접근할 때의 경우).
    - 그렇기에, 데이터 자체를 변경하고자 하면 기본형 데이터와 마찬가지로 **기존 데이터는 변하지 않는다**.
    - 그렇기에, 내부 프로퍼티를 변경할 필요가 있을 때마다 
        1) 매번 새로운 객체를 만들어 재할당(spread operator, Object.assign)하기로 규칙을 정하거나 
        2) 자동으로 새로운 객체를 만드는 도구를 활용(Immutable.js, Immer.js, immutability-helper, baobab.js)한다면 객체 역시 불변성을 확보할 수 있을 것이다.

- 객체의 가변성에 따른 문제점
    
    ```jsx
    var user = {
    	name: 'Shin',
    	gender: 'female'
    }
    
    var changeName = (user, userName) => {
    	var newUser = user;
    	newUser.name = newName;
    	return newUser;
    }
    
    var user2 = changeName(user, 'Judy')
    
    if (user !== user2) {
    	console.log('changed')
    }
    
    console.log(user.name, user2.name) // Judy Judy 
    // -> 원본 객체가 바뀌는 문제 -> 1depth에 같은 주소를 바라보니까
    console.log(user === user2) // true
    ```
    

- 책에서 제시한 기존 정보를 복사해서 새로운 객체를 반환하는 함수 → 가변성으로 인한 문제를 해결해주는 함수
    
    ```jsx
    var copyObject = (target) => {
    	var result = {};
    	for (var prop in target) {
    		result[prop] = target[prop];
    	}
    	return result;
    }
    
    var user2 = copyObject(user)
    user2.name = 'Judy'
    
    if (user !== user2) {
    	console.log('changed')  // changed
    }
    
    console.log(user.name, user2.name) // Shin Judy 
    console.log(user === user2) // false
    ```
    
    → 프로토타입 체이닝 상의 모든 프로퍼티를 복사하는 점, getter/setter는 복사하지 않는 점, 얕은 복사만을 수행한다는 점이 아쉬움
    
- 얕은 복사 : 바로 아래 단계의 값만 복사, 중첩된 객체에서 참조형 데이터가 저장된 프로퍼티 복사할 때 그 주솟값만 복사한다는 의미. 그러면 프로퍼티에 대해 원본과 사본이 모두 동일한 참조형 데이터의 주소를 가리키게 되어 사본을 바꾸면 원본도 바뀌고 원본을 바꾸면 사본도 바뀜.
    - 참조형 데이터가 있을 때마다 재귀적으로 수행해야만 비로소 깊은 복사가 됨
    
- 책에서 제시한 얕은 복사의 문제를 해결해주는 함수
    
    ```jsx
    var copyObjectDeep = (target) => {
    	var result = {};
    	if (typeof target === 'object' && target !== null) {
    		for(var prop in target) {
    			result[prop] = copyObjectDeep(target[prop]);
    		}
    	}	else {
    		result = target;
    	}
    	return result;
    }
    ```
    
    - **hasOwnProperty**메서드 활용해 프로토타입 체이닝을 통해 상속된 프로퍼티를 복사하지 않게끔 할 수 있다.
    
    ```jsx
    var copyObjectDeep = (target) => {
        var result = {};
        if (typeof target === 'object' && target !== null) {
            for (var prop in target) {
                if (**target.hasOwnProperty(prop)**) {
                    result[prop] = copyObjectDeep(target[prop]);
                }
            }
        } else {
            result = target;
        }
        return result;
    }
    
    // 프로토타입 체인을 통해 상속된 속성을 가지는 객체 생성
    var proto = { protoProp: 'proto' };
    var user = Object.create(proto); // user객체는 proto객체를 프로토타입으로 가짐 -> 상속받음
    user.name = 'Shin';
    user.age = 30;
    
    // 깊은 복사 수행
    var user2 = copyObjectDeep(user);
    user2.name = 'Judy';
    
    // 결과 출력
    console.log(user.name, user2.name); // Shin Judy
    console.log(user.protoProp); // proto
    console.log(user2.protoProp); // undefined, 상속된 프로퍼티는 복사되지 않음
    console.log(user === user2); // false
    
    ```
    
    - JSON문법으로 표현된 문자열로 전환했다가 다시 JSON객체로 바꾸는 방법도 있는데 __proto__나 getter/setter등과 같이 JSON으로 변경할 수 없는 프로퍼티들은 무시함. 그래서 HttpRequest로 받은 데이터를 저장한 객체를 복사할 때 등 순수한 정보만 다룰 때 활용하기 좋음
    
    
    > 💡 그런데, 객체의 키가 존재하는지 확인할 때는 hasOwnProperty를 지양합시다. 
    > 저는 Reflect.has({b:1}, ‘b’)을 사용하는 방법을 추천받았어요.
    >
    > [hasOwnProperty를 사용하면 안되는 이유](https://velog.io/@jay/be-carefule-use-hasownproperty)
    > 
    > [hasOwnProperty 와 hasOwn](https://velog.io/@deepthink/hasOwnProperty-%EC%99%80-hasOwn)
    
    </aside>
    

## 06. Undefined & null

- undefined : 값이 존재하지 않을 때 자바스크립트 엔진이 자동으로 부여하는 경우도 존재
    - 사용자가 응당 어떤 값을 지정할 것이라고 예상되는 상황임에도 실제로 그렇게 하지 않았을 때 undefined반환
    
    ```jsx
    var a;
    console.log(a);  // (1) undefined. 값을 대입하지 않은 변수(데이터 영역의 메모리 주소를 지정하지 않은 식별자)에 접근
    
    var obj = { a: 1};
    console.log(obj.a); // 1
    console.log(obj.b); // (2) 객체 내부의 존재하지 않는 프로퍼티에 접근
    console.log(b);     // c.f) ReferenceError: b is not defined
    
    var func = function() {  };
    var c = func();     // (3) 반환(return) 값이 없으면 Undefined를 반환한 것으로 간주
    console.log(c);     // undefined
    ```
    
- 단순히 비어있는 요소가 undefined가 아님. 비어있는 요소와 undefined를 할당한 요소는 다른 요소임
    
    ```jsx
    var arr1 = [];
    arr1.length = 3;
    console.log(arr1); // [empty * 3]
    
    var arr2 = new Array(3);
    console.log(arr2); // [empty * 3]
    
    var arr3 = [undefined, undefined, undefined];
    console.log(arr3); // [undefined, undefined, undefined]
    ```
    
- 비어있는 요소는 순회와 관련된 많은 배열 메서드들의 순회 대상에서 제외된다. forEach, map, filter, reduce 등에서 어떠한 처리도 하지 않고 건너뛴다.
    - 배열은 객체와 마찬가지로 특정 인덱스에 값을 지정할 때 비로소 빈 공간을 확보하고 인덱스를 이름으로 지전하고 데이터의 주솟값을 저장하는 등의 동작을 한다. 즉, 값이 지정되지 않은 인덱스는 아직은 존재하지 않는 프로퍼티에 지나지 않는 것.
    
    ```jsx
    var arr1 = [undefined, 1];
    var arr2 = [];
    arr2[1] = 1;
    
    arr1.forEach((v, i) => console.log(v, i));  
    // undefined 0
    // 1 1
    arr2.forEach((v, i) => console.log(v, i));  // 1 1
    
    arr1.map((v, i) => { return v + i })    // [NaN, 2]
    arr2.map((v, i) => { return v + i })    // [비어 있음, 2]
    
    arr1.filter((v) => { return !v })    // [undefined]
    arr2.filter((v) => { return !v })    // []
    
    arr1.reduce((p, c, i) => { return p + c + i }, '')  // undefined011
    arr2.reduce((p, c, i) => { return p + c + i }, '')  // 11
    ```
    
- **💡 undefined의 의미 재정리**
    1. 사용자가 명시적으로 부여한 경우
        
        : 그 자체로 값임. 하나의 값으로 동작하기에 프로퍼티나 배열의 요소는 고유의 키값이 실존하게 되고, 순회의 대상이 될 수 있음
        
    2. 비어있는 요소에 접근하려 할 때 반환되는 경우
        
        : 해당 프로퍼티 내지 배열의 키값 자체가 존재하지 않음을 의미. 
        
- 자바스크립트 엔진이 반환하는 경우는 우리의 통제 범위를 벗어남. 그러니 직접 undefined를 할당하지 않기만 하면 됨 → 대신에 null을 쓰자
    
    ex. var vs let, const
    
- null의 버그 → typeof null이 object라는 점
    - 어떤 변수의 값이 null인지 여부를 판별하기 위해선 typeof대신 === 연산자를 쓰자
    
    ```jsx
    var n = null;
    console.log(typeof n); // object
    
    console.log(n == undefined) // true
    console.log(n == null)      // true
    console.log(n === undefined)// false
    console.log(n === null)     // true
    ```