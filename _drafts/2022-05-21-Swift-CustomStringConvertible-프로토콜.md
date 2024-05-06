---
title: (Swift) CustomStringConvertible 프로토콜
author: Cody
date: 2022-05-21 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

`CustomStringConvertible` 프로토콜은

객체를 `String`으로 변환해서 표현하고 싶을 때 사용하는 프로토콜로, 주로 `print()`함수를 통해 출력할 때 유용합니다.

정의를 보면 `description` 연산 프로퍼티를 구현하도록 되어 있는 것을 확인할 수 있습니다.

예제에서는 `Point` 구조체를 `CustomStringConvertible` 프로토콜을 구현하여, `print()`함수 호출시 출력할 `String`을 정의해주었습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/16a6cfd6-2909-411e-a929-a0f4498e9494)

사실 구조체는 `CustomStringConvertible`을 구현하지 않아도 어느 정도 `String`으로 변환해서 보여줍니다.

아래는 `Almond` 구조체입니다. `print`했을 때, 객체의 타입과 프로퍼티 값들을 보여줍니다.

```swift
struct Almond {
    var name: String
    var yummy: Bool
}

let someAlmond = Almond(name: "Mint Choco", yummy: false)
print(someAlmond) // Almond(name: "Mint Choco", yummy: false)
```

아래는 `Almond` 클래스입니다.  구조체였을 때와 달리 출력했을 때 의도하지 않은 `String`이 출력됩니다.

```swift
class Almond {
    var name: String
    var yummy: Bool
    
    init(name: String, yummy: Bool) {
        self.name = name
        self.yummy = yummy
    }
}

let someAlmond = Almond(name: "Mint Choco", yummy: false)
print(someAlmond) // __lldb_expr_281.Almond
```

이럴 때, `CustomStringConvertible`을 구현해주면 원하는 String으로 해당 클래스의 객체를 표현해줄 수 있게 됩니다.

```swift
extension Almond: CustomStringConvertible {
    var description: String {
        return "\(name), \(yummy ? "Yummy!" : "Nope!")"
    }
}

let someAlmond = Almond(name: "Mint Choco", yummy: false)
print(someAlmond) // Mint Choco, Nope!
```