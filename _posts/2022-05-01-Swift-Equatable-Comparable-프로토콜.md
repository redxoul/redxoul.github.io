---
title: (Swift) Equatable, Comparable 프로토콜
author: Cody
date: 2022-05-01 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

## Equatable
`Equatable`은

두개의 값이 동일한 값인지 아닌지 비교를 하기 위해서 따라야하는 프로토콜입니다.

이 프로토콜을 따르는 타입의 인스턴스는 `==` 나 `!=` 연산자로 같은지, 같지 않은지 판단할 수 있게 됩니다.

Swift 표준 라이브러리의 대부분의 기본 타입들(`Int`, `Double`, `Float`, `String`, `Bool`, ...)은 이 프로토콜을 따르고 있습니다.

기본 타입들이 아닌 클래스, 구조체도 해당 프로토콜을 따르면

비교연산자를 통해서 같은지, 아닌지 판단할 수 있게 됩니다.

`Equatable` 프로토콜을 살펴보면

`static func ==(lhs: Self, rhs: Self)->Bool` 함수를 구현함으로써 해당 프로토콜을 쓸 수 있게 되어 있습니다.

`lhs`는 `==`의 왼쪽에, `rhs`는 `==`의 오른쪽에 들어갈 타입을 의미합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ab1bb298-f0b3-4b69-81f1-ae04ed552831)

아래는 예시입니다.

`Equatable`프로토콜을 따르는 `AlmondPackage`는

저장프로퍼티 중에서 `Almond`값을 비교해서 `같은 패키지를 골랐는지 비교`할 수 있게 되었습니다.

```swift
enum Almond {
    case honeyButter, wasabi, corn, mintChoco
}

class AlmondPackage: Equatable {
    var almond: Almond
    var priority: Int
    init(almond: Almond = .honeyButter, priority: Int = 1) {
        self.almond = almond
        self.priority = priority
    }
    
    public static func ==(lhs: AlmondPackage, rhs: AlmondPackage) -> Bool {
        return lhs.almond == rhs.almond
    }
}

let myAlmondPackage = AlmondPackage(almond: .wasabi)
let yourAlmondPackage = AlmondPackage(almond: .mintChoco)
let wivesAlmondPackage = AlmondPackage(almond: .wasabi)

myAlmondPackage == yourAlmondPackage // false
myAlmondPackage == wivesAlmondPackage // true
```

## Comparable

`Comparable`은 이름에서도 느낌이 오듯이,

두개의 값 중에 어떤 값이 큰값인지 아닌지 비교를 하기 위해서 따라야하는 프로토콜입니다.

이 프로토콜을 따르는 타입의 인스턴스는 `<` 나 `>`, `<=`, `>=` 연산자로 같은지, 같지 않은지 판단할 수 있게 됩니다.

`Comparable` 프로토콜을 살펴보면

`Comparable` 프로토콜이 `Equatable` 프로토콜`을 따르고 있습니다.

`Comparable`을 위해서는 `Equatable`의 메서드도 구현해주어야 합니다.

그리고 아래와 같은 4개의 함수가 정의되어 있는데, 이 중 하나의 함수를 구현함으로써 해당 프로토콜을 쓸 수 있게 되어 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9888c30a-6817-4c51-9e0b-6feae8a00913)

아래는 예시입니다.

`Comparable`프로토콜을 따르는 `AlmondPackage`는

저장프로퍼티 중에서 `priority`값을 비교해서 `같은 패키지를 골랐는지 비교`할 수 있게 되었습니다.

```swift
enum Almond {
    case honeyButter, wasabi, corn, mintChoco
}

class AlmondPackage: Comparable {
    
    var almond: Almond
    var priority: Int
    init(almond: Almond = .honeyButter, priority: Int = 1) {
        self.almond = almond
        self.priority = priority
    }
    
    public static func ==(lhs: AlmondPackage, rhs: AlmondPackage) -> Bool {
        return lhs.almond == rhs.almond
    }
    
    public static func < (lhs: AlmondPackage, rhs: AlmondPackage) -> Bool {
        return lhs.priority < rhs.priority
    }
}

let myAlmondPackage = AlmondPackage(almond: .wasabi, priority: 3)
let yourAlmondPackage = AlmondPackage(almond: .mintChoco)

myAlmondPackage > yourAlmondPackage // true
myAlmondPackage == yourAlmondPackage // false
```