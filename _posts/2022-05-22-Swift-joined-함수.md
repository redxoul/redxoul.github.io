---
title: (Swift) joined 함수
author: Cody
date: 2022-05-22 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

`joined` 함수를 정리해봅니다.

Sequence가 들어있는 배열일 때, 하나의 Sequence로 이어주는 함수입니다.

`String`도 `Charactor`의 Sequence라고 볼 수 있고, `Int` 배열 해당이 됩니다.

`String`일 때와 그 이외의 배열일 때가 약간 다릅니다.

### String 배열의 joined

`String`의 배열의 `joined` 함수의 정의는 아래와 같습니다. 매개변수로 `seperator`가 있고 생략했을 때는 `""`가 기본으로 적용됩니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0a78413c-ab5e-4cc2-b4dd-1d8f0db21aa6)

`String`의 배열에 `joined`를 적용하면 아래처럼 이어집니다.

```swift
let someAlmonds =  ["HoneyButter", "Wasabi", "Corn", "Buldak", "MintChoco"]
let joinedAlmonds = someAlmonds.joined()

print(joinedAlmonds) // HoneyButterWasabiCornBuldakMintChoco
```

`joined(separator:)`를 적용하면 `String` 사이에 들어갈 `String`을 넣어줄 수 있습니다.

```swift
let joinedAlmonds = someAlmonds.joined(separator: " and ")

print(joinedAlmonds) // HoneyButter and Wasabi and Corn and Buldak and MintChoco
```

### 그 외 Sequence의 배열에서의 joined

`String`이 아닐 때는 `joined()`와 `joined(seperator:)`가 조금 다릅니다.

먼저 그냥 `joined()`를 보면, `FlattenSequence`를 반환한다고 되어 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5b74de8a-aa68-4393-97c6-92a6c73b4ae7)

아래는 예제입니다.

`joined`만 한 경우 `FlattenSequence`가 반환되어, 배열로 사용하고 싶을 땐 배열에 해당값으로 초기화하여 사용해야 합니다.

```swift
let someNumbers = [[0, 1, 2, 3, 4], [5, 6 ,7, 8, 9]]
let joinedNumbers = someNumbers.joined()
let joinedNumberArray = Array(joinedNumbers)

print(joinedNumbers) // FlattenSequence<Array<Array<Int>>>(_base: [[0, 1, 2, 3, 4], [5, 6, 7, 8, 9]])
print(joinedNumberArray) // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

다음은 `joined(seperator:)`입니다.

`seperator`의 Sequence값과 `joined`할 시퀸스가 같은 타입으로 넣도록 제네릭으로 선언되어 있고,

`where`절로 `Seperator`가 Sequence 프로토콜을 따르고, `Sequence`의 `Element`가 `joined`하는 `Sequence`의 `Element`의 `Element`가 같은 타입이어야 한다고 제한되어져 있습니다.

`JoinedSequence`가 반환된다는 점도 다릅니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c3d1cbff-0783-4e15-be28-36489450a0c7)

아래는 `joined(seperator:)`의 예제입니다.

`JoinedSequence`가 반환되며, 배열로 사용하고 싶을 땐 마찬가지로 배열에 해당값으로 초기화하여 사용합니다.

```swift
let someNumbers = [[0, 1, 2, 3, 4], [5, 6 ,7, 8, 9]]
let joinedNumbers = someNumbers.joined(separator: [-1])
let joinedNumberArray = Array(joinedNumbers)

print(joinedNumbers) // JoinedSequence<Array<Array<Int>>>(_base: [[0, 1, 2, 3, 4], [5, 6, 7, 8, 9]], _separator: ContiguousArray([-1]))
print(joinedNumberArray) // [0, 1, 2, 3, 4, -1, 5, 6, 7, 8, 9]
```