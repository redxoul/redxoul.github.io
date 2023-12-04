---
title: (Swift) @dynamicMemberLookup과 Builder 패턴
author: Cody
date: 2023-03-01 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, dynamicMemberLookup, Builder]
pin: false
mermaid: true
---
## dynamicMemberLookup
**`dynamicMemberLookup`**은
**`class`**, **`struct`**, **`enum`**, **`protocol`**에 적용하여 런타임에 **dot**(**`.`**) 문법으로 접근할 수 있도록 해주는 편리한 기능입니다.

**`dynamicMemberLookup`**을 사용하려면 **`subscript(dynamicMember:)`**를 구현해주어야 합니다.

```swift
@dynamicMemberLookup
struct Person {
    var firstName: String
    var lastName: String

    subscript(dynamicMember key: String) -> String {
        switch key {
        case "fullName":
            return "Hey, \(lastName) \(firstName)!!!"
        default:
            return "nope!!!"
        }
    }
}
```

그리고 아래와 같이 사용합니다.

```swift
var man = Person(firstName: "Cody", lastName: "Swifty")
man.fullName // Hey, Swifty Cody!!!
man.say // Nope!!!
```

반대로 얘기하면 컴파일 시점에서는 알 수 없는 멤버의 이름을 사용할 수 있게 해 주기 때문에

컴파일러가 잘못된 멤버이름을 호출하는 것을 감지해주지 못합니다.

그래서 예상하지 못한 에러가 발생하거나, 가독성이 떨어질 수가 있기 때문에 충분한 주석과 적절한 이름을 부여하는 것이 필요합니다.

## **@dynamicmemberlookup과 Builder 패턴**

**`Builder`**패턴은 객체를 생성하는 과정을 캡슐화하여 코드의 가독성과 유연성을 높이는 디자인 패턴 중 하나인데

생성자에 매개변수가 많이 필요할 때 고려해볼 수 있는 패턴입니다.

예를 들어 아래와 같이 작성할 수가 있습니다.

많이 보던 **`setter`**와 비슷하지만 **`Self`**를 **`return`**하는 것이 포인트입니다.

```swift
class Person {
    private var firstName: String = ""
    private var lastName: String = ""
    private var nickName: String = ""
    private var age: Int = 0
    private var nationality = ""

    func setFirstName(_ firstName: String) -> Self {
        self.firstName = firstName
        return self
    }

    func setLastName(_ lastName: String) -> Self {
        self.lastName = lastName
        return self
    }

    func setNickName(_ nickName: String) -> Self {
        self.nickName = nickName
        return self
    }

    func setAge(_ age: Int) -> Self {
        self.age = age
        return self
    }

    func setNationality(_ nationality: String) -> Self {
        self.nationality = nationality
        return self
    }
}
```

그리고 객체를 생성 시 이렇게 해줍니다.

```swift
let cody = Person()
    .setFirstName("Cody")
    .setLastName("Swifty")
    .setNickName("Redxoul")
    .setAge(20)
    .setNationality("Korean")
```

어디서 많이 본 패턴이죠?

**SwiftUI**에서 **View**작성시에도 사용하고,

**RxSwift**에서 **`.rx`**을 통해서 **Reactive extension**을 사용할 때에도 이 패턴이 등장합니다.

그런데 **RxSwift**에서 **Builder**패턴(**`.rx`**)을 구현할 때 위 예제처럼 프로퍼티마다 **`setter`**를 만들어주진 않았습니다.

저렇게 프로퍼티가 늘어날 때마다 **`setter`**를 만들어주면 모듈 작성 시 매우 매우 번거롭고 지저분해지겠죠?

그래서 바로 전에 정리했던 **`KeyPath`**와 **`@dynamicMemberLookup`**을 이용해서

**`NSObject`**의 서브클래스들의 **`Writable Reference Property`**들의 **`setter`**를 자동으로 **Builder**패턴으로 만들어줄 수가 있습니다.

```swift
@dynamicMemberLookup
public struct Builder<Base: AnyObject> {

    private let base: () -> Base

    public init(_ build: @escaping () -> Base) {
        self.base = build
    }

    public init(_ base: Base) {
        self.base = { base }
    }

    public subscript<Property>(dynamicMember keyPath: ReferenceWritableKeyPath<Base, Property>) -> (Property) -> Builder<Base> {
        { [build = base] value in
            Builder {
                let object = build()
                object[keyPath: keyPath] = value
                return object
            }
        }
    }

    public func build() -> Base {
        base()
    }
}

public protocol Buildable {
    associatedtype BuildBase: AnyObject
    var builder: Builder<BuildBase> { get set }
}

extension Buildable where Self: AnyObject {
    public var builder: Builder<Self> {
        get { Builder(self) }
        set { }
    }
}

extension NSObject: Buildable { }
```

이렇게 작성을 하면,

**`NSObject`**의 서브클래스들은 모두 **Builder**패턴으로 인스턴스를 작성할 수 있게 됩니다.

아래처럼 따로 **Builder**패턴의 **`setter`**들을 작성하지 않아도

```swift
class Person: NSObject {
    var firstName: String = ""
    var lastName: String = ""
    var nickName: String = ""
    var age: Int = 0
    var nationality = ""
}
```

이렇게 **`.builder`**를 통해 작성할 수 있게 됩니다.

```swift
var person = Person().builder
    .firstName("Cody")
    .lastName("Swifty")
    .nickName("Redxoul")
    .age(20)
    .nationality("Korean")
    .build()
```

물론 **Foundation**, **UIKit**도 마찬가지입니다.

```swift
var label = UILabel().builder
    .text("Cody")
    .textColor(.blue)
    .backgroundColor(.yellow)
    .textAlignment(.center)
    .build()
```

**RxSwift**의 **Reactive** 확장(**`.rx`**)도 같은 방식으로 구현되어 있는 것을 확인할 수 있습니다.

```swift
// RxSwift패키지의 Reactive.swift

@dynamicMemberLookup
public struct Reactive<Base> {
/// Base object to extend.public let base: Base

/// Creates extensions with base object.////// - parameter base: Base object.public init(_ base: Base) {
        self.base = base
    }

/// Automatically synthesized binder for a key path between the reactive/// base and one of its propertiespublic subscript<Property>(dynamicMember keyPath: ReferenceWritableKeyPath<Base, Property>) -> Binder<Property> where Base: AnyObject {
        Binder(self.base) { base, value in
            base[keyPath: keyPath] = value
        }
    }
}

/// A type that has reactive extensions.public protocol ReactiveCompatible {
/// Extended typeassociatedtype ReactiveBase

/// Reactive extensions.static var rx: Reactive<ReactiveBase>.Type { get set }

/// Reactive extensions.var rx: Reactive<ReactiveBase> { get set }
}

extension ReactiveCompatible {
/// Reactive extensions.public static var rx: Reactive<Self>.Type {
        get { Reactive<Self>.self }
// this enables using Reactive to "mutate" base type// swiftlint:disable:next unused_setter_valueset { }
    }

/// Reactive extensions.public var rx: Reactive<Self> {
        get { Reactive(self) }
// this enables using Reactive to "mutate" base object// swiftlint:disable:next unused_setter_valueset { }
    }
}

import Foundation

/// Extend NSObject with `rx` proxy.extension NSObject: ReactiveCompatible { }
```

**RxSwift**, **RxCocoa**를 그냥 쓰고만 있다가,

'기존 **Foundation**, **UIKit**들이 어떻게 **`.rx`**나 **`.bind`**로 확장해서 쓸 수 있도록 구현한걸까?🤔' 라는 궁금증에 이렇게 정리를 해보게 되었네요.

**`@dynamicMemeberLookup`**과 **Builder**패턴에 대한 정리는 여기까지입니다.😀