> 코드리뷰할 때 이런 것도 있어요~! 하면서 아는척 하고 싶어서,
최근 3년에 걸쳐 나온 새로운 version 중에 몇가지 문법 들고왔어요
> 

<br/>

- **`Array.at()`, `string.at()`**
    
    ```jsx
    const fruits = ["Banana", "Orange", "Apple", "Mango"];
    fruits.at(2);    // "Apple"
    fruits[2];       // "Apple"
    
    fruits.at(-1)    // "Mango"
    fruits[-1]       // undefined
    ```
    
    다른 언어들은 음수 인덱싱을 허용하는데, 자바스크립트는 안됨.
    
    at()연산자를 통해 실행할 수 있게 됨.
    
<br/>

- **`Array.findLast()`, `Array.findLastIndex()`**
    
    ```jsx
    const temp = [27, 28, 30, 40,41, 42, 35, 30];
    let high1 = temp.find(x => x > 40);        // 41
    let high2 = temp.findLast(x => x > 40);    // 42
    ```
    
    - find → 앞에서부터, findLast → 뒤에서부터

<br/>

- **`Array.toReversed()` , `Array.toSorted()` , `Array.toSpliced()`**
    
    ```jsx
    // reverse()
    const months = ["Jan", "Feb", "Mar", "Apr"];
    const reversed = months.reverse();
    
    console.log(months); // 출력: ["Apr", "Mar", "Feb", "Jan"]
    console.log(reversed); // 출력: ["Apr", "Mar", "Feb", "Jan"]
    
    // toReversed()
    const months = ["Jan", "Feb", "Mar", "Apr"];
    const reversed = months.toReversed();
    
    console.log(months); // 출력: ["Jan", "Feb", "Mar", "Apr"]
    console.log(reversed); // 출력: ["Apr", "Mar", "Feb", "Jan"]
    ```
    
    ```jsx
    // sort()
    const months = ["Jan", "Feb", "Mar", "Apr"];
    const sorted = months.sort();
    
    console.log(months); // 출력: ["Apr", "Feb", "Jan", "Mar"]
    console.log(sorted); // 출력: ["Apr", "Feb", "Jan", "Mar"]
    
    // toSorted()
    const months = ["Jan", "Feb", "Mar", "Apr"];
    const sorted = months.toSorted();
    
    console.log(months); // 출력: ["Jan", "Feb", "Mar", "Apr"]
    console.log(sorted); // 출력: ["Apr", "Feb", "Jan", "Mar"]
    ```
    
    ```jsx
    // splice()
    const months = ["Jan", "Feb", "Mar", "Apr"];
    const splicedMonths = months.splice(1, 1, "May");
    
    console.log("Original:", months); // 출력: ["Jan", "May", "Mar", "Apr"]
    console.log("Spliced:", splicedMonths); // 출력: ["Feb"]
    
    // toSpliced()
    const months = ["Jan", "Feb", "Mar", "Apr"];
    const newMonths = months.toSpliced(1, 1, "May");
    
    console.log("Original:", months); // 출력: ["Jan", "Feb", "Mar", "Apr"]
    console.log("New:", newMonths); // 출력: ["Jan", "May", "Mar", "Apr"]
    ```
    
    - reverse(), sort(), splice() : 원본 배열 자체를 변경
    - toReversed(), toSorted(), toSpliced() : 원본 배열을 변경하지 않고 새로운 배열을 반환 → 불변성 유지

<br/>

- **`Array.with()`**
    
    `with(위치, 변경할 요소)` : 원본 배열을 변경하지 않고, 지정된 인덱스의 요소를 새 값으로 대체한 새로운 배열을 반환한다.
    
    ```jsx
    const months = ["Januar", "Februar", "Mar", "April"];
    const newMonths = months.with(2, "March");
    
    console.log(months); // 출력: ["Januar", "Februar", "Mar", "April"]
    console.log(newMonths); // 출력: ["Januar", "Februar", "March", "April"]
    ```
    
<br/>

- **`Object.groupBy()` , `Map.groupBy()` → 무려 2024 July**
    
    객체의 요소들을 콜백 함수에서 반환된 문자열 값에 따라 그룹화한다.
    
    원본 객체를 변경하지 않으며, 그룹화된 객체를 반환한다.
    
    - 콜백 함수 : 각 요소를 그룹화할 기준을 정의한다.
    - **참조 공유**: 반환된 객체의 요소와 원본 객체의 요소가 동일한 참조를 공유합니다. (같은 메모리 주소를 가리킴) → 원본 객체나 반환된 객체의 요소를 변경하면, 다른 쪽에서도 그 변경 사항이 반영된다.
    
    ```jsx
    const fruits = [
      { name: "apples", quantity: 300 },
      { name: "bananas", quantity: 500 },
      { name: "oranges", quantity: 200 },
      { name: "kiwi", quantity: 150 }
    ];
    
    function myCallback({ quantity }) {
      return quantity > 200 ? "ok" : "low";
    }
    
    // 기존 방식
    const result = fruits.reduce((acc, fruit) => {
      const key = myCallback(fruit);
      if (!acc[key]) {
        acc[key] = [];
      }
      acc[key].push(fruit);
      return acc;
    }, {});
    
    // 한 줄로 끝
    const result = Object.groupBy(fruits, myCallback);
    
    result.ok[0].quantity = 1000;
    
    console.log(result);
    // 출력:
    // {
    //   ok: [
    //     { name: "apples", quantity: 1000 },
    //     { name: "bananas", quantity: 500 }
    //   ],
    //   low: [
    //     { name: "oranges", quantity: 200 },
    //     { name: "kiwi", quantity: 150 }
    //   ]
    // }
    
    console.log(fruits);
    // 출력:
    // [
    //   { name: "apples", quantity: 1000 },
    //   { name: "bananas", quantity: 500 },
    //   { name: "oranges", quantity: 200 },
    //   { name: "kiwi", quantity: 150 }
    // ]
    ```
    
    ```jsx
    // Map도 동일한 특징을 지니고 동일한 결과를 내줌
    const result = Map.groupBy(fruits, myCallback);
    
    console.log(result)
    // 출력:
    // {
    //   "ok": [
    //   {
    //        "name": "apples",
    //        "quantity": 1000
    //    },
    //    {
    //        "name": "bananas",
    //        "quantity": 500
    //    }
    //  ],
    //  "low": [
    //    {
    //        "name": "oranges",
    //        "quantity": 200
    //    },
    //    {
    //        "name": "kiwi",
    //        "quantity": 150
    //    }
    //  ]
    //}
    ```
    

---

점점 불변성을 지키는 것을 중요시 여기고 있다.

코어 자바스크립트의 1, 2장은 앞으로의 자바스크립트 변화의 예언이나 마찬가지.