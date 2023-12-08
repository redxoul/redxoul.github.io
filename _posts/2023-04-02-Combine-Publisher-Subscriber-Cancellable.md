---
title: (Combine) Publisher, Subscriber, Cancellable
author: Cody
date: 2023-04-02 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---

## `Publisher`

`Publisher Protocol`은 `Combine`의 핵심 중 하나입니다.

시간이 지남에 따라 하나 이상의 `Subscriber`들에게 일련의 값을 전달할 수 있는 `Type`에 대한 요구사항을 정의합니다.

`Publisher`는 두 가지 이벤트를 내보냅니다.

- `Element`라고 불리는 `value`
- `Completion`이벤트(Success, Failure)

0개 이상의 `value`를 내보낼 수 있지만 `Completion`이벤트는 하나만 내보낼 수 있습니다. `Completion`이벤트를 내보내고 완료되고 나면 더 이상 아무 이벤트도 내보낼 수 없습니다.

가장 간단하게 만들어볼 수 있는 `Publisher`는 `Just`입니다.

```swift
// 1. 단일 값으로 publisher를 생성할 수 있는 Just를 이용하여 publisher를 생성
let just = Just("Hello world!")
```

`Just`를 `option-click`해보면 '각 `subscriber`에게 `output`을 한번 내보낸 후 완료하는 `publisher`'라고 정의되어 있습니다.

`Just Publisher`는 `단일 value`를 받아서 `subscribe`하자마자 `value`를 내보냅니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c5b069e9-f99b-4070-afea-5d9240649e81){: width="50%" height="50%"}

이제 위의 `Just`로부터 값을 받으려면 `Subscriber`가 필요합니다.

## `Subscriber`

`Subscriber`는 `Publisher`로부터 입력을 받을 수 있는 타입의 요구사항을 정의하는 `Protocol`입니다.

`Subscribe`하여 `Publisher로부터` 값을 받을 수 있는 방법은 `sink(_:_:)`와 `assign(to:on:)`이 있습니다.

먼저 `sink`를 확인하기 위한 코드를 위 `Just` 아래에 추가합니다.

```swift
// 1. 단일 값으로 publisher를 생성할 수 있는 Just를 이용하여 publisher를 생성
let just = Just("Hello world!")

// (추가)
// 2. publisher에 대한 subscription을 만들고, 받은 각각의 이벤트에 대한 메세지를 print
_ = just
    .sink(
        receiveCompletion: {
            print("Received completion", $0)
        },
        receiveValue: {
            print("Received value", $0)
        })
```

`sink(_:_:) Operator`는 `Publisher`가 방출하는 값들을 계속 받습니다(=무제한 demand).

`sink`는 `completion`이벤트(success 혹은 failure)를 받아 핸들링하는 `receiveCompletion 클로저`와

받은 `value`를 처리하는 `receiveValue 클로저`를 제공합니다.

위 코드를 실행하면 아래와 같이 출력합니다.

```
Received value Hello world!
Received completion finished
```

예제에 코드를 더 추가해 보면

```swift
// 1. 단일 값으로 publisher를 생성할 수 있는 Just를 이용하여 publisher를 생성
let just = Just("Hello world!")

// 2. publisher에 대한 subscription을 만들고, 받은 각각의 이벤트에 대한 메세지를 print
_ = just
    .sink(
        receiveCompletion: {
            print("Received completion", $0)
        },
        receiveValue: {
            print("Received value", $0)
        })

// (추가)
_ = just
    .sink(
        receiveCompletion: {
            print("Received completion (another)", $0)
        },
        receiveValue: {
            print("Received value (another)", $0)
        })
```

각각의 `subscriber`가 모두 `just`로부터 `value`를 받고 `completion`이벤트를 받는 것이 확인됩니다.

```
Received value Hello world!
Received completion finished
Received value (another) Hello world!
Received completion (another) finished
```

두 번째로 `subscribe`할 수 있는 방법은 `assign(to:on:)`입니다.

`assing(to:on:) operator`를 사용하면 수신된 값을 객체의 `KVO(Key-Value Observing)` 호환 프로퍼티에 할당(`assign`)할 수 있습니다.

```swift
// 1. didSet을 가진 클래스를 정의
class SomeObject {
    var value: String = "" {
        didSet {
            print(value)
        }
    }
}

// 2. 해당 클래스의 인스턴스 생성
let object = SomeObject()

// 3. String 배열에서 publisher를 만듦
let publisher = ["Hello", "world!"].publisher

// 4. publisher를 subscibe하여 받은 각 값을 object의 value 프로퍼티에 할당(assign)
_ = publisher
    .assign(to: \.value, on: object)
```

```
// (출력)
Hello
world!
```

이런 `assing(to:on:)`의 특성으로 `Label`, `TextView`, `Checkbox` 등 여러 `UI Component`들에 직접 값을 할당할 수 있기 때문에 `UIKit`의 특정 `property`와 바인딩하기 좋습니다.

```swift
let publisher = ["Hello", "world!"].publisher

let label = UILabel()
label.text = ""

_ = publisher
    .assign(to: \.text!, on: label) // asign으로 UILabel.text에 할당

PlaygroundPage.current.liveView = label
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/af82255e-8abb-4922-90f7-9b0dde2ccf0a)

`assign(to:on:)`의 변형으로 `assign(to:)`가 있습니다.

`@published` 프로퍼티 래퍼로 마킹된 또 다른 프로퍼티를 통해서 `publisher가 내보낸 값을 다시 publish(republish)`하는데 사용할 수 있습니다.

아래 예시에서 `SomeObject` 클래스의 `value` 프로퍼티는 `@published` 프로퍼티 래퍼로 마킹하여,

변경된 값을 `sink`를 통해 방출하기도 하고, 또 다른 `publisher`의 `assign(to:)`를 통해서 값을 받기도 합니다.

```swift
// 1. @Published가 달린 프로퍼티를 가진 클래스를 정의
// 이 프로퍼티는 보통 프로퍼티로 접근할 수 있고, 값에 대한 publisher도 생성 가능
class SomeObject {
    @Published var value = 0
}

let object = SomeObject()

// 2. @Published 프로퍼티에 $접두사를 사용하여 기본 publisher에 접근하고, subscribe하여 수신된 값을 print
object.$value
    .sink {
        print($0)
    }

// 3. Int publisher를 생성하고, object의 값 publisher에 Int publisher가 내보내는 값을 할당(assign)
// &접두사는 프로퍼티에 대한 inout 참조를 나타내기 위해 사용
(0..<10).publisher
    .assign(to: &object.$value)
```

```
// (출력)
0 // sink와 동시에 초기값 0을 받음
0 // (0..<10).publisher를 통해 받기 시작
1
2
3
4
5
6
7
8
9
```

`@published` 마킹된 프로퍼티(`value`)가 `deinitialize`될 때, `assign(to:) operator`는 내부적으로 라이프사이클을 관리하고 `subscription`을 취소시키기 때문에 `AnyCancellable`을 반환하지 않습니다.

(= `@published` 프로퍼티에 `assign(to:)`하는 것이 `Set<AnyCancellable>`에 `store`하는 것과 동일한 효과)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1b34a429-6362-4d89-a578-a68b300ac8e4){: width="50%" height="50%"}
AnyCancellable을 반환하는 assing(to:on:)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/507ee99d-af42-4ab5-bda5-39edf7b53b4f){: width="50%" height="50%"}
아무것도 반환하지 않는 대신 published의 Publisher를 inout 파라미터로 받는 assign(to:)

또 다른 예로 아래와 같은 경우,

`@published word`를 `assign(to:\.word, on: self)`에서도 사용하고, 반환된 `AnyCancellable`을 `store`로 `self`의 `subscriptions`에 다시 저장함으로써 `strong`한 참조 사이클이 될 수가 있습니다.

```swift
class SomeObject {
    @Published var word: String = ""
    var subscriptions = Set<AnyCancellable>()

    init() {
        ["A", "B", "C"].publisher
            .assign(to: \.word, on: self)
            .store(in: &subscriptions)
    }
}
```

이런 경우 아래와 같이 `assign(to:on:)`을 `assign(to:)`로 변경해 줌으로써 이런 문제를 방지할 수 있습니다.

(= `word`가 `deinitialize`되면서 `assign(to:)`의 참조도 함께 해제)

```swift
class SomeObject {
    @Published var word: String = ""

    init() {
        ["A", "B", "C"].publisher
            .assign(to: &$word)
    }
}
```

## `Cancellable`

`Publisher`를 `subscribe(sink, assign)`하면 `AnyCancellable` 인스턴스를 반환하고, 해당 `AnyCancellable` 인스턴스는

1. 명시적으로 `cancel()` 메서드를 호출하거나,
2. `Publisher`가 `complete`이벤트`(success, failure)`를 내보내거나,
3. 메모리 관리로 인스턴스가 `deinitialize될` 때 `Subscription`이 취소됩니다.

그동안 이 글에서 들었던 예시(Just, Int 혹은 String 시퀀스 배열)들은 모두 한 블럭 내에서 `Subscription`을 하고 즉시 방출된 값을 소비하고 `complete`했기 때문에 방출된 `AnyCancellable`을 따로 `store`할 필요는 없었습니다.

하지만 그 외에 `Subscribe`하는 즉시 값을 방출하지 않는 커스텀한 `Publisher`를 생성(사용자 액션, URLSession 등)하고

이를 `Subscribe`하여 방출된 `AnyCancellable`인 경우 반드시 `store`로 `AnyCancellable 컬렉션`에 저장을 해야 해당 블럭을 벗어났을 때에도 `Subscription`이 유지가 됩니다.