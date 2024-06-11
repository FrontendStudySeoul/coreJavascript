# 03. 변수 선언과 데이터 할당
## 1-3-1 변수 선언
<img width="687" alt="image" src="https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/620d8f81-ad1c-4eba-bb3f-b94078e11ed8">
- **변수**: 변경 가능한 데이터가 담길 수 있는 공간
- **식별자**: 위 사진의 result, 메모리 공간의 이름. 식별자는 변수의 값이 아닌 메모리 주소를 저장하고 있다
> 사용자가 result에 접근 -> 메모리에서 result라는 이름을 가진 주소를  검색 -> 공간에 담긴 데이터(30) 반환

## 1-3-2 데이터 할당
아래의 코드가 실행되면 다음과 같은 과정을 거친다
```javascript
var a;
a = 'abc';
```
1. 변수 영역에서 빈 공간 확보
2. 확보한 공간의 식별자를 a로 지정
3. 데이터 영역의 빈 공간에 'abc' 저장
4. 변수 영역에서 a라는 식별자 검색
5. 3에서 저장한 문자열의 주소를 2공간에 대입

![image](https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/e22e21ae-8ede-4f4e-8721-7103cfef0ea8)

새로운 문자를 할당하면, 별도의 공간을 **새로** 만들어 저장하고, 그 주소를 연결한다.
이렇게하는 이유는, 문자열은 숫자형 데이터와 달리 **특별히 정해진 규격이 없기 때문**이다.

> 더이상 사용하지 않는 데이터는 "가비지 컬렉터"의 수거 대상이 된다

또, 자바스크립트는 동일한 값을 바라보는 여러 식별자가 있으면, 같은 주소를 바라보도록 설계되어 있다.
즉 같은 값으로 N개의 변수 공간을 낭비하지 않는다.

<br/><br/>

# 04. 기본형 데이터와 참조형 데이터
## 1-4-1 불변값
- **변수**: 변수 영역 메모리를 다른 데이터로 변경 가능
- **상수**: 변수 영역 메모리를 다른 데이터로 변경 불가능

- **불변값**: 기본형 데이터 - 숫자, 문자열, boolean, null, undefined, Symbol. 절대 변하지 않는 값
```jsx
let a = 'abc;
a = a + 'def';

let b = 5;
let c = 5;
b = 7;
```
![image](https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/b5d6625c-260e-43d2-9993-d1e09c099502)
위 예시에서 값을 7로 바꾸면 
1. 기존에 만들어놓은 데이터 중 7이 있는지 확인하고
2. 없으면 새로 만들어(@5003) 저장

변경은 새로 만드는 동작을 통해서만 이루어지는 것, 이를 "불변성"이라 한다!

## 1-4-2 가변값
- 가변값 - 참조형 데이터

```javascript
var obj1 = {
	a: 1,
	b: 'bbb'
}
```
![image](https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/d47357f6-610f-4caa-bb8c-917e2df4dc77)

1. 변수의 빈 공간 확보(@1002) - obj1
2. 객체 내부의 프로퍼티를 저장하기 위한 별도의 변수 영역을 마련(@5001)하고, 주소를 저장
3. **@7103과 @7104에 각 프로퍼티의 이름과 값을 가리키는 주소를 저장**
4. 데이터 영역에 1과 'bbb' 저장

> 3이 기본형 데이터와의 차이점이다. 이 영역으로 인해 불변하지 않은 특징을 가진다


```javascript
obj1.a = 2;
```
프로퍼티 a의 값이 바뀌어도 기존 @1002의 주소는 변하지 않고 객체 내부(@7103)만 변한것을 확인할 수 있다.

<br/>

- 중첩객체: 참조형 데이터의 프로퍼티에 다시 참조형 데이터를 할당하는 경우
```js
var obj = {
	x: 3,
	arr: [3,4,5]
}
```
![image](https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/71890a13-2747-4bd4-9424-617bbd8049bc)


> obj.arr[1]을 검색한다면..?

이때, 재할당 명령을 내리면 아래처럼 변하게 된다.
```js
obj.arr = 'str';
```
![image](https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/91bfcaf2-df29-4f78-886b-374da262327a)

위 사진에서 파란색 영역이 모두 GC(Garbage Collector)의 수거 대상이 된다.

## 1-4-3. 변수 복사 비교
```jsx
let a = 10;
let b = a;

let obj1 = { c: 10, d: 'ddd' };
let obj2 = obj1;
```

![image](https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/bed3bb45-8340-4ffb-92f1-46dc0f5ea6a7)

단순 복사는 기본형/참조형 모두 같은 주소를 바라보고 있는것을 알 수 있다.

👉 값을 할당해보자!
```jsx
b = 15;
obj2.c = 20;
```

![image](https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/4d12122d-3bb8-4f20-899a-7fe0cd7fb1d3)

결과는 
```js
a !== b // 기본형
obj1 === obj2
```
- 기본형 - 주솟값을 복사하는 과정이 한 번
- 참조형 - 기본형보다 한단계를 더 거침. 여전히 같은 값(@5002)를 바라본다

> **사실은 어떤 데이터 타입이든 변수에 할당하기 위해서는 주솟값을 복사해야한다.**
> 참조형은 한번 더 거친다는 차이밖에 없다.


👉 아예 객체 값 자체를 변경하면 어떻게 될까?
```js
obj2 = {c: 20, d: 'ddd'}
```
![image](https://github.com/FrontendStudySeoul/coreJavascript/assets/70925962/1317dd31-df71-4a7c-bfbc-33320f655e93)


새로운 변수 영역이 생기게 되므로, 값이 달라지는 것을 확인할 수 있다. (@5006)

결과는
```js
obj1 !== obj2
```
> 즉, 참조형 데이터가 '가변값'이라고 설명할 때의 '가변'은 **내부의 프로퍼티를 변경할 때만 성립**한다.


# Ref.
https://velog.io/@modolee/core-javascript-01




