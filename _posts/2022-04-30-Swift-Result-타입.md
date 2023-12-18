---
title: (Swift) Result 타입
author: Cody
date: 2022-04-30 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

Swift 5.0부터 도입된 `Result`타입에 대해 정리해봅니다.

`Result`타입은 실패가 가능한 작업을 할 때

기존에 성공시 원하는 데이터를, `error`발생시 `Error`를 `throw`하는 문법 대신에

`Result`타입을 던지도록 되어 있는 문법입니다.

이걸 도입한 이유는 기존에 `error`를 처리하던 `throw`방식에 몇가지 문제가 있었기 때문입니다.

### 기존 throw, do, try, catch 문법

`Result`타입을 보기 전에 먼저 기존의 `throw`문법을 보겠습니다. 아몬드를 주문하는 예제입니다.

```swift
enum AlmondOrderError: Error, CaseIterable {
    case invalidSelection // 잘못된 선택
    case lackOfMoney // 예산 부족
    case outOfStock // 재고 부족
}

struct Almond {
    var name: String
    var price: Int
    var count: Int
}

struct Reciept {
    var almonds: [Almond]
    var totalPrice: Int {
        return almonds.map { $0.price }
            .reduce(0) { $0+$1 }
    }
    var totalCount: Int {
        return almonds.map { $0.count }
            .reduce(0) { $0+$1 }
    }
}

let budget = 10000
var almondStock = 100
```

이제 기존 `throw` 문법으로 아몬드를 주문하는 함수를 작성해봅니다.

아래 함수는 `Almond` 배열을 받아서 잘못된 주문일 때 `뭔가`(여기서는 `AlmondOrderError`)를 던지고, 그렇지 않을 땐 `Reciept`를 반환하는 함수입니다.

여기서 `뭔가` 라고 한 이유는 정말 함수가 `([Almond]) throws -> Reciept` 형태로 무엇을 던지는지 특정할 수가 없기 때문입니다. 이 특성으로 인해 이를 받아서 사용하는 쪽에서는 문제가 발생합니다.

```swift
func orderAlmond(_ orderedAlmonds: [Almond]) throws -> Reciept {
    var order = [Almond]()

    for almond in orderedAlmonds {
        guard almond.name != "MintChoco" else {
            throw AlmondOrderError.invalidSelection
        }
        guard almond.price <= budget else {
            throw AlmondOrderError.lackOfMoney
        }
        guard almondStock > 0 else  {
            throw AlmondOrderError.outOfStock
        }

        order.append(almond)
    }

    return Reciept(almonds: order)
}
```

이제 위 함수를 사용해보겠습니다.

발생한 `AlmondOrderError`는 `orderError(error:)`에서 처리하도록 했습니다.

```swift
func someFunction() {
    do {
        let reciept = try orderAlmond(almonds)
        print("총 주문된 아몬드 갯수: ", reciept.totalCount)
    }
    catch AlmondOrderError.invalidSelection {
        orderError(error: .invalidSelection)
    }
    catch AlmondOrderError.lackOfMoney {
        orderError(error: .lackOfMoney)
    }
    catch AlmondOrderError.outOfStock {
        orderError(error: .outOfStock)
    }
    catch {
        print("알 수 없는 에러")
    }
}
```

```swift
func orderError(error: AlmondOrderError) {
    switch error {
    case .invalidSelection:
        print(error)
    case .lackOfMoney:
        print(error)
    case .outOfStock:
        print(error)
    }
}
```

위에서 `orderAlmond()`가 어떤 것을 던지는지 `특정할 수가 없다`고 했는데,

그로 인해서 던져진 `error`를 잡기 위해 `catch`문에 명확하게 `AlmondOrderError.invalidSelection` 라고 잡을 `error`를 명시해주어야 합니다. 던져지는 `error`가 많아질 수록 `catch`문이 늘어나 `someFunction()`의 길이도 길어질 것 입니다.

그리고 `AlmondOrderError`에 새로운 케이스가 추가가 된다면?

```swift
enum AlmondOrderError: Error, CaseIterable {
    case invalidSelection
    case lackOfMoney
    case outOfStock
    case addedSomeError // newError added
}
```

`AlmondOrderError`를 받아서 처리하는 `orderError(error:)`에서는 오류가 발생하지만, `someFunction()`의 `do-catch` 문은 평온합니다.

누군가가 위 처럼 새 `AlmondOrderError`를 추가하고 `orderError(error:)`만 수정했다면, `someFunction()`을 처리하던 사람은 이를 눈치채지 못하고 `addedSomeError`는 처리되지 않고 있을 가능성이 생깁니다.

`AlmondOrderError`가 아닌 또다른 종류의 `error`를 `throw`하도록 수정되는 케이스도 마찬가지의 문제가 발생할 수 있습니다.

```swift
enum AnotherOrderError: Error, CaseIterable {
    case nameCountZero
    case anotherError
}
```

```swift
func orderAlmond(_ orderedAlmonds: [Almond]) throws -> Reciept {
    ...
        guard almond.name.count > 0 else { // 새로운 종류의 error를 추가
            throw AnotherOrderError.nameCountZero
        }
```

어떤 것을 던지는지 특정할 수가 없기 때문에

`throw`를 변경할 때마다 해당 함수를 사용하는 곳(`someFunction()`)과 처리하는 곳(`orderError(error:)`)을 따로 수정해주어야 하고, 이를 놓칠수 있는 가능성이 생겨나는 것이죠.

### Result 타입

이제 `Result`타입을 살펴보겠습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8bc24e10-ce48-4da4-8e63-621f0ff067dc)

`Result`타입은 [[Swift] 제네릭 Generic](https://swiftycody.github.io/posts/Swift-Generic/) `(Generic Enumeration)`입니다.

`Result`로 `Success`시 반환할 데이터를 관련값으로 담거나, `Failure`시에 반환할 값을 관련값으로 담으면 되는데

`where`절에서 `Failure`는 `Error`프로토콜`을 따르도록 제한하고 있습니다.

이제 `Result`를 사용하는 `orderAlmonds()`를 작성해보겠습니다.

이 함수의 형태는 `([Almond]) -> Result<Reciept, AlmondOrderError>` 로

`Almond`배열을 매개변수로 받아서

`Success`시 `Reciept`를, `Failure`시 `AlmondOrderError`를 연관값으로 주는 `Result`타입을 반환합니다.

```swift
func orderAlmonds(_ orderedAlmonds: [Almond]) -> Result<Reciept, AlmondOrderError> {
    var order = [Almond]()

    for almond in orderedAlmonds {
        guard almond.name != "MintChoco" else {
            return .failure(.invalidSelection)
        }
        guard almond.price <= budget else {
            return .failure(.lackOfMoney)
        }
        guard almondStock > 0 else  {
            return .failure(.outOfStock)
        }

        order.append(almond)
    }

    return .success(Reciept(almonds: order))
}
```

위 함수를 사용하는 예제입니다.

마찬가지로 받은 `error`를 위에서 작성한 `orderError(error:)`에서 처리하도록 했습니다.

```swift
func someFunction() {
    let almonds = [Almond(name: "MintChoco", price: 5000, count: 1)]

    let orderResult = orderAlmonds(almonds)

    switch orderResult {
    case .success(let reciept):
        print("총 주문된 아몬드 갯수: ", reciept.totalCount)
    case .failure(let error):
        orderError(error: error)
    }
}
```

```swift
func orderError(error: AlmondOrderError) {
    switch error {
    case .invalidSelection:
        print(error)
    case .lackOfMoney:
        print(error)
    case .outOfStock:
        print(error)
    }
}
```

깔끔하게 정리가 된 `someFunction()` 입니다.

반환되는 값이 `throw`로 던져지는 것이 아니라 `AlmondOrderError`로 명확하게 특정되어지기 때문에,

컴파일 단계에서 `error`에 대한 처리를 더 명확하게 해줄 수 있게 되었습니다.

`error`의 케이스가 추가, 변경되어도 이에 대한 변경을 `orderError(error:)`에서 처리해주면 됩니다.

error처리를 `orderError(error:)`에서 하든, `case .failure`에서 하든 상관없이,

일관된 위치에서 일관된 흐름으로 처리할 수 있게 되었습니다.

### 정리

`throw` 문은 어떤 값을 던지는 지 특정하지 않기 때문에,

해당 함수를 사용하는 곳에서 어떤 `error`를 처리해야 하는지 명확하게 알기 어렵고, `throw`의 변경이 생겼을 때 이에 대한 변화를 알아채기가 어렵습니다.

`Result`타입을 사용하면 성공시 반환할 타입과 실패시 반환할 `Error`타입을 명시해 줌으로써,

컴파일 단계에서 명확하게 이를 처리할 수 있게해 위 문제를 해결해주며, 처리에 대한 흐름을 단순하고 안정성있게 만들어줍니다.