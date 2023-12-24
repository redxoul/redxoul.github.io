---
title: (Combine) Future
author: Cody
date: 2023-05-05 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---
이전 글([[Combine] Publisher, Subscriber, Cancellable](https://swiftycody.github.io/posts/Combine-Publisher-Subscriber-Cancellable/))에서 `Subscribe`하자마자 단일값을 방출하고 `Complete`시키는 `Publisher`의 예시로 `Just`를 작성했었습니다.

`Future`를 사용하면 단일값을 비동기적으로 생성하고 `Complete`하는 `Publisher`를 정의할 수 있습니다.

아래는 `Int`, `Never`의 `Future`를 생성하여 반환하는 `factory`함수입니다.

(`Never`는 `failure`를 절대 반환하지 않음. [이전글](https://swiftycody.github.io/posts/Combine-Publisher-Subscriber-Custom-Subscriber/) 참조)

```swift
func futureIncrement(
    integer: Int,
    afterDelay delay: TimeInterval) -> Future<Int, Never> {

    }
```

`Future<Int, Never>`를 반환하는 함수 본문을 채워봅니다.

함수 호출부가 전달해 준 `afterDelay`값만큼 `delay`시킨 후, `integer`값을 증가시킨 값을 `promise`를 실행시킴으로써 방출시킵니다.

`Future`는 비동기적으로 단일값을 전달시킬 `promise`의 실행절차를 정의하는 클로저를 받는 `Publisher`라고 생각하면 됩니다.

```swift
func futureIncrement(
    integer: Int,
    afterDelay delay: TimeInterval) -> Future<Int, Never> {
        Future<Int, Never> { promise in // 추가
                        DispatchQueue.global().asyncAfter(deadline: .now() + delay) {
                promise(.success(integer + 1))
            }
        }
    }
```

그런데 `promise`의 인자를 보아하니 `Result`타입을 닮아있어 익숙하죠?

`Future`의 정의를 따라가보면 `promise`는 `(Result<Output, Failure>) -> Void` 클로저로 `typealias` 정의된 `Promise`의 인자값으로, `Result`타입이 맞습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/27ac923d-6ee1-4b6a-943f-c87537329aa4)

위에서 정의한 `futureIncrement(integer:afterDelay:)`를 사용해봅니다.

생성된 `Publisher`를 저장하기 위한 `Set`도 추가해줍니다.

```swift
var subscriptions = Set<AnyCancellable>() // 추가

func futureIncrement(
    integer: Int,
    afterDelay delay: TimeInterval) -> Future<Int, Never> {
        Future<Int, Never> { promise in
            DispatchQueue.global().asyncAfter(deadline: .now() + delay) {
                promise(.success(integer + 1))
            }
        }
    }

// 추가
// 1. 위 factory함수(futureIncreament)를 사용하여 3초 delay후 전달한 Int를 증가시키는 Future를 생성
let future = futureIncrement(integer: 1, afterDelay: 3)

// 2. 받은 값과 completion이벤트를 subscribe 및 print하고,
// 생성된 subscription을 subscriptions 세트에 저장
future
    .sink(
        receiveCompletion: { print($0) },
        receiveValue: { print($0) }
    )
    .store(in: &subscriptions)
```

```
// 출력
2
finished
```

위 코드에 `Future` 내부에 `print`를 추가하고, 두번째 `subscription`을 추가해봅니다.

```swift
var subscriptions = Set<AnyCancellable>()

func futureIncrement(
    integer: Int,
    afterDelay delay: TimeInterval) -> Future<Int, Never> {
        Future<Int, Never> { promise in
            print("Original") // 추가

            DispatchQueue.global().asyncAfter(deadline: .now() + delay) {
                promise(.success(integer + 1))
            }
        }
    }

let future = futureIncrement(integer: 1, afterDelay: 3)

future
    .sink(
        receiveCompletion: { print($0) },
        receiveValue: { print($0) }
    )
    .store(in: &subscriptions)

future // 추가
    .sink(
        receiveCompletion: { print("Second", $0) },
        receiveValue: { print("Second", $0) }
    )
    .store(in: &subscriptions)
```

```
// 출력
Original
2
finished
Second 2
Second finished
```

출력을 보면 `subscription`이 발생하기 전에 `Future`는 `promise`를 한번만 실행시켜 '`Original`'을 출력하는 것을 볼 수 있습니다.

이는 `Future`가 만들어지자마자 실행이 된다는 의미입니다.

그 후에는 `promise`를 재실행시키지 않고 `output`을 `share`한다는 것을 알 수 있습니다.

> `Future`는 비동기적으로 단일값을 전달시킬 `promise`의 실행절차를 정의하는 클로저를 받는 `Publisher`.  
(`promise`는 `(Result<Output, Failure>) -> Void` 클로저로 정의된 `Promise`의 인자값)  
`Future`는 만들어지자마자 실행이 되어 `output`을 만들고, 여러 `subscription`이 생겨도 한번 만들어진 동일한 `output`을 `share` 받음.
{: .prompt-tip }