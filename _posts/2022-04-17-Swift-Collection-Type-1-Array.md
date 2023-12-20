---
title: (Swift) Collection Type (1)Array
author: Cody
date: 2022-04-17 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---
### 배열의 생성

```swift
// 빈 배열 생성
var someInts = [Int]()

// 3을 추가
someInts.append(3)

// 배열을 비움. Type은 Int로 유지됨
someInts = []

```

### 기본값으로 빈배열 생성

```swift
var threeDoubles = Array(repeating: 0.0, count: 3)
// threeDoubles : Double 타입의 [0.0, 0.0, 0.0]
```

### 배열끼리 합

```swift
var anotherThreeDoubles = Array(repeating: 2.5, count: 3)
// anotherThreeDoubles : [2.5, 2.5, 2.5]

var sixDoubles = threeDoubles + anotherThreeDoubles
// sixDoubles : [0.0, 0.0, 0.0, 2.5, 2.5, 2.5]
```

### 리터럴로 배열 생성

```dart
var shoppingList: [String] = ["Six Eggs", "Milk"]
// 더 간단히, 타입 추론
var shoppingList = ["Six Eggs", "Milk"]

```

### 배열 접근, 변환

```swift
// 갯수
print("The shopping list contains \(shoppingList.count) items.")

// 빈 배열
if shoppingList.isEmpty {
    print("The shopping list is empty.")
}else {
    print("The shopping list is not empty.")
}

// 원소 추가
shoppingList.append("Four")
shoppingList += ["Baking Powder"]
shoppingList += ["Chocolate Spread", "Cheese", "Butter"]

// 인덱스로 접근
var firstItem = shoppingList[0]

// 4, 5, 6번째 인덱스 아이템을 Banana, Apples로 변환. 즉, 아이템 3개가 2개로 줄었다.
shoppingList[4..6] = ["Bananas", "Apples"]

// 특정 인덱스에 원소 추가/삭제/접근
shoppingList.insert("Maple Syrup", at:0)

let mapleSyrup = shoppingList.remove(at: 0)

firstItem = shoppingList[0]
// firstItem : "Six eggs"

let apples = shoppingList.removeLast()

```

### 배열의 순회

```swift
for item in shoppingList {
	print(item)
}
// Six eggs
// Milk
// Flour
// Baking Powder
// Bananas

// enumerated(): 배열의 값, 인덱스를 튜플로 제공
for (index, value) in shoppingList.enumerated() {
	print("Item \\(index + 1): \\(value)")
}
// Item 1: Six eggs
// Item 2: Milk
// Item 3: Flour
// Item 4: Baking Powder
// Item 5: Bananas

```

### 배열의 원소를 연결 (joined)

```swift
let cast = ["Vivien", "Marlon", "Kim", "Karl"]
let list = cast.joined(separator:", ")
print(list)
// "Vivien, Marlon, Kim, Karl" 출력

let list = cast.joined()
print(list)
// "VivienMarlonKimKarl" 출력

```