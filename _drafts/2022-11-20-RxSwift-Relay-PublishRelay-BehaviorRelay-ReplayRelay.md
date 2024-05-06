---
title: (RxSwift) Relay - PublishRelay, BehaviorRelay, ReplayRelay
author: Cody
date: 2022-11-20 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---
이번엔 `Subject`에서 파생된 `Relay`의 정리글입니다.

`Relay`를 설명하기 전에 `Relay`의 등장배경을 먼저 정리를 하자면.

우리는 `RxSwift`를 앞서 정리한 예제들과 같이 데이터를 다룰때 뿐만 아니라,

iOS 앱을 개발할 때 `UI`와 바인딩을 하여 활용을 하게 될텐데요.

앞에서 배웠던 `Observable`, `Subject`들을 그대로 사용했을 때 이들의 특징 때문에 문제가 발생할 수 있습니다.

`Observable`과 `Subject`는 `error`가 발생했을 때 시퀀스가 종료되고, 종료된 시퀀스는 재활용을 할 수가 없습니다.

시퀀스가 종료되면 `UI`와의 바인딩도 종료되어 더이상 동작을 하지 않게 됩니다.

그래서 `error`와 `complete`이벤트가 발생하지 않는,

`절대 종료되지 않음이 보장이 되는 Subject`가 필요해졌는데

이런 필요에 의해 `Relay`가 등장하게 됩니다.

---

정리를 하자면, `Relay`는 절대로 종료(`error`, `complete`)되지 않는 `Subject`입니다.

그리고 오직 값을 받을 수만 있기 때문에 `next`이벤트 대신 `accept`이벤트를 방출합니다.

`Subject`의 변형인 `Relay`에는 `PublishRelay`, `BehaviorRelay`, `ReplayRelay`가 있습니다.

이름에서도 눈치챌 수 있듯이 `PublishSubject`, `BehaviorSubject`, `ReplaySubject`의 `Relay` 버전입니다.

(AsyncSubject는 complete이벤트가 발생해야만 마지막 Element를 방출하는 Subject니까

complete, error가 발생하지 않는 AsyncRelay같은 건 당연히 존재하지 않습니다)

그래서 주요 특징이 모두 동일하므로 간단히 정리하고 넘어가보겠습니다.

## PublishRelay

`PublishSubject`와 동일하지만, `accept`이벤트만 방출하는 `Relay`입니다.

```swift
print("- - - - - PublishRelay  - - - - -")
let publishRelay = PublishRelay<String>()

publishRelay.accept("(1) first accept")

publishRelay
    .subscribe(onNext: {
        print("sub: ", $0)
    })
    .disposed(by: disposeBag)

publishRelay.accept("(2) second accept")
```

(출력)

`PublishSubject`와 마찬가지로 `subscribe`한 이후의 이벤트만 받는 것을 확인.

```
- - - - - PublishRelay  - - - - -
sub:  (2) second accept
```

---

## BehaviorRelay

`BehaviorSubject`와 동일하지만, `accept`이벤트만 방출하는 `Relay`입니다.

```swift
print("- - - - - BehaviorRelay  - - - - -")
let behaviorRelay = BehaviorRelay<String>(value: "(0) init value")

behaviorRelay
    .subscribe(onNext: {
        print("sub1: ", $0)
    })
    .disposed(by: disposeBag)

behaviorRelay.accept("(1) first accept")

behaviorRelay
    .subscribe(onNext: {
        print("sub2: ", $0)
    })
    .disposed(by: disposeBag)

behaviorRelay.accept("(2) second accept")
```

(출력)

첫번째 ``observer``는 ``subscribe``하자마자 초기값인 ``(0) init value``를 받고,

두번째 ``observer``는 ``subscribe``하자마자 마지막 방출된 값인 ``(1) first accept``를 받는 것을 확인됩니다.

```
- - - - - BehaviorRelay  - - - - -
sub1:  (0) init value
sub1:  (1) first accept
sub2:  (1) first accept
sub1:  (2) second accept
sub2:  (2) second accept
```

---

## ReplayRelay

`ReplaySubject`와 동일하지만, `accept`이벤트만 방출하는 `Relay`입니다.

```swift
print("- - - - - ReplayRelay  - - - - -")
let replayRelay = ReplayRelay<String>.create(bufferSize: 2)

replayRelay.accept("(1) first accept")

replayRelay
    .subscribe(onNext: {
        print("sub1: ", $0)
    })
    .disposed(by: disposeBag)

replayRelay.accept("(2) second accept")

replayRelay
    .subscribe(onNext: {
        print("sub2: ", $0)
    })
    .disposed(by: disposeBag)
```

(출력)

`buffer`가 `2`인 `ReplayRelay`

첫번째 `observer`는 `subscribe`하자마자 `buffer`에 쌓인 `(1)` 이벤트를 받고,

두번째 `observer`는 `subscribe`하자마자 `buffer`에 쌓인 `(1)`, `(2)` 이벤트를 모두 받는 것을 확인할 수 있습니다.

```
- - - - - ReplayRelay  - - - - -
sub1:  (1) first accept
sub1:  (2) second accept
sub2:  (1) first accept
sub2:  (2) second accept
```