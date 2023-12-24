---
title: (Swift) 열거형 Enumeration
author: Cody
date: 2022-04-30 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

`Swift`의 열거형(`enumeration`, `enum`)에 대해서 작성해봅니다.

열거형은 유사성을 가진 값들을 공통된 타입으로 선언해서, `타입 안정성(type-safety)`을 보장하는 코드를 작성할 수 있게 해줍니다.

`rawValue`와 `관련값`을 가질 수 있고, [일급객체](https://swiftycody.github.io/posts/Swift-1%EA%B8%89-%EA%B0%9D%EC%B2%B4-First-class-citizen-%EC%99%80-%EA%B3%A0%EC%B0%A8%ED%95%A8%EC%88%98-%EA%B3%A0%EA%B3%84%ED%95%A8%EC%88%98/)이기 때문에 `계산프로퍼티`, `함수`를 작성할 수 있고 `초기화를 지정`하거나 `extension`도 가능합니다.

## 열거형의 기본 문법

열거형의 기본 문법은 아래와 같습니다.

열거형은 타입 정의와 같기 때문에 네이밍을 `파스칼표기법`으로 해주고, case문에는 `카멜표기법`으로 해줍니다.

```swift
enum SomeEnum {
    case someCase
    case anotherCase
    case otherCase
}
```

사용 예.

```swift
enum Almond {
    case honeyButter
    case wasabi
    case corn
    case mintChoco
}
```

여러 `case`들을 한 줄로 정의해 줄 수도 있습니다. 위와 동일한 코드입니다.

```swift
enum Almond {
    case honeyButter, wasabi, corn, mintChoco
}
```

## 열거형의 기본값, 그리고 Objective-C(이하 Objc)의 열거형과의 차이점

`ObjC`에서는 `NSInteger`형으로 열거형을 쓸 수 있었지만,

`Swift`에서는 `Int`, `String`, `Charactor`, `Float`과 같은 타입들을 모두 열거형으로 쓸 수 있습니다.

`ObjC`에서는 열거형을 `typedef`로 정의해서 사용하여 `NSInteger`처럼 사용도 가능하며 정의할 때 해당 `case`의 값을 지정해주지 않으면 암시적으로 `0`부터 차례로 `NSInteger`값이 매겨집니다.

`ObjC`에서는 아래와 같은 코드가 가능합니다.

```cpp
// Objective-C
typedef enum
{
    AlmondTypeHoneyButter,
    AlmondTypeWasabi,
    AlmondTypeCorn,
    AlmondTypeMintChoco,
} AlmondType;

AlmondType myAlmond = AlmondTypeWasabi;

if (myAlmond == 1) { // NSInteger와 동일하게 사용가능
    NSLog(@"myAlmond is 1");
}
```

하지만 `Swift`에서의 `열거형`은 타입을 지정하지 않아도, 지정하여도 해당 타입처럼 사용할 수 없습니다.

정의된 `case`의 값 그대로의 사용만이 가능합니다.

`열거형`의 타입은 아래처럼 지정해주면 됩니다.

마찬가지로 `Int`의 경우 값을 지정해주지 않으면, `0`부터 차례로 `Int`값이 매겨집니다(0, 1, 2, ...).

다만 위 `ObjC`에서 처럼 `Int`와 비교를 해보려하면 `error`를 내놓습니다.

```swift
enum Almond: Int {
    case honeyButter
    case wasabi
    case corn
    case mintChoco
}

let myAlmond = Almond.wasabi

if myAlmond == 1 {	// error!
    print("myAlmond is 1")
}
```

## 열거형의 rawValue

`Swift`의 열거형에서는 `rawValue`가 별도로 존재합니다.

위에서 설명한 해당 `case`의 값을 정의한 타입으로 사용하고자하면 `rawValue`로 꺼내어 사용하면 됩니다.

```swift
if myAlmond.rawValue == 1 {
    print("myAlmond.rawValue is 1") // myAlmond.rawValue is 1
}
```

그리고 `열거형`을 초기화할 때, `rawValue`로 초기화가 가능합니다.

`rawValue`로 초기화를 할 때 정의되지 않은 `case`의 `rawValue`가 들어올 수도 있기 때문에 해당 이니셜라이저는 `optional`을 반환`합니다.

(실제 사용할 때는 `rawValue`로 초기화하는 것을 추천하지 않습니다. 얘기치 않게 지정하지 않은 rawValue가 들어가 오류를 발생시킬 수있는 가능성이 있기 때문입니다)

```swift
let myAlmond: Almond? = Almond(rawValue: 1) // wasabi
let yourAlmond: Almond = Almond(rawValue: 4) ?? .mintChoco // mintChoco
```

열거형을 `String`타입으로 선언하면 `rawValue`는 어떻게 될까요?

별도로 지정을 해주지 않으면, 해당 `case`의 이름이 그대로 `rawValue`로 초기화됩니다.

```swift
enum Almond: String {
    case honeyButter
    case wasabi
    case corn
    case mintChoco = "Your Almond"
}

var oneAlmond: Almond = .honeyButter
print(oneAlmond.rawValue) // honeyButter
oneAlmond = .mintChoco
print(oneAlmond.rawValue) // Your Almond
```

## 열거형과 switch문

열거형과 `switch`문은 매우 잘 어울리는 쌍입니다.

열거형의 `case`에 따라 다른 동작을 작성할 때 늘 쓰이고,

해당 처리에 놓친 부분이 있거나 변동이 있을 때 잘 알아챌 수 있도록 컴파일러가 도와주기 때문입니다.

처리가 다 되지 않은 `case`문이 있다면 아래처럼 `error`를 내놓아 컴파일이 되지 않습니다.

다 처리를 해놓았더라도 후에 `case`문이 추가가 되거나, 수정이 된다면 `error`로 알아챌수 있는 여지를 줍니다.

```swift
switch myAlmond { // Error: Switch must be exhaustive
case .wasabi:
    print("Wasabi is mine")
case .mintChoco:
    print("MintChoco is yours")
}
```

위처럼 일부의 `case`만 처리를 하고, 처리되지 않은 `case`들을 공통으로 빼고 싶다면

아래처럼 `default` case를 작성해주면 `error`를 피할 수 있습니다~~(민트초코도 피할 수 있죠)~~.

```swift
let myAlmond: Almond = Almond(rawValue: 3) ?? .mintChoco

switch myAlmond {
case .wasabi:
    print("Wasabi is mine")
case .mintChoco:
    print("MintChoco is yours") // MintChoco is yours
default:
    print("Other Almonds is mine")
}
```

## 열거형의 관련값

`Swift`의 열거형은 `rawValue`외에도 관련값을 가질 수 있습니다.

관련값은 각 `case`마다 별도의 추가 정보를 저장할 수 있도록 하는 값입니다.

아래처럼 `case`문 뒤에 소괄호`( )`로 정의해 사용할 수 있습니다. 열거형을 만들 때마다 서로 다른 관련값을 넣어줄 수 있습니다.

```swift
enum Almond {
    case honeyButter(Int)
    case wasabi(Int, Int)
    case corn
    case mintChoco(Int)
    
    func printAlmond() {
        switch self {
        case .honeyButter(let count):
            print("My HoneyButter: ", count)
        case .wasabi(let mine, let wives):
            print("My Wasabi:", mine, "/ Wives:", wives)
        case .corn:
            print("Corn Almond")
        case .mintChoco(let yours):
            print("Your MintChoco: ", yours)
        }
    }
}

var almond:Almond = .wasabi(2, 3)
almond.printAlmond() // My Wasabi: 2 / Wives: 3
almond = .mintChoco(5)
almond.printAlmond() // Your MintChoco:  5
```

만약 관련값이 여러개이고, 모두 상수이거나 변수이면 `case` 뒤에 공통으로 `let`, `var`를 써주어 줄일 수도 있습니다.

```swift
enum Almond {
	...
    func printAlmond() {
        switch self {
        ...
        case let .wasabi(mine, wives):
            print("My Wasabi:", mine, "/ Wives:", wives)
        ...
```

Swift 5.0부터 도입된 [Result타입](https://swiftycody.github.io/posts/Swift-Result-%ED%83%80%EC%9E%85/)도 [제네릭](https://swiftycody.github.io/posts/Swift-Generic/) 열거형으로 만들어진 타입이니, 이를 한번 보는 것도 좋은 활용 예시가 될 거 같습니다.