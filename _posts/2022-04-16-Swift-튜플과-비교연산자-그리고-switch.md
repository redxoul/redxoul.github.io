---
title: (Swift) 튜플과 비교연산자, 그리고 switch
author: Cody
date: 2022-04-16 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

## 튜플
Swift에서의 비교연산자의 기본적인 내용은 타 언어의 그것과 같습니다.  
나머지는 생략하고 튜플의 비교에 대해서 정리해봅니다.  
Swift에서는 같은 타입의 값을 갖는 두 개의 튜플을 비교할 수 있습니다.  
튜플의 비교는 왼쪽에서 오른쪽 방향으로 이뤄지고 한번에 한개의 값만 비교합니다.  
이 비교를 다른 두 값을 비교하게 될 때까지 수행합니다.

예시:

```swift
(1, "zebra") < (2, "apple")
// true, 1이 2보다 작고; zebra가 apple은 비교하지 않기 때문
(3, "apple") < (3, "bird")
// true 왼쪽 3이 오른쪽 3과 같고; apple은 bird보다 작기 때문
(4, "dog") == (4, "dog")
// true 왼쪽 4는 오른쪽 4와 같고 왼쪽 dog는 오른쪽 dog와 같기 때문
```

튜플은 비교 연산자가 해당 값을 비교할 수 있는 경우에만 비교를 수행할 수 있습니다.

```swift
("blue", -1) < ("purple", 1)        // 비교가능, 결과 : true
("blue",false) < ("purple",true)  // 에러, Boolean 값은 < 연산자로 비교할 수 없기 때문에
```

Swift 표준 라이브러리에서는 7개 요소 미만을 갖는 튜플만 비교할 수 있으며,

만약 7개 혹은 그 이상의 요소를 갖는 튜플을 비교하고 싶으면 직접 비교 연산자를 구현해야 합니다.

## 튜플과 switch
swift에서는 switch문에서 튜플을 조건으로 사용이 가능합니다.

```swift
let somePoint = (1, 1)

switch somePoint {
case (0, 0):
    print("\(somePoint) is at the origin")
case (_, 0):
    print("\(somePoint) is on the x-axis")
case (0,_):
    print("\(somePoint) is on the y-axis")
case (-2...2, -2...2):
    print("\(somePoint) is inside the box")
default:
    print("\(somePoint) is outside of the box")
}
// Prints "(1, 1) is inside the box"
```

그리고 case문에서 튜플의 각 값을 정의하고 사용할 수 있는데,  
이런 방법을 `값 바인딩(value-bindings)`라고 합니다.  
어떤 case문에서 정의한 값을 다른 case문에서도 사용할 수가 있습니다.

```swift
// 특정 x, y 값을 각각 다른 case에 정의하고 그 정의된 상수를 또 다른 case에서 사용할 수 있음
let anotherPoint = (2, 0)

switch anotherPoint {
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case (0,let y):
    print("on the y-axis with a y value of \(y)")
case let (x, y):
    print("somewhere else at (\(x), \(y))")
}
// Prints "on the x-axis with an x value of 2"
```

case문에 `where`를 사용하여 조건문을 사용할 수도 있습니다.

```swift
// where 조건문
let yetAnotherPoint = (1, -1)

switch yetAnotherPoint {
case let (x, y)where x == y:
    print("(\(x), \(y)) is on the line x == y")
case let (x, y)where x == -y:
    print("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    print("(\(x), \(y)) is just some arbitrary point")
}
// Prints "(1, -1) is on the line x == -y"
```