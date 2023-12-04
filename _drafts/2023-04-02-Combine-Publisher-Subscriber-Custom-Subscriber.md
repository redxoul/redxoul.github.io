---
title: (Combine) Publisher와 Subscriber 프로토콜 사이 흐름의 이해. 그리고 Custom Subscriber
author: Cody
date: 2023-04-02 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---


## Publisher와 Subscriber 프로토콜 사이 흐름

아래는 **`Publisher`**와 **`Subscriber`** 사이의 흐름을 나타내는 UML다이어그램입니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d1eb0a22-52d6-4be6-b8a9-54dd3b5c3792)

1. **subscriber**가 **publisher**를 **subscribe**합니다.
2. **publisher**가 **subscription**을 생성해서 **subscriber**에게 제공합니다.
3. **subscriber**가 **value**를 요청합니다.
4. **publisher**가 **value**를 보냅니다.
5. **publisher**가 **completion**을 보냅니다.

위 흐름에 맞춰서 아래의 **Publisher**와 **Subscriber**의 프로토콜을 살펴봅시다.

먼저 **`Publisher`** 프로토콜을 살펴보면 아래와 같습니다.

```swift
public protocol Publisher {
    // [1] Publisher가 생성할 수 있는 값의 타입
    // = Subscriber의 Input이 맞춰야 할 타입
    associatedtype Output

    // [2] publisher가 생성할 수 있는 Error.
    // Publisher가 Error를 생성하지 않도록 보장하는 경우 Never
    associatedtype Failure : Error

    // [4] subscribe(_:)의 구현부에서 receive(subscriber:)을 호출하여 Subscriber를 Publisher에 연결.
    // 즉, subscription을 생성
    func receive<S>(subscriber: S)
    where S: Subscriber,
          Self.Failure == S.Failure,
          Self.Output == S.Input
}

extension Publisher {
    // [3] Subscriber가 Publisher의 subscibe(_:) 호출을 통해 전달됨
    public func subscribe<S>(_ subscriber: S)
    where S : Subscriber,
          Self.Failure == S.Failure,
          Self.Output == S.Input
}
```

위 **`Publisher`**의 **`subscribe(_:)`** 메서드가 **다이어그램에서 (1)번 과정**에서 호출됩니다.

**`Subscriber`** 프로토콜을 보면 아래와 같습니다.

잠시 후 **Custom Subscriber**를 만들 때 따라야 하는 프로토콜입니다.

```swift
public protocol Subscriber: CustomCombineIdentifierConvertible {
    // [1] Subscriber가 받을 수 있는 값의 타입
    associatedtype Input

    // [2] Subscriber가 받을 수 있는 Error 타입
    // Subscriber가 Error를 받지 않으면 Never
    associatedtype Failure: Error

    // [3] Publisher는 Subscriber의 receive(subscription:)을 호출하여
    // subscription을 제공
    func receive(subscription: Subscription)

    // [4] Publisher는 Subscriber의 receive(_:)를 호출하여
    // 방금 publish된 새 값을 Subscriber에게 보냄
    func receive(_ input: Self.Input) -> Subscribers.Demand

    // [5] Publisher는 Subscriber의 receive(completion:)을 호출하여
    // 정상적으로 혹은 Error로 인해 값 생성이 완료되었음을 호출
    func receive(completion: Subscribers.Completion<Self.Failure>)
}
```

위 **Subscriber** 프로토콜을 보면

**[3]**에서 **Publisher** 내부에서 **Subscriber**에 구현된 **receive(subscription:)**을 호출하여 **Subscription**을 제공하는 것을 볼 수 있습니다.

이 **Subscription**이 **Publisher**와 **Subscriber**의 연결고리입니다. **(다이어그램에서 (2)번 단계)**

이 **Subscription** 프로토콜을 보면 아래와 같습니다.

```swift
public protocol Subscription: Cancellable, CustomCombineIdentifierConvertible {
    func request(_ demand: Subscribers.Demand)
}
```

**Cancellable**프로토콜을 준수하는 **Subscription**은 **`request(_:)`**를 구현하도록 되어있고, 

**Publisher**가 위 **Subscriber** 프로토콜의 **`receive(subscription:)`** 를 호출했을 때 넘겨주는 **subscription**의 **`request(_:)`**메서드를 호출시켜 **Subscription**을 통해 **얼마나 값을 수요(`Demand`)할지에 대한 초기값과 함께 요청**(request)합니다.**(다이어그램에서 (3)번 단계)**

**`Demand`**를 따라가보면 **`.unlimited`**, **`.none`**, **`.max(_:)`** 이렇게 3가지를 받을 수 있음을 알 수 있습니다.

기본적으로는 0개를 받는데,

**`.unlimited`**를 설정해 주면 **무제한**으로 받고,

**`.none`**으로 하면 **아무 값도 받지 않**고,

**`.max(_:)`**로 하면 **+n개**만큼 받습니다(음수를 넣으면 **fatal error**가 일어나니 주의)

!https://blog.kakaocdn.net/dn/bFmvRL/btr7mumJ8nR/BkRhgyYSdKLiQFxMwCKy51/img.png

그리고 **[4]**에서는 **Publisher**가 **Subscriber**에 구현된 **`receive(_:) -> Subscribers.Demand`** 를 호출하여 **Input값(Publisher의 Output)을 전달**하면서 동시에 앞으로 받을 **`Demand`**(수요)를 더 늘릴지를 전달받습니다.**(다이어그램에서 (4)번 단계)**

위 **[3]**에서 **Demand**의 초기값을 정해주었더라도, **[4]**에서 유동적으로 앞으로 얼마나 더 받을지를 정해줄 수 있습니다.

이 케이스는 아래에서 **Custom Subscriber**를 만들 때 예시로 작성할 예정입니다.

마지막**[5]**으로 **Publisher**가 모든 값을 내보내고 나면 **Subscriber**의 **receive(completion:)**을 호출하여 **completion**을 전달하고 더 이상 값을 보내지 않음을 알립니다.
**(다이어그램에서 (5)번 단계)**

## **Custom Subscriber**

아래는 **`Int`**를 **`Input`**으로 받고, **`error`**타입은 **`Never`**(Failure를 받지 않는)인 **Custom Subscriber**입니다.

이 **IntSubscriber**는 **Publisher**로부터 최대 **3**개의 값만 받습니다.

```swift
// 1. 시퀀스 배열의 publisher 프로퍼티를 통해 Int의 Publisher를 생성
let publisher = [0, 1, -1, -2, 2, -3, 3, -4, 4].publisher

// 2. Custom Subscriber로 IntSubscriber 정의
final class IntSubscriber: Subscriber {
    // 3. typealias를 구현
    typealias Input = Int   // subscriber가 Int입력을 받을 수 있으며
    typealias Failure = Never   // Error을 받지 않도록 지정

    // 4. publisher로부터 호출할 receive(subscription:)으로 시작하여 필요한 메서드를 구현
    // 이 메서드에서 subscriber가 subscribe시 최대 3개의 값을 수신할 것을 지정하는 subscription의 .request(_:)를 호출
    func receive(subscription: Subscription) {
        // .max(3)은 기본 max값 0에 +3하여 최대 3개를 받겠다는 의미
        subscription.request(.max(3))
    }

    // 5. 받은 input을 print. subscriber가 Demand를 조정하지 않음을 나타내는 .none을 반환.
    func receive(_ input: Int) -> Subscribers.Demand {
        print("Received value", input)
        return .none    // 기존 max(여기서는 3)값대로 받겠다는 의미 == .max(0)과 동일
    }

    // 6. completion 이벤트를 print
    func receive(completion: Subscribers.Completion<Never>) {
        print("Received completion", completion)
    }
}

// publisher가 어떤것을 publish하려면 subscriber가 필요
let subscriber = IntSubscriber()

publisher.subscribe(subscriber)
```

```
// (출력)
Received value 0
Received value 1
Received value -1
```

출력값을 보면 **Publisher**가 보낸 값을 앞의 **3**개 **[0, 1, -1]**까지만 받았습니다.

**[0, 1, -1, -2, 2, -3, 3, -4, 4].publisher**는 마지막 값 **4**까지 받아야 **Subscriber**의 **`receive(completion:)`**을 호출하여 **complete**이벤트를 보내는 **Publisher**인데,

**Subscriber**가 최대 **3**개만 받는다고 선언했기 때문에 **completion**을 받지 않은 것을 확인할 수 있습니다.

만약 **Publisher**가 보낸 값을 무제한으로 받고 싶다면 **`receive(subscription:)`**에서 **`.max(3)`**을 **.`unlimited`**로 바꿔주면 됩니다.

```swift
    func receive(subscription: Subscription) {
            // .unlimited 는 Publisher가 주는 모든 값을 받겠다는 의미
        subscription.request(.unlimited)
    }
```

바꿔주면 아래처럼 모든 값을 받고, **completion**까지 받는 것이 확인됩니다.

```swift
// (.max(3)을 .unlimited로 바꿔주었을 때의 출력)
Received value 0
Received value 1
Received value -1
Received value -2
Received value 2
Received value -3
Received value 3
Received value -4
Received value 4
Received completion finished
```

다시 **`.max(3)`**으로 되돌리고,

이번엔 **Publisher**가 주는 **`Int`값을 3개만 받되 0 이상의 값만 받아서 처리**하고 싶을 수가 있습니다.

이럴 땐 **`receive(_:) -> Subscribers.Demand`** 메서드를 아래와 같이 수정해서 처리할 수 있습니다.

```swift
func receive(_ input: Int) -> Subscribers.Demand {
    if input < 0 {      // Publisher에게 받은 input값이 음수일 땐
        return .max(1)  // 기존 max값에 +1 하겠다는 의미 = 1개를 더 받음
    }
    else {
        print("Received value", input)
        return .none    // 기존 max값대로 받겠다는 의미
    }
}
```

```
// (출력)
Received value 0
Received value 1
Received value 2
```

처음처럼 **Publisher**로부터 **최대 3개**만 받지만, 음수를 받을 때마다 **max+1**개를 더 받아서 처리되는 것을 확인할 수 있습니다.

만약 **publisher**를 아래와 같이 **String** 시퀀스로 바꿔주면?

```swift
let publisher = ["A", "B", "C", "D", "E", "F"].publisher
```

당연하게도 **Publisher** 프로토콜에서 봤던 **`public func subscribe<S>(_ subscriber: S) where S : Subscriber, Self.Failure == S.Failure, Self.Output == S.Input`** 조건에 맞지 않기 때문에 **`subscribe(_:)`**에서 에러가 발생합니다.

```swift
publisher.subscribe(subscriber) // No exact matches in call to instance method 'subscribe'
```