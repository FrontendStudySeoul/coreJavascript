# 5. 클로저
## 5. 클로저의 의미 및 원리 이해
**어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상** 이 문장을 끝날때까지 기억해주시길 바랍니다.
<br>
일반적인 함수가 있습니다. 앞서 2장에서 배운 lexical환경에 따라 아래 함수를 생각해봅시다.
```tsx
var outer = function(){
  var a = 1;
  var inner = function(){
    return ++a;
  }
  return inner()
}
var outer2 = outer();
console.log(outer2)
```
전역 객체 -> outer함수 -> inner함수순으로 실행될것이며 outer은 전역 객체를, inner는 outer를 outerEnvironmentReference로 가지게 될 것입니다.<br>
그리고 함수의 실행 컨텍스트가 종료된 시점에는 a 변수를 참조하는 대상이 없어지고 참조 대상이 없어지면 이는 GC에 의해서 메모리에서 소멸하게 됩니다.<br>
잠깐 가비지 컬렉션에 대해서 궁금한점이 있으시지 않은가요? 아래는 가비지 컬렉션에 의해 명시적인 조건이 아니면 사라지지 않는 리스트들입니다.
- 현재 함수의 지역 변수와 매개변수
- 중첩 함수의 체인에 있는 함수에서 사용되는 변수와 매개변수
- 전역 변수
특히 전역 변수는 자바스크립트 실행 환경 내에서 항상 도달 가능한 상태로 유지되기 때문에 큰 데이터를 담아두는건 좋은 설계는 아닙니다.(eventListner도 포함입니다!)<br>
<br>
비슷한 함수가 있습니다. 혹시 이상한점이 보이시나요?

```tsx
var outer = function(){
  var a = 1;
  var inner = function(){
    return ++a;
  }
  return inner;
}

var outer2 = outer();
console.log(outer2());
console.log(outer2());
```

inner 함수의 실행 컨텍스트에서 environmentRecord는 수집할것이 없고 outerEnvironmentReference로는 함수가 선언된 위치의 LexicalEnvironment가 참조 복사 됩니다.<br>
<img width="1006" alt="image" src="https://github.com/FrontendStudySeoul/coreJavascript/assets/103626175/8d7a8272-79ee-44ac-9d20-b0e64e9c202e">
<br>
그래서 **어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상**은 다시 말하면 **외부 함수의 EnvironmentReference가 특정 상황에서는 가비지 컬렉팅되지 않는 현상**와도 같습니다.

## 5.2 클로저와 메모리 관리
요즘은 개발자의 의도와 다르게 메모리가 누수되는일이 거의 없어졌기에 클로저를 잘 파악해서 관리하면 됩니다.<br>
관리방법은 정말 간단한데 의도적으로 함수의지역변수를 사용하므로 이 지역변수의 참조를 없애주면 됩니다.
```tsx
var outer = function(){
  var a = 1;
  var inner = function(){
    return ++a;
  }
  return inner;
}

var outer2 = outer();
console.log(outer2());
console.log(outer2());
outer2= null;//참조 끊기
```
## 5.3 클로저 활용 사례
클로저의 실제 활용 사례를 확인해봅시다.
### 5.3.1 콜백 함수 내부에서 외부 데이터를 사용하고자 할 떄
아래처럼 콜백 함수 내부에서 외부 데이터를 클로저를 활용해서 사용할 수 있습니다. 다만 아래에는 문제가 있는데 뭐가 문제일까요?
```tsx
var fruits = ["apple","banana","peach"];
var $ul = document.createElement("ul");

var alertFruit = function(fruit:string){
  alert("your choice is" + fruit)
}
fruits.forEach(function(fruit){
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListner("click",alertFruit);
  $ul.appendChild($li);
})
document.bodoy.appendChild($ul);
alertFruit(fruits[1]);
```
저는 처음에 이렇게 실행하면 fruit로 받는 인자값이 undefined여서 출력이 안될줄 알았는데, click의 이벤트 객체가 출력됩니다.
<br>
이는 addEventListner로 전달된 두번째인자는 콜백함수이고 콜백함수의 첫번째 인자로 이벹트객체가 들어가기때문에 발생하는 문제입니다. 아래 코드로 해결할 수 있습니다.
```tsx
//1
$li.addEventListner("click",alertFruit.bind(null,fruit));
//2
$li.addEventListeer("click",(e)=>{
  alertFruit(fruit);
})
//3 책에서는 고차함수로 풀었네요.
var alertFruit = function(fruit:string){
  return function(){
    alert("your choice is" + fruit)
  }
}
fruits.forEach(function(fruit){
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListner("click",alertFruit(fruit));
  $ul.appendChild($li);
})
```
저는 두번째 방법을 선호하는데 이렇게하면 이벤트객체의 특성도 추후 필요할 시 활용할 수 있기 때문입니다.
