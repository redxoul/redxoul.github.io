---
title: (Swift) 제네릭 Generic
author: Cody
date: 2022-04-24 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

`제네릭 Generic`은 단어의 뜻이 '`포괄적인`, 통칭의'라는 의미입니다.

단어의 의미처럼 `제네릭`은 타입에 종속적이지 않도록

Swift 코드를 좀 더 유연하고, 재사용 가능한 함수 및 코드를 작성할 수 있도록 도와주는 요소입니다.

Swift 표준 라이브러리에서 기본 제공하는 `swap` 함수를 예시로 들 수 있습니다.

Swift의 기본 swap함수가 없다고 가정하고,

`두 Int값을 inout 매개변수로 받아서 해당 원본값을 서로 바꿔주는 함수`를 작성한다고 한다면,

```swift
// a, b 두 개의 Int값을 바꾸려는 함수
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
let temporaryA = a
    a = b
    b = temporaryA
}

var intOne = 3
var intTwo = 107
swapTwoInts(&intOne, &intTwo)
// intOne: 107, intTwo: 3
```

만약 후에 `Int`가 아니라, `String`이나 또다른 타입을 위한 함수가 필요해진다면,

해당 타입에 대한 함수를 또 작성해야 합니다. `제네릭`이 없다면 말이죠.

```swift
func swapTwoStrings(_ a: inout String, _ b: inout String) {
let temporaryA = a
    a = b
    b = temporaryA
}

func swapTwoDoubles(_ a: inout Double, _ b: inout Double) {
let temporaryA = a
    a = b
    b = temporaryA
}
```

## 제네릭 함수와 타입 파라미터

제네릭을 사용하면 위 함수들을

매개변수 타입이 달라도 공통으로 사용할 수 있는 `제네릭 함수`를 작성할 수 있습니다.

함수의 형식은 위 함수들과 동일하지만

함수명 바로 뒤에 `<T>`를 붙여주고, 소괄호 안에 해당 `T`로 타입을 정의해줍니다.

여기서 `T`는 `타입 파라미터`라고 합니다.

```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
let temporaryA = a
    a = b
    b = temporaryA
}
```

`타입 파라미터`는 꼭 `T`로 명명하지 않아도 됩니다. 보통은 `T`, `U`, `V`, ... 처럼 단일 대문자로 표기하고,

`Dictionary`의 `Key`, `Value`와 같이 요소간에 상관관계가 있는 경우 의미있는 이름을 붙입니다.

단일 대문자로 짓지 않는 경우엔 타입의 이름이기 때문에 `파스칼 표기법(PascalCase)`으로 표기합니다.

여러 타입 파라미터를 받고 싶을 때는 `쉼표`( `,`)로 구분하여 `<T, U, V>`과 같이 여러개를 작성합니다.

## 제네릭 타입

`제네릭 함수`를 만들듯이 `제네릭 타입`을 작성할 수 있습니다.

`제네릭 타입`으로 `Stack`을 아래와 같이 구현할 수 있습니다.

`stack`에 넣을 객체의 타입에 의미를 부여하기 위해 `Element`라는 매개변수 이름으로 작성합니다.

```swift
struct Stack<Element> {
    var items = [Element]()
    mutatingfuncpush(_ item: Element) {
        items.append(item)
    }
    mutatingfuncpop() -> Element {
        return items.removeLast()
    }
}
```

스택에 `push`를 할 때는 아래와 같이 실행하고,

```swift
var almondsStack = Stack<String>()
almondsStack.push("MintChoco")
almondsStack.push("Corn")
almondsStack.push("Wasabi")
almondsStack.push("HoneyButtuer")

print(almondsStack.items)
// ["MintChoco", "Corn", "Wasabi", "HoneyButtuer"]
```

`pop`도 아래와 같이 실행할 수 있습니다.

~~(민트초코는 먹지 않으니 pop하지 않아도 됩니다)~~

```swift
almondsStack.pop() // HoneyButtuer
almondsStack.pop() // Wasabi
almondsStack.pop() // Corn
```

## 제네릭 타입 확장

`extension`으로 `제네릭 타입`도 확장이 가능합니다.

확장할 `제네릭 타입`에서 사용했던 매개변수의 이름을 그대로 가져와서 사용합니다.

```swift
extension Stack {
var topItem: Element? {
return items.isEmpty ? nil : items[items.count - 1]
    }
}

// 확장으로 작성한 topItem에 접근
if let topItem = stackOfStrings.topItem {
    print("The top item:", topItem)
}
// The top item: MintChoco
```

## 타입 파라미터의 타입제한

`타입 파라미터`가 기본적으로는 어떤 타입이든 받을 수 있도록 해주지만,

타입을 제한적으로 받고 싶을 수 있습니다.

Swift의 기본 타입중 하나인 `Dictionary`는 `Key`값으로 여러타입을 받지만,

유일한 값이어야만 하기 때문에 

`프로토콜`을 따르는 타입으로 `제한`됩니다.

이처럼 제네릭에서도 `의도한 클래스를 상속하거나, 의도한 프로토콜을 따르도록 명시`할 수가 있습니다.

`타입제한을 위한 문법`은 아래와 같습니다.

```swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // 함수 body
}
```

어떤 배열에서 특정 문자를 찾아 인덱스를 반환하는 함수(값이 없다면 `nil`을 반환)를 예시로 들면,

```swift
func findIndex(ofString findString: String, in array: [String]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == findString {
            return index
        }
    }
    return nil
}
```

아몬드 배열을 만들어 민트초코의 인덱스를 찾아보면, 해당 인덱스를 잘 찾아줍니다.

~~(이제 해당 인덱스만 빼고 맛있게 먹으면 되겠죠?)~~

```nix
let almonds = ["HoneyButter", "Wasabi", "Corn", "Buldak", "Ddukboki", "MintChoco"]
if let almondIndex = findIndex(ofString: "MintChoco",in: almonds) {
    print("Mint Choco's Index: ", almondIndex) // 5
}
```

위 `findIndex()`를 제네릭으로 구현을 하면,

'`Binary operator '==' cannot be applied to two 'T' operands`' 이라고 에러가 발생합니다.

`==` 연산자`를 사용하기 위해서는 두 객체가 `Equatable`프로토콜`을 따라야하기 때문입니다.

```swift
func findIndex<T>(of findValue: T, in array: [T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == findValue {
            // Binary operator '==' cannot be applied to two 'T' operands
            return index
        }
    }
    return nil
}
```

`T` `타입 파라미터`에 타입제한을 `Equatable`로 주면 정상적으로 컴파일이 됩니다.

```swift
func findIndex<T: Equatable>(of findValue: T, in array: [T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == findValue {
            return index
        }
    }
    return nil
}

let almonds = ["HoneyButter", "Wasabi", "Corn", "Buldak", "Ddukboki", "MintChoco"]
if let almondIndex = findIndex(of: "MintChoco", in: almonds) {
    print("Mint Choco's Index: ", almondIndex) // 5
}
```

## 프로토콜과 연관타입 associatedtype

`연관타입`은 `프로토콜`에서 사용할 수 있습니다.

타입에 플레이스 홀더 이름을 부여해서 특정 타입을 동적으로 지정해서 사용할 수 있게 해줍니다.

아래와 같이 `Item`에 `associatedtype`을 지정해줄 수 있습니다.

이렇게 지정하면 `Item`은 어떤 타입으로든 받도록 프로토콜을 작성할 수 있습니다.

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```

`Container`프로토콜을 `Stack`구조체에 적용을 해보면 아래와 같습니다.

```swift
struct Stack<Element>: Container {
    // Stack<Element> implementation
    var items = [Element]()
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
    
    // for Container protocol
    mutating func append(_ item: Element) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Element {
        return items[i]
    }
}
```

## 기존 타입에 연관타입을 확장

기존 `Array` 타입을 `Container`프로토콜로 연관타입을 확장.

기존 `Array`는 이미 `append()`,` `count`, `subscript`가 정의되어 있기 때문에 아래처럼 확장만 해주어도 간단하게 확장 가능.

```swift
extension Array: Container {}
```

## 프로토콜에 associatedtype 적용을 제한하기

`타입 파라미터`를 작성할 때 `특정 타입`이나, `프로토콜`으로 제한하듯이

프로토콜에 `associatedtype`을 적용할 때도, 조건을 작성하여 제한을 둘 수 있습니다.

`associatedtype` 선언 뒤에 `where`절을 적용하면 이를 적용할 수 있습니다.

```swift
protocol SuffixableContainer: Container {
    associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
    func suffix(_ size: Int) -> Suffix
}
```

## SuffixableContainer

→ `Suffix`가 `SuffixableContainer`프로토콜을 따르고,

`where Suffix.Item == Item`

→ `Item`타입이 반드시 `Container`의 `Item`타입이어야 한다는 조건으로 제한

`Stack` 구조체를 `extension`으로 확장하여, `suffixableContainer` 프로토콜을 적용.

```swift
extension Stack: SuffixableContainer {
        // Stack의 연관 타입인 Suffix 또한 Stack
        // Stack의 suffix의 실행으로 또 다른 Stack을 반환
    func suffix(_ size: Int) -> Stack {
        var result = Stack()
        for index in (count-size)..<count {
            result.append(self[index])
        }
        return result
    }
}
```

`SuffixableContainer`를 따르는 `Stack`의 사용 예.

~~(민트초코를 뺀 `suffixAlmonds`만 먹으면 됩니다)~~

```swift
var almondsStack = Stack<String>()
almondsStack.append("MintChoco")
almondsStack.append("HoneyButter")
almondsStack.append("Wasabi")
almondsStack.append("Corn")
almondsStack.append("Buldak")

let suffixAlmonds = almondsStack.suffix(4)
print(suffixAlmonds.items) // ["HoneyButter", "Wasabi", "Corn", "Buldak"]
```

## 제네릭의 where절

제네릭에서도 `where`절의 사용이 가능합니다.

Container 2개를 서로 비교하여 모든 값이 같을 때 `true`를 반환하는 `allItemsMatch()` 제네릭함수를 구현.

```swift
// Container 종류 자체는 달라도 상관없음
// 두 Container의 같은 인덱스의 모든 값이 같기만 하면 결과로 true를 반환
func allItemsMatch<C1: Container, C2: Container>
    (_ someContainer: C1, _ anotherContainer: C2) -> Bool
    where C1.Item == C2.Item, C1.Item: Equatable {

        // 두 Container의 count를 먼저 체크
        if someContainer.count != anotherContainer.count {
            return false
        }

        // count가 같다면 각 쌍을 비교
        for i in 0..<someContainer.count {
            if someContainer[i] != anotherContainer[i] {
                return false
            }
        }

        // 모든 아이템이 같음
        return true
}
```

`allItemsMatch()` 제네릭함수의 사용.

```swift
var almondsPackage = Stack<String>()
almondsPackage.append("HoneyButter")
almondsPackage.append("Wasabi")
almondsPackage.append("Corn")
almondsPackage.append("Buldak")
almondsPackage.append("MintChoco")

let almondsIncludeMincho = ["HoneyButter", "Wasabi", "Corn", "Buldak", "MintChoco"]

if allItemsMatch(almondsPackage, almondsIncludeMincho) {
    print("민트초코가 들어 있는 패키지")
}
else {
    print("민트초코가 없는 패키지")
}
```

## where절을 포함하는 제네릭 익스텐션

제네릭을 `extension`으로 확장할 때, `where`절을 포함시킬 수 있습니다.

```swift
// isTop함수를 익스텐션으로 추가하면서
// 이 함수가 추가되는 Stack은 반드시 Equatable 프로토콜을 따라야 한다고 제한
extension Stack where Element: Equatable {
    func isTop(_ item: Element) -> Bool {
        guard let topItem = items.last else {
            return false
        }
        return topItem == item
    }
}
```

`where`절을 통해 `Equatable`프로토콜을 따르는 `Stack`일 때 `isTop`을 사용하는 예시

```swift
// String은 Equatable프로토콜을 따르므로 true에 해당하는 분기가 실행
if almondsPackage.isTop("MintChoco") {
    print("민트초코가 제일 위에 있음")
} else {
    print("민트초코가 제일 위에 있지 않음")
}
// 민트초코가 제일 위에 있지 않음
```

`Extension`으로 `Container`의 `Item`이 `Equatable`프로토콜`을 따를 때 `startWith()`함수를 쓸 수 있도록 확장.

```swift
extension Container where Item: Equatable {
    func startsWith(_ item: Item) -> Bool {
        return count >= 1 && self[0] == item
    }
}
```

`"MintChoco"` 문자열은 `Equatable`프로토콜`을 따르기 때문에 `startWith()` 함수 사용가능.

```swift
let almondsPackage = ["MintChoco", "HoneyButter", "Wasabi", "Corn", "Buldak"]

if almondsPackage.startsWith("MintChoco") {
    print("Starts with MintChoco.")
} else {
    print("Starts with something else.")
}
// Starts with something else.
```

`where`절은 프로토콜 뿐만 아니라 `특정 타입일 때도 제한`을 둘 수 있음.

```swift
extension Container where Item == Double {
    // Item이 Double타입일 때, average함수를 쓸 수 있음
    func average() -> Double {
        var sum = 0.0
        for index in 0..<count {
            sum += self[index]
        }
        return sum / Double(count)
    }
}

print([1260.0, 1200.0, 98.6, 37.0].average()) // 648.9
// Container의 Item [1260.0, 1200.0, 98.6, 37.0]이 Double형이기 때문에
// average()를 사용할 수 있음
```

## 제네릭의 연관타입에 where절 사용

제네릭의 연관타입에도 `where`절을 사용하여 제한을 둘 수 있습니다.

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
    
    // (아래 코드는 기존 Container protocol에 추가)
	// 연관 타입 Iterator에 Iterator의 Element가 Item과 같아야 한다는 제한을 둠
    associatedtype Iterator: IteratorProtocol where Iterator.Element == Item
    func makeIterator() -> Iterator
}
```

다른 프로토콜을 상속받는 프로토콜에서도 `where`절로 제한을 둘 수 있습니다.

```swift
protocol ComparableContainer: Container where Item: Comparable { }
```

## 제네릭의 subscript에 where절로 제한

제네릭의 `subscript`에도 `where`절로 제한을 둘 수 있습니다.

```swift
// Indices.Iterator.Element가 Int타입이어야 한다는 제한을 둠
extension Container {
    subscript<Indices: Sequence>(indices: Indices) -> [Item]
        where Indices.Iterator.Element == Int {
            var result = [Item]()
            for index in indices {
                result.append(self[index])
            }
            return result
    }
}
```