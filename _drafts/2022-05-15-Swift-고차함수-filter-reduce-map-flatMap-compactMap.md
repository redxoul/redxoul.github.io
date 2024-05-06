---
title: (Swift) 고차함수 filter, reduce, map + flatMap, compactMap
author: Cody
date: 2022-05-15 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

이전에 썼던 고차함수 글에 이어서,Swift 표준 라이브러리에서 지원하는 고차함수인 `filter`, `reduce`, `map`을 정리해보겠습니다. 이 고차함수들은 `컨테이너 타입(Array, Dictionary, Set, ...)`에 구현되어 있는 [제네릭 함수](https://swiftycody.github.io/posts/Swift-Generic/)입니다.우리가 주로 for문을 돌면서 어떤 결과를 추려낼 때 하던 작업을 이 함수들로 대체할 수 있습니다.

```swift
// 예시로 사용할 컨테이너
let someNumbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

## filter

`filter`는 컨테이너의 값을 걸러서 새로운 컨테이너로 내보내줍니다.이름에서도 느껴지듯이 필터링을 해줍니다.배열 내에 있는 '`Element`타입을 받아서 `Bool`을 반환하는 클로저'를 넣어주면 `Element`배열을 내보내주는데, 이 클로저에 `Element`값을 내보내지는 배열에 포함할지를 결정할 수 있는 조건문을 반환하도록 작성해주면 됩니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/016c9b59-8fcc-41b4-ad2e-63fedee3b173)

먼저 기존의 `for`문을 사용해서 위 예시 컨테이너의 값을 필터링하기 위해 아래와 같이 작성해봅니다.

```swift
// for문 사용
var filtered: [Int] = [Int]()

for number in someNumbers {
    if number % 2 == 0 { // 짝수만 필터링
        filtered.append(number)
    }
}

print(filtered) // [0, 2, 4, 6, 8]
```

아래는 `filter`를 사용해서 작성한 것입니다.

```swift
// filter 사용

// numbers의 요소 중 짝수를 걸러내어 새로운 배열로 반환
let evenNumbers: [Int] = someNumbers.filter ({ (number: Int) -> Bool in
    return number % 2 == 0
})
print(evenNumbers) // [0, 2, 4, 6, 8]
```

`filter`에는 어떤 클로저를 넣어야하는지 타입추론이 가능하기 때문에 매개변수와 반환 타입을 생략이 가능하고, 해당 매개변수를 축약인자 `$0`로 받아서 사용할 수 있습니다. 클로저 외에 넣는 매개변수가 없기 때문에 후행 클로저 규칙으로 소괄호`()`를 생략이 가능합니다. 아래는 단일 표현으로 `return` 키워드까지 생략하여 작성되었습니다.

```swift
// 매개변수, 반환 타입, 반환 키워드(return) 생략, 후행 클로저
let oddNumbers: [Int] = someNumbers.filter {
    $0 % 2 != 0
}
print(oddNumbers) // [1, 3, 5, 7 ,9]
```

## reduce

`reduce`는 컨테이너의 값들을 하나의 값으로 줄이는데 사용됩니다. `reduce`는 두개의 매개변수를 받습니다. 첫번째 매개변수는 초기값(`Result`)입니다. 두번째 매개변수는 기존 결과값(`Result`)과 현재값(`Element`)를 받아서 새 Result를 반환하는 클로저입니다. 숫자 컨테이너의 경우 첫 매개변수(초기값 `Result`)에 보통 `0`을 넣고, 해당 컨테이너 외의 값에 결과를 이어서 계산하고 싶다면 해당 값을 넣어주면 됩니다. 그러면 두번째 매개변수 클로저로 해당 초기값을 컨테이너의 값과 함께 넘겨받게 됩니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f759ed4d-7845-460d-b70b-cd63d7f22195)

먼저 기존의 `for`문을 사용해서 위 예시 컨테이너의 값을 모두 더한 값으로 줄이기 위해 아래와 같이 작성해봅니다.

```swift
// for문 사용
var result: Int = 0

// someNumbers의 모든 요소를 더함
for number in someNumbers {
    result += number
}

print(result) // 45
```

아래는 `reduce`를 사용해서 작성한 것입니다.

```swift
// reduce 사용

// 초기값이 0이고 someNumbers 내부의 모든 값을 더함
let sum: Int = someNumbers.reduce(0, { (result: Int, currentItem: Int) -> Int in
    return result + currentItem
})

print(sum)  // 45
```

위 `reduce`는 아래처럼 동작하여 결과값을 내놓습니다.첫 `currentItem`은 `reduce`첫 인수로 받은 `0`을 받아서 계산하고, 두번째 `currentItem`부터는 앞단계에서의 결과값(`return`)을 받아서 계산하게 됩니다.

| result | currentItem | return |
| --- | --- | --- |
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 3 | 3 | 6 |
| 6 | 4 | 10 |
| ... | ... | ... |
| 36 | 9 | 45 |

`reduce`역시 어떤 매개변수를 넣어야하는지 타입추론이 가능하여 매개변수와 반환 타입을 생략이 가능하고, 해당 매개변수를 순서대로 축약인자 `$0`, `$1`로 받아서 사용할 수 있습니다. 클로저가 후행 클로저 규칙으로 소괄호`()`를 생략이 가능합니다.아래는 단일 표현으로 `return` 키워드까지 생략하여 작성되었습니다.

```swift
// 초깃값이 0이고 someNumbers 내부의 모든 값을 뺌
let subtract = someNumbers.reduce(0) { $0 - $1 }

print(subtract) // -45
```

## map

`map`은 컨테이너의 기존 데이터를 변형(`transform`)하여 새로운 컨테이너를 생성해 반환합니다. 배열 내에 있는 '`Element`타입을 받아서 `T`타입을 반환하는 클로저'를 넣어주면 `T`타입의 배열을 내보내주는데, 이 클로저에 `Element`값을 가공하여 새로운 값 `T`를 반환하도록 작성해주면 됩니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3e23bfa1-9756-435b-995e-b66d0d49bee9)

먼저 기존의 `for`문을 사용해서 위 예시 컨테이너의 값을 변형하기 위해 아래와 같이 작성해봅니다. 기존의 값을 `x2`한 값을 반환하는 `map`입니다.

```swift
// for문을 사용

var multiple = [Int]()

for number in someNumbers {
    multiple.append(number * 2)
}

print(multiple) // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

아래는 `map`을 사용해서 작성한 것입니다.

```swift
// map을 사용

// someNumbers 각 요소를 2배하여 새로운 배열 반환
multiple = someNumbers.map({ (number: Int) -> Int in
    return number * 2
})

print(multiple) // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

`map`도 어떤 클로저를 넣어야하는지 타입추론이 가능하기 때문에 매개변수와 반환 타입을 생략이 가능하고, 해당 매개변수를 축약인자 `$0`로 받아서 사용할 수 있습니다. 클로저 외에 넣는 매개변수가 없기 때문에 후행 클로저 규칙으로 소괄호`()`를 생략이 가능합니다. 아래는 단일 표현으로 `return` 키워드까지 생략하여 작성되었습니다.

```swift
// someNumbers의 각 요소를 3배하여 새로운 배열 반환
let triple = someNumbers.map { $0 * 3 }
print(triple) // [0, 3, 6, 9, 12, 15, 18, 21, 24, 27]
```

## 그리고 flatMap & compactMap

`map`에서 파생된 형태로 `flatMap`과 `compactMap`이 있습니다. 위에서 설명한 `map`과 기본적인 동작은 같지만, 추가적인 기능이 있습니다.

## flatMap

`flatMap`은 `2차원 배열일 때, 1차원 배열로 flat하게 만들어`줍니다.이 때 `배열 내에 nil이 있을 때 제거해주지는 않`습니다.

```swift
let overlayedArray: [[Int?]] = [[1, 2, 3], [nil, 5], [6, nil], [nil, nil]]
let flatMapTest = overlayedArray.flatMap { $0 }

print(flatMapTest)
// [Optional(1), Optional(2), Optional(3), nil, Optional(5), Optional(6), nil, nil, nil]
```

1차원 배열에 `flatMap`을 쓰면 어떻게 될까요? swift4.1부터 `deprecated`되었다면서, `compactMap(_:)`을 사용하라는 경고가 뜹니다. 하지만 일단 결과값은 나오는데, `nil`이 제거가 되어 있습니다. 4.1 이전에는 해당 기능이 `flatMap`에 있었지만 그 이후부터는 `compactMap`으로 이전되었습니다.

```swift
let array = [1, nil, 3, nil, 5, 6, 7]
let flatMapTest = array.flatMap{ $0 } // 'flatMap' is deprecated: Please use compactMap(_:) for the case where closure returns an optional value

print(flatMapTest) // [1, 3, 5, 6, 7]
```

## compactMap

`compactMap`은 (위에도 썼다시피 swift 4.1이전에 flatMap에 있던)'`1차원 배열의 nil값을 제거한 배열을 반환`'해주는 기능을 가지고 있습니다.

```swift
let array = [1, nil, 3, nil, 5, 6, 7]
let compactMapTest = array.compactMap{ $0 }

print(compactMapTest) // [1, 3, 5, 6, 7]
```

이번엔 `compactMap`을 2차원 배열에서 쓰면 어떻게 될까요? 옵셔널 바인딩만 걸립니다.

```swift
let overlayedArray: [[Int?]] = [[1, 2, 3], [nil, 5], [6, nil], [nil, nil]]
let compactMapTest = overlayedArray.compactMap { $0 }

print(compactMapTest)
// [[Optional(1), Optional(2), Optional(3)], [nil, Optional(5)], [Optional(6), nil], [nil, nil]]
```

## 고차함수들의 chaining

그렇다면 2차원 배열인데 `nil`도 제거하고 싶고, 1차원 배열로 flat하게 만들어주고 싶다면? 앞에서 정리한 `flatMap`과 `compactMap`을 체이닝으로 사용하면 됩니다.

```swift
let array: [[Int?]] = [[1, 2, 3], [nil, 5], [6, nil], [nil, nil]]
let flatCompactMapTest = array.flatMap { $0 }.compactMap({ $0 })

print(flatCompactMapTest) // [1, 2, 3, 5, 6]
```

`filter`, `map`, `reduce`과 같은 함수들도 마찬가지입니다. 컨테이너의 값을 홀수만 `filtering`해서, `x2`를 한 값들을 모두 더하고 싶다면, 아래처럼 체이닝을 하면 됩니다.

```swift
let someNumbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

let result = someNumbers.filter { $0 % 2 != 0 } // [1, 3, 5, 7, 9]
    .map { $0 * 2 } // [2, 6, 10, 14, 18]
    .reduce(0) { $0 + $1 } // 50

print(result) // 50
```