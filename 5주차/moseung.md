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
