---
title: (Combine) Subject
author: Cody
date: 2023-05-06 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---
`Subject`는 `non-Combine` 코드로부터 값을 받아서 `Subscriber`에게 값을 방출시키는 `Publisher`입니다.

`PassthroughtSubject`로 `value`를 전달하는 방식은 명령형 코드를 `Combine`을 통한 선언형 코드로 연결하는 좋은 방법입니다.

(`@published`를 마킹한 프로퍼티와 유사한 징검다리 역할입니다. `non-Combine` 코드로부터 값을 받아, `sink`를 통해 값을 내보낼 수 있죠)

## PassthroughSubject

( = RxSwift의 PublishSubject)

`Subject`의 가장 기본형입니다.

아래는 우선 커스텀 `Subscriber`의 예시입니다([Custom Subscriber](https://swiftycody.github.io/posts/Combine-Publisher-Subscriber-Custom-Subscriber/)는 이전글 참조)

```swift
// 1. Error 타입을 정의
enum MyError: Error {
    case test
}

// 2. String, MyError를 받는 Custom Subscriber를 정의
final class StringSubscriber: Subscriber {
    typealias Input = String
    typealias Failure = MyError

    func receive(subscription: Subscription) {
        subscription.request(.max(2))
    }

    func receive(_ input: String) -> Subscribers.Demand {
        print("Received value", input)
// 3. 받은 값을 기준으로 demand를 조절함
                return input == "World" ? .max(1) : .none
    }

    func receive(completion: Subscribers.Completion<MyError>) {
        print("Received completion", completion)
    }
}

// 4. Custom Subscriber의 인스턴스를 생성
let subscriber = StringSubscriber()
```

`String`, `MyError`를 받는 `StringSubscriber`라는 커스텀 `subscriber`입니다.

`subscribe`시 최대 2개값을 받고, `world`라는 값을 받으면 최대값 2개에서 +1씩 더 받습니다.

이어서 아래에 `PassthroughtSubject`를 생성하고, `Subscription`을 `StringSubscriber`와 `sink`를 통해 만들어봅니다.

당연하게도 `PassthroughtSubject`는 `<String, MyError>`타입으로 정의되어야 `StringSubscriber`가 받을 수 있습니다.

```swift
enum MyError: Error {
    case test
}

final class StringSubscriber: Subscriber {
    typealias Input = String
    typealias Failure = MyError

    func receive(subscription: Subscription) {
        subscription.request(.max(2))
    }

    func receive(_ input: String) -> Subscribers.Demand {
        print("Received value", input)
        return input == "World" ? .max(1) : .none
    }

    func receive(completion: Subscribers.Completion<MyError>) {
        print("Received completion", completion)
    }
}

let subscriber = StringSubscriber()

// 추가
// 5. String과 MyError가 정의된 타입의 PassthroughSubject인스턴스를 생성
let subject = PassthroughSubject<String, MyError>()

// 6. subscriber를 subject에 subscribe등록
subject.subscribe(subscriber)

// 7. sink로 또다른 subscription을 생성
let subscription = subject
    .sink(
        receiveCompletion: { completion in
            print("Received completion (sink)", completion)
        },
        receiveValue: { value in
            print("Received value (sink)", value)
        }
    )
```

`subscription`들도 설정되었으니, `subject`에 값을 넣고 방출시키기 위한 코드를 작성합니다.

`subject`로 값은 `.send()`를 통해 전달합니다.

```swift
subject.send("Hello")
subject.send("World")
```

각 `subscriber`가 `publish`될 때마다 `value`를 받아서 출력합니다.

```bash
// 출력
Received value Hello
Received value (sink) Hello
Received value World
Received value (sink) World
```

아래에 `subscription`을 취소시키는 코드를 작성하고 한번 더 값을 전달해봅니다.

```swift
subject.send("Hello")
subject.send("World")

// 추가
// 8. 두번째 구독을 cancel
subscription.cancel()

// 9. 새로운 값을 내보냄
subject.send("Still there?")
```

첫번째 `subscriber`만 값을 받아 출력되는 것을 확인할 수 있습니다.

```bash
Received value (sink) Hello
Received value Hello
Received value (sink) World
Received value World
Received value Still there?
```

이어서 `subject`로 `complete` 이벤트를 전달 후 값을 다시 전달해봅니다.

```swift
subject.send("Hello")
subject.send("World")

subscription.cancel()

subject.send("Still there?")

// (추가)
// finish 이벤트
subject.send(completion: .finished)
subject.send("How about another one?")
```

두번째 `subscription`은 이전에 `cancel`되었기 때문에 `completion`을 받을 수 없고,

첫번째 `subscriber`는 "How about another one?" 메세지를 받기전에 `complete` 이벤트를 받았기 때문에 값을 받지 못합니다.

```bash
Received value (sink) Hello
Received value Hello
Received value (sink) World
Received value World
Received value Still there?
Received completion finished
```

마지막으로 `subject.send(completion: .finished)` 바로 윗줄에 `failure` 이벤트를 전달하는 코드를 작성해봅니다.

```swift
subject.send("Hello")
subject.send("World")

subscription.cancel()

subject.send("Still there?")

subject.send(completion: .failure(MyError.test)) // 추가

subject.send(completion: .finished)
subject.send("How about another one?")
```

`Publisher`가 `failure`로 `error`를 방출하여 `Complete`되고, 더이상 값을 내보내지 않는 것을 첫번째 `subscriber`를 통해 확인할 수 있습니다.

```bash
Received value (sink) Hello
Received value Hello
Received value (sink) World
Received value World
Received value Still there?
Received completion failure(__lldb_expr_47.MyError.test)
```

> `PassthroughtSubject`는 `Subject`의 기본형으로,
`non-Combine` 코드로부터 값을 받아서 `Subscriber`에게 값을 방출시키는 `Publisher`.
{: .prompt-info }

## CurrentValueSubject

( = RxSwift의 BehaviorSubject)

때에 따라 명령형 코드에서 `Publisher`의 현재(마지막) `value`를 확인해야 할 필요가 생기는데,

이런 경우를 위해 `CurrentValueSubject`가 있습니다.

용법은 `PassthroughtSubject`와 동일하지만, 초기화시 `초기값이 필요`하다는 차이점이 있습니다.

아래는 `CurrentValueSubject`의 예시입니다.

```swift
// 1. subscription을 저장할 Set 생성
var subscriptions = Set<AnyCancellable>()

// 2. Int, Never 타입의 CurrentValueSubject 생성. 초기값은 0
let subject = CurrentValueSubject<Int, Never>(0)

// 3. subscription 생성. subject에게 값을 받아 print
subject
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions) // 4. subscriptions에 store
```

출력을 보면? `subscribe`하자마자 초기값을 받아오는 것이 확인됩니다.

```
0
```

`PassthroughSubject`처럼 `.send()`를 통해 값을 전달할 수 있습니다.

아래에 코드를 추가합니다.

```swift
var subscriptions = Set<AnyCancellable>()

let subject = CurrentValueSubject<Int, Never>(0)

subject
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

// 추가
subject.send(1)
subject.send(2)
```

출력.

```
0
1
2
```

현재(마지막) 값을 받아오는 코드를 아래에 추가합니다.

`.value`를 통해서 받아올 수 있습니다.

```swift
var subscriptions = Set<AnyCancellable>()

let subject = CurrentValueSubject<Int, Never>(0)

subject
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

subject.send(1)
subject.send(2)

// 추가
print("value: ", subject.value)
```

출력.

```
0
1
2
value:  2
```

`value`에 직접 값을 전달해보는 코드를 아래에 추가해봅니다.

```swift
var subscriptions = Set<AnyCancellable>()

let subject = CurrentValueSubject<Int, Never>(0)

subject
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

subject.send(1)
subject.send(2)

print("value: ", subject.value)

// 추가
subject.value = 3
print("value: ", subject.value)
```

출력을 보면, `value`로 값을 변경시켜도 `.send()`로 전달하는 것과 동일한 효과인 것을 알 수 있습니다.

```
0
1
2
value:  2
3
value:  3
```

이어서 `subscription`을 추가해보겠습니다. 

두번째 `subscription`에는 `sink` 위에 `.print()`가 추가되었습니다.

그리고 첫번째 `subscription`에도 `.print()`를 추가했습니다.

`.print()`를 사용할 땐 `prefix`값을 넣어서 구분시켜줄 수 있습니다.

```swift
var subscriptions = Set<AnyCancellable>()

let subject = CurrentValueSubject<Int, Never>(0)

subject
    .print("[1st subscription]") // 추가
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

subject.send(1)
subject.send(2)

print("value: ", subject.value)

subject.value = 3
print("value: ", subject.value)

// 추가
subject
    .print("[2nd subscription]")
    .sink(receiveValue: { print("2nd subscription:", $0) })
    .store(in: &subscriptions)
```

출력이 조금 복잡해지겠지만 실행시켜보면,

`subscription`의 변경이 생길 때마다 상세하게 볼 수 있는 `print`가 되는 것을 볼 수 있습니다.

```
[1st subscription]: receive subscription: (CurrentValueSubject)
[1st subscription]: request unlimited
[1st subscription]: receive value: (0)
0
[1st subscription]: receive value: (1)
1
[1st subscription]: receive value: (2)
2
value:  2
[1st subscription]: receive value: (3)
3
value:  3
[2nd subscription]: receive subscription: (CurrentValueSubject)
[2nd subscription]: request unlimited
[2nd subscription]: receive value: (3)
2nd subscription: 3
```

마지막으로 `complete`이벤트를 전달하면

```swift
// 추가
subject.send(completion: .finished)
```

두 `subscription`이 모두 종료를 전달받습니다.

```
[2nd subscription]: receive finished
[1st subscription]: receive finished
```

> `CurrentValueSubject`는 `PassthroughSubject`의 변형으로, `.value` 프로퍼티를 통해 현재(마지막) 방출된 값을 받아올 수 있음.  
이런 특성이 있기 때문에 초기화 시에 초기값을 넣어주어야 함.  
`.value` 프로퍼티로 직접 값을 넣어도 `.send()` 함수로 값을 전달하는 것과 동일한 효과(`subscriber`들에게 `value`를 방출).
{: .prompt-info }


> `Publisher`나 `Subject`를 `sink`하는 구문 전에 `.print("prefix")`를 넣어주면, `subscription`의 변경이 있을 때마다 출력을 통해 정보를 알려줌.
{: .prompt-tip }
