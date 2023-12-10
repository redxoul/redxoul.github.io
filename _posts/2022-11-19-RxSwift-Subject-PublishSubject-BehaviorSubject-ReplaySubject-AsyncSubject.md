---
title: (RxSwift) Subject - PublishSubject, BehaviorSubject, ReplaySubject, AsyncSubject
author: Cody
date: 2022-11-19 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---
`Observable`은 `create`할 때부터, 어떻게 데이터를 방출할지 정해져 있는 시퀸스입니다.

`create`된 `Observable`을 `subscribe`함으로써 `observer`는 값을 받거나 받은 값을 가공할 수만 있는 `read-only`와 같습니다.

`Subject`는 `Observable`이자 `Observer`역할을 모두 할 수 있고,

`create` 이후에도 `Observable`의 외부에서 동적으로 원하는 값을 추가할 수 있는 시퀀스입니다.

`Subject`에는 `PublishSubject`, `BehaviorSubject`, `ReplaySubject`, `AsyncSubject`가 있는데,

주로 `PublishSubject`, `BehaviorSubject`가 많이 쓰입니다.

하나씩 정리해보겠습니다.

---

### PublishSubject

이름과 같이 정보를 받아서 `Observer`에게 `Publish(발행)`를 하는 `Subject`입니다.

`Observer`가 `subscribe`한 시점부터

`subscribe`를 `dispose`하거나 종료이벤트(`completed`, `error`)를 받을 때까지

새로운 이벤트를 받고 싶을 때 유용합니다.

`PublishSubject`는 `Subscribe`하고 나서부터 이벤트를 받을 수 있기 때문에 그전에 발생된 값은 받지 못합니다.

![제일 윗줄은 PublishSubject, 그 아랫줄들은 이를 subscribe한 Observer들](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/168c45c6-ec2c-4c29-b529-050135554961)
제일 윗줄은 PublishSubject, 그 아랫줄들은 이를 subscribe한 Observer들

`PublishSubject` 동작의 예제를 보겠습니다.

```swift
print("- - - - - PublishSubject - - - - -")
let disposeBag = DisposeBag()

let publishSubject = PublishSubject<String>()
publishSubject.onNext("(1) 안녕하세요!") // subscribe하기 전 이벤트 발생
let subscriber1 = publishSubject
    .subscribe(onNext: {
        print("sub1: \($0)")
    })
    .disposed(by: disposeBag)

publishSubject.onNext("(2) Hello!?")
publishSubject.onNext("(3) Hello!?!?")
```

(출력) `subscribe`한 후에 방출된 이벤트만 받는 것을 확인할 수 있습니다.

```
- - - - - PublishSubject - - - - -
sub1: (2) Hello!?
sub1: (3) Hello!?!?
```

이번엔 `subscriber1`을 `dispose`시키고 `subscriber2`를 `subscribe`해봅니다.

```swift
print("- - - - - PublishSubject - - - - -")
let disposeBag = DisposeBag()

let publishSubject = PublishSubject<String>()
publishSubject.onNext("(1) 안녕하세요!")

let subscriber1 = publishSubject
    .subscribe(onNext: {
        print("sub1: \($0)")
    })
//    .disposed(by: disposeBag)
publishSubject.onNext("(2) Hello!?")
publishSubject.onNext("(3) Hello!?!?")

subscriber1.dispose() // subscriber1을 취소
let subscriber2 = publishSubject
    .subscribe(onNext: {
        print("sub2: \($0)")
    })
    .disposed(by: disposeBag)

publishSubject.onNext("(4) Hello!?!?!?")
publishSubject.onCompleted() // publishSubject 완료
publishSubject.onNext("(5) Hello!?!?!?!?")
```

(출력)

`dispose`된 `subscriber1`은 이후에 방출된 `(4)`이벤트를 받을 수 없게 된 것을 확인할 수 있습니다.

`Observable`과 마찬가지로 `Subject`또한 `complete`이벤트 이후로는 값을 방출하지 않기 때문에 `(5)`이벤트는 방출되지 않음을 확인할 수 있습니다.

```
- - - - - PublishSubject - - - - -
sub1: (2) Hello!?
sub1: (3) Hello!?!?
sub2: (4) Hello!?!?!?
```

이미 `complete`된 이벤트를 `subscribe`하면?

```swift
// - - - (위 코드 바로 아래에) - - -
publishSubject
    .subscribe {
        print("sub3: ", $0.element ?? $0)
    }
    .disposed(by: disposeBag)

publishSubject.onNext("(6) Hello!?!?!?!?!?")
```

(출력) subscribe하면 바로 `completed`됨이 확인됩니다.

```
sub3:  completed
```

---

### BehaviorSubject

`BehaviorSubject`는 `PublishSubject`와 거의 동일하고,

새로 `subscribe`한 `observer`에게 마지막 `next`이벤트를 replay시켜준다는 특징이 추가된 `Subject`입니다.

`subscribe`하자마자 마지막 `element`를 방출시켜줘야 하기 때문에, `초기값`이 필요합니다.

초기값을 넣을 수 없다면 `PublishSubject`를 사용하거나 `element`를 옵셔널로 모델링해주어야 합니다.

`BehaviorSubject`는 미리 fetch해 온 데이터로 화면을 그리고, 후에 다시 fetch한 데이터로 화면을 갱신하는 것과 같은 동작에 유용합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/25bc46cc-2e57-406d-9860-8be032526393)

`BehaviorSubject` 동작의 예제를 보겠습니다.

```swift
print("- - - - - BehaviorSubject - - - - -")
let behaviorSubject = BehaviorSubject<String>(value: "(0) init")

behaviorSubject
    .subscribe {
        print("sub1: ", $0.element ?? $0)
    }
    .disposed(by: disposeBag)

behaviorSubject.onNext("(1) first")

behaviorSubject
    .subscribe {
        print("sub2: ", $0.element ?? $0)
    }
    .disposed(by: disposeBag)
```

(출력)

첫번째 `observer`는 `subscribe`하자마자 초기값 `(0) init`을 받고

그 다음 `behaviorSubject`에서 방출된 `(1) first`을 받고

두번째 `observer`는 `subscribe`하자마자 `behaviorSubject`가 마지막 방출한 `(1) first`를 받는 것을 확인할 수 있습니다.

```
- - - - - BehaviorSubject - - - - -
sub1:  (0) init
sub1:  (1) first
sub2:  (1) first
```

그리고 `BehaviorSubject`는 마지막 방출한 값을 `value()` 메서드를 통해 가져올 수 있습니다.

```swift
if let value = try? behaviorSubject.value() { // 최신 값을 가져올 수 있음
    print("value: ", value)
}
```

(출력)

```
value:  (1) first
```

### ReplaySubject

`ReplaySubject`는 `BehaviorSubject`와 유사하게 `subscribe`하기 이전에 방출되었던 값을 받을 수 있습니다.

`BehaviorSubject`는 `buffer`가 1개였다면,

`ReplaySubject`는 세팅한 `bufferSize` 만큼 `subscribe` 이전에 방출된 `Element`들이 유지가 됩니다.

그리고 `ReplaySubject`는 초기값 설정이 없고, 이전에 방출되었던 값이 없었다면 `subscribe`시 아무런 값을 받지 않습니다.

버퍼에 쌓인 만큼 메모리를 차지하고 있어서

`bufferSize`와 `Element`에 따라 메모리에 엄청난 부하를 줄 수 있기 때문에

사용시 주의가 필요한 `Subject`입니다.
![bufferSize가 2인 ReplaySubject의 마블다이어그램](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b4cc5ea3-ae80-44e6-bcb3-eb9365b0e622)
bufferSize가 2인 ReplaySubject의 마블다이어그램

`ReplaySubject`의 예제를 보겠습니다.

```swift
print("- - - - - ReplaySubject - - - - -")
enum SubjectError: Error {
    case someError
}

// bufferSize가 2인 ReplaySubject
let replaySubject = ReplaySubject<String>.create(bufferSize: 2)

replaySubject.onNext("(1) first")
replaySubject.onNext("(2) second")

replaySubject
    .subscribe { // subscribe하자마자 (1), (2)를 받음
        print("sub1: ", $0.element ?? $0)
    }
    .disposed(by: disposeBag)

replaySubject.onNext("(3) third")

replaySubject
    .subscribe { // subscribe하자마자 (2), (3)을 받음
        print("sub2: ", $0.element ?? $0)
    }
    .disposed(by: disposeBag)

replaySubject.onNext("(4) fourth")
replaySubject.onError(SubjectError.someError)
replaySubject.dispose() // replaySubject가 dispose됨
replaySubject
    .subscribe { // 이미 dispose되에 error가 발생
        print("sub3: ", $0.element ?? $0)
    }
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - ReplaySubject - - - - -
sub1:  (1) first
sub1:  (2) second
sub1:  (3) third
sub2:  (2) second
sub2:  (3) third
sub1:  (4) fourth
sub2:  (4) fourth
sub1:  error(someError)
sub2:  error(someError)
sub3:  error(Object `RxSwift.(unknown context at $104ee1af0).ReplayMany<Swift.String>` was already disposed.)
```

### AsyncSubject

`AsyncSubject`는 앞서 설명한 `Subject`들과 전혀 다르게 동작하는 `Subject`입니다.

`completed`이벤트가 발생할 때까지 `observer`들은 아무 `Element`를 받을 수 없습니다.

`completed`이벤트가 발생하면 마지막으로 발생했던 `next`이벤트를 받거나,

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c5850846-79d1-4e8f-8696-78a73d819943)

`next`이벤트가 발생된 적이 있었더라도,

`error`이벤트로 종료가 되면 아무런 `Element`도 받지 못하고 `error`이벤트를 받고 종료됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8172aad8-b05b-4dab-a0b6-389d94cf2b59)

`AsyncSubject`의 예제입니다.

```swift
print("- - - - - AsyncSubject - - - - -")
let asyncSubject = AsyncSubject<String>()

asyncSubject
    .subscribe {
        print("sub1: ", $0.element ?? $0)
    }
    .disposed(by: disposeBag)

asyncSubject.onNext("(1) first")
asyncSubject.onNext("(2) second")
asyncSubject.onNext("(3) third")

asyncSubject.onCompleted()
```

(출력)

마지막으로 받은 `next`이벤트만 받는 것이 확인됩니다.

```
- - - - - AsyncSubject - - - - -
sub1:  (3) third
sub1:  completed
```

`AsyncSubject`의 두번째 예제입니다.

```swift
print("- - - - - AsyncSubject (2) - - - - -")
let asyncSubject2 =AsyncSubject<String>()

asyncSubject2
    .subscribe {
        print("sub2: ", $0.element ?? $0)
    }
    .disposed(by: disposeBag)

asyncSubject2.onNext("(1) first")
asyncSubject2.onNext("(2) second")
asyncSubject2.onNext("(3) third")

asyncSubject2.onError(SubjectError.someError)
```

(출력)

`next`이벤트가 세번 발생했지만 `error`로 종료가 되어 `error`이벤트만 받게 되었습니다.

```
- - - - - AsyncSubject (2) - - - - -
sub2:  error(someError)
```