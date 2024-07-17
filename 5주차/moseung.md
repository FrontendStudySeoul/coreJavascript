 # 7. 클래스
## 7.3.2 클래스가 구체적인 데이터를 지니지 않게 하는 방법
가장 쉬운방법은 일단 만들고나서 프로퍼티들을 일일이 지우고 더는 추가적인 할당을 하지 못하게 하는것입니다.
```tsx
delete Square.prototype.width;
delete Square.prototype.height;
Object.freeze(Square.prototype)
```
다만 위코드들은 명령적이므로 여러 프로퍼티들이 들어올 때 동작할 수 있는 우아한 코드로 변경해봅시다.
```tsx
var extendClass1 = function (Superclass, SubClass, subMethods) {
	SubClass.prototype = new SuperClass();
    for (var prop in SubClass.prototype) {
    	if (SubClass.prototype.hasOwnProperty(prop)) {
        	delete SubClass.prototype[prop];
        }
    }
	if (subMethods) {
		for (var method in subMethods) {
    		SubClass.prototype[method] = subMethods[method];
    	}
	}
	Object.freeze(SubClass.prototype);
	return SubClass;
};

var Square = extendClass1(Rectangle, function (width) {
	Rectangle.call(this, width, width);
});
```
extendClass라는 함수로 슈퍼클래스,서브클래스를 받아서 프로토타입에 존재하는 메서드를 모두 지워주고, 더이상 추가할 수 없게하며, 만약 서브메서드가 있다면 그대로 서브클래스에 값을 추가해줍니다.

<br>
두 번째로 다른 방안도 있습니다. subClass의 prototype SuperClass의 인스턴스를 할당하는 대신 아무런 프로퍼티를 생성하지 않는 빈 생성자함수의 prototype이 SuperClass를 바라본 후,subClass는 빈 생성자함수의 인스턴스를 할당합니다.
<br>

```tsx
var extendClass2 = (function () {
	var Bridge = fuction () {};
    return function (SuperClass, SubClass, subMethods) {
    	Bridge.prototype = SuperClass.prototype;
        SubClass.prototype = new Bridge();
        if (subMethods) {
        	for (var method in subMethods) {
            	SubClass.prototype[method] = subMethods[method];
            }
        }
        Object.freeze(SubClass.prototype);
        return SubClass;
    };
})();
```
이처럼 코드를 작성하면 위 함수와는 다르게 SubClass의 프로토타입이 SuperClass의 인스턴스를 상속하지 않기때문에 삭제하지 않아도 됩니다.
<br>
Es5에서 제공하는 Object.create로도 가능합니다.
```tsx
Square.prototype[method] = Object.freeze(Object.create(Reactangle,prototype));
```


### Object.create
Object.create함수를 잠깐 설명드립니다.<br>
- value : 실제 해당 키값에 들어갈 value를 설정
- writable: 해당 키값에 들어갈 value에 대한 수정 여부
- enumerable: 속성이 열거 가능한지 여부를 제어합니다.
- configurable: 해당 객체에 값들을 새로 정의하거나 삭제(delete) 불가능해집니다.
```tsx
var person = {
    isHuman: false,
    printIntroduction: function () {
        console.log(`My name is ${this.name}. Am I human? ${this.isHuman}`);
    }
};

var me = Object.create(person, {
    name: {
        value: 'Matthew',
        writable: true,
        enumerable: true,
        configurable: true
    },
    isHuman: {
        value: true,
        writable: true,
        enumerable: true,
        configurable: true
    }
});
```

## 7-3-3 constructor 복구하기
기본적인 상속에는 성공했지만 subClass의 인스턴스는 여전히 SuperClass를 가리키고 있습니다. 그래서 이부분을 다시 정상화 시켜줍니다.
```tsx
var extendClass2 = (function () {
	var Bridge = fuction () {};
    return function (SuperClass, SubClass, subMethods) {
    	Bridge.prototype = SuperClass.prototype;
        SubClass.prototype = new Bridge();
	//이부분이 달라졌어요.
	**SubClass.prototype.constructor = SubClass;**
        if (subMethods) {
        	for (var method in subMethods) {
            	SubClass.prototype[method] = subMethods[method];
            }
        }
        Object.freeze(SubClass.prototype);
        return SubClass;
    };
})();
```

## 7-3-4 상위 클래스에의 접근 수단 제공
때로 하위 클래스의 메서드에서 상위 클래스의 메서드 실행 결과를 바탕으로 작업을 수행하고 싶을 때가 있습니다.
<br>
이런 별도의 수단인 super를 흉내 내보고자 합니다.
```tsx
var extendClass2 = (function () {
	var Bridge = fuction () {};
    return function (SuperClass, SubClass, subMethods) {
    	Bridge.prototype = SuperClass.prototype;
        SubClass.prototype = new Bridge();
	SubClass.prototype.constructor = SubClass;
	SubClass.prototype.super = function(propName){
	var self = this;//  this 고정
	if(!propName) return function(){
SuperClass.apply(serlf,arguments);
}
var prop = SuperClass.prototype[propName];
if(typeof prop !== "function") return prop;
return function(){
return prop.apply(self,arguments);
}
}
        if (subMethods) {
        	for (var method in subMethods) {
            	SubClass.prototype[method] = subMethods[method];
            }
        }
        Object.freeze(SubClass.prototype);
        return SubClass;
    };
})();
```
## 7-4 Es6의 클래스 및 클래스 상속
Es6부터 본격적으로 클래스가 도입됐습니다.ES5와 ES6에서의 문법 비교를 보면 더 와닿을 것 같습니다.
```tsx
var ES5 = function (name) {
	this.name = name;
};
ES5.staticMehod = function () {
	return this.name + 'staticMethod';
};
ES5.prototype.method = function () {
	return this.name + 'method';
};
var es5Instance = new ES5('es5'));
console.log(ES5.staticMethod()); // es5 staticMethod
console.log(es5Instace.method()); // es5 method

var ES6 = class {
	constructor (name) {
    	this.name = name;
    }
    static staticMethod() {
    	return this.name + 'staticMethod';
    }
    method () {
    	return this.name + 'method';;
    }
};
var es6Instance = new ES6('es6');
console.log(ES6.staticMethod()); // es6 staticMethod
console.log(es6Instace.method()); // es6 method
```

## 번외
사실 이처럼 배워보니 자바스크립트에서 클래스를 구현하기 위해서 부족한점이 많은 것 같습니다. 대부분의 객체지향에서 제공하는 정적언어 즉 컴파일 기능은 자바스크립트에서 제공하지 않으며 다양한 접근 제어자가 없습니다.
<br>
물론 ES2020(ES11)에서 나온 #문법을 사용하면 Private을 구현할 순 있습니다.
```tsx
class Person {
    #name;
    #age;

    constructor(name, age) {
        this.#name = name;
        this.#age = age;
    }

    getName() {
        return this.#name;
    }

    getAge() {
        return this.#age;
    }
}

```
<br>
요즘 프론트엔드 개발자들이 많이 사용하는 타입스크립트에서는 이런 부족한점이 어느정도 해결돼서 나오는데 같이 공유하려고 합니다.
<br>
1. 타입 안전성 부족
- 자바스크립트는 동적 타입 언어로, 컴파일 시 타입 체크를 하지 않습니다. 이로 인해 런타임 오류가 발생할 가능성이 높아집니다.
2. 접근 제어자 부족
- 자바스크립트는 기본적으로 클래스 속성에 대한 접근 제어자를 제공하지 않습니다. 모든 속성이 기본적으로 공개(public)되어 있어 캡슐화가 어려울 수 있습니다.
3. 인터페이스와 추상 클래스의 부재
- 자바스크립트에는 인터페이스와 추상 클래스와 같은 개념이 없습니다. 이는 클래스 간 계약을 정의하거나 강제하는 데 어려움을 줍니다.
4. 컴파일 타임 에러 확인 부족
- 자바스크립트는 컴파일 타임에 에러를 확인할 수 없기 때문에 런타임에 많은 에러가 발생할 수 있습니다.

위와 같은 문제들을 강력한 정적타입언어로 컴파일 시 에러를 잡을 수 있습니다.


