---
title: (Swift) Property Wrapper (1)
author: Cody
date: 2022-08-30 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, SwiftUI, iOS13]
pin: false
mermaid: true
---
Swift, SwiftUI로 코드를 작성하다보면

프로퍼티 앞에 붙여주는 아래와 같은 친구들(?)을 마주하게 됩니다.

`@main`, `@Environment`, `@State`, `@Binding`, `@Published`, `@ObservedObject`, `@ViewBuilder`, `@escaping` 등..

`@`가 앞에 붙어 있는 이 친구들은 `Property Wrapper`라고 부릅니다.

그 동안은 그냥 각각의 용도에 맞춰서 프로퍼티 앞에 붙여서 사용하고 있었는데,

(1)에서는 이 Property Wrapper가 무엇인지 알아보고,

(2)(다음글)에서는 자주 마주하는 Property Wrapper들의 용도도 정리해보려 합니다.

`swift.org`의 property 문서를 보면 `Property Wrapper`의 설명이 아래와 같이 되어 있습니다.

(https://docs.swift.org/swift-book/LanguageGuide/Properties.html)

> A property wrapper adds a layer of separation between code that manages how a property is stored and the code that defines a property. ...(중략)... When you use a property wrapper, you write the management code once when you define the wrapper, and then reuse that management code by applying it to multiple properties.
> 

`property가 저장되는 방법을 관리하는 코드와 property를 정의하는 코드 사이에 구분 레이어를 추가`해준다.

Property Wrapper를 사용할 땐, `wrapper를 정의할 때 관리코드를 한번 작성`하고

`여러 property들에 적용하여 해당 관리코드를 재사용`한다.

...라고 합니다.

핵심은 `'관리코드를 한번 작성'`하고, `해당 관리코드를 재사용한다`인 거 같습니다.

문서 바로 아래에 예제가 있는데,

필요한 Property Wrapper를 만들어서 사용할 수 있다라는 걸 알았습니다.

Property Wrapper를 정의해주려면

아래와 같이 `@propertyWrapper`를 `구조체`, `열거형`, `클래스` 앞에 선언해주고,

`wrappedValue` 연산프로퍼티를 정의해주면 됩니다.

문서의 예제를 참고로 해서 응용하여,

서버로부터 SnakeCase 문자열을 받았을 때, CamelCase의 문자열로 저장해주는

`@CamelFromSnake` 라는 Property Wrapper를 작성해봤습니다.

```swift
@propertyWrapper
struct CamelFromSnake {
    private var string = ""
    
    var wrappedValue: String {
        get { return string }
        set {
            let seperated = newValue.components(separatedBy: "_")
            string = (seperated.first?.lowercased() ?? "") + seperated.dropFirst()
                .map { $0.prefix(1).uppercased() + $0.dropFirst() }
                .reduce("") { $0 + $1 }
        }
    }
    
    init(wrappedValue initialValue: String) {
        self.wrappedValue = initialValue
    }
}
```

위처럼 작성한 Property Wrapper는
아래처럼 사용을 해주면 됩니다.

```swift
class SomeClass {
    @CamelFromSnake var someProperty: String = ""
}

var someInstance = SomeClass()
someInstance.someProperty = "from_server_property"
print("someProperty: \(someInstance.someProperty)")
```

그럼 아래처럼 출력이 되어 CamelCase로 값이 저장이 된 것을 확인할 수 있습니다.

```swift
someProperty: fromServerProperty
```

`Property Wrapper`를 활용하면,

관리가 필요한 프로퍼티에 대한 정의를 하고, 이를 간단하게 선언함으로써 반복되는 코드도 많이 줄일 수 있어 보입니다.

[Swift-evolution](https://github.com/apple/swift-evolution/blob/main/proposals/0258-property-wrappers.md)의 Property Wrapper를 보면 이를 활용한 다양한 예제들을 확인해 볼 수가 있습니다.