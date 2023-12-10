---
title: (Combine) Combine, RxSwift 각 주요요소 비교
author: Cody
date: 2023-02-07 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---

`Combine`을 익히려고 보다 보면,

`RxSwift`와 비교를 하면서 익힐 수밖에 없는 듯하여 둘의 핵심요소들의 비교를 정리해 봅니다.

(참고: https://github.com/CombineCommunity/rxswift-to-combine-cheatsheet)

### Basic Spec.

|  | RxSwift | Combine |
| --- | --- | --- |
| Deployment Target | iOS 8.0+ | iOS 13.0+ |
| Platforms supported | iOS, macOS, tvOS, watchOS, Linux | iOS, macOS, tvOS, watchOS, UIKit for Mac |
| Spec | Reactive Extensions (ReactiveX) | Reactive Streams (+ adjustments) |
| Framework Consumption | Third-party | First-party (built-in) |
| Maintained by | Open-Source / Community | Apple |
| UI Bindings | RxCocoa | SwiftUI |

최소 타겟이 `Combine`은 `SwiftUI`와 함께 `iOS13.0 이상`부터 가능합니다.

`RxSwift`는 `Swift`뿐만 아니라 다른 언어에서도 `ReactiveX` 생태계를 통해 동일한 `API`로 다룰 수 있는 `Multiplatform` `API`입니다.

---

### AnyPublisher

`RxSwift`의 `Observable`에 해당하는 요소로, `Publisher` 프로토콜을 따르는 `struct`입니다.

시간 흐름에 따라 정의된 객체를 방출하는 타입입니다.

`Observable`은 방출할 `Element` 타입을 정의해 주었는데,

`AnyPublisher`는 방출할 `Element` 뿐만 아니라 `Error`타입도 함께 지정을 해주어야 합니다.

| AnyPublisher | Observable |
| --- | --- |
| `public protocol Publisher { }` / `struct AnyPublisher: Publisher { }` (`Publisher`는 사실 Protocol이고, 실제 사용하는 것은 이 protocol을 받은 `struct AnyPublisher`) | `class Observable: ObservableType { }` |
| `AnyPublisher`는 struct로 `Value Type` | `Observable`은 class로 `Reference Type` |
| `AnyPublisher`는 `AssociatedType`으로 `Output(Data type)`과 `Failure(Error type)`을 내보냄 | `Observable`은 `Element(Data type)`을 내보냄. `Element`로 `Result` type을 사용하면 이와 유사하게 사용할 수 있음. |
| `associatedtype Output` / `associatedtype Failure: Error` | `class Observable<Element>` |
| `AnyPublisher<String, Error>` / `AnyPublisher<String, Never>` | `Observable<Result<String, Error>>`  / `Observable<String>` |

---

### AnySubscriber

`RxSwift`의 `Observer`에 해당하는 요소로, `Subscriber` 프로토콜을 따르는 `struct`입니다.

`AnyPublisher`를 `subscribe 하여` `AnyPublisher`가 방출한 이벤트를 받을 수 있는 타입니다.

---

### Core Components

`RxSwift`의 `Observer`, `Traits`, `Subject`, `Relay`에 해당하는 요소들입니다.

같은 요소도 있고, `Combine`에는 없거나, 있어도 다르게 동작하는 것도 있습니다.

| RxSwift | Combine | Notes |
| --- | --- | --- |
| Observable | Publisher |  |
| AnyObserver | AnySubscriber |  |
| Observer | Subscriber |  |
| Disposable | Cancellable | anyCancellable.store(in: &collection)을 호출하고, collection은 Array, Set, RangeReplaceableCollection이 될 수 있음. |
| DisposeBag | A collection of AnyCancellables |  |
| Single | Deferred + Future |  |
| Completable | ❌ |  |
| Maybe | Optional.Publisher |  |
| SubjectType | Subject |  |
| PublishSubject | PassthroughSubject |  |
| BehaviorSubject | CurrentValueSubject |  |
| ReplaySubject | ❌ |  |
| PublishRelay | ❌ | Combine에서는 다시 구현해서 사용할 수 있음. |
| BehaviorRelay | ❌ | Combine에서는 다시 구현해서 사용할 수 있음. |
| CompositeDisposable | ❌ |  |
| ConnectableObservableType | ConnectablePublisher |  |
| Driver | ObservableObject | 둘 다 Failure가 없음을 보장함. Driver는 Main Thread에서의 전달을 보장하고, 대신 Combine은 SwiftUI에서 Main Thread에서 전체 View 계층을 다시 그려줌. |
| Signal | ❌ |  |
| ScheduledDisposable | ❌ |  |
| SchedulerType | Scheduler |  |
| SerialDisposable | ❌ |  |
| TestScheduler | ❌ | There doesn't seem to be an existing testing scheduler for Combine code |

---

### Operators

`AnyPublisher`를 조합하거나, 가공할 때 필요한 `Operator`들입니다.

(Combine에만 있는 operator는 빨간색으로 표시했습니다)

| RxSwift | Combine | Notes |
| --- | --- | --- |
| amb() | ❌ |  |
| asObservable() | eraseToAnyPublisher() |  |
| asObserver() | ❌ |  |
| bind(to:) | assign(to:on:) | Assign은 유용하게 KeyPath를 사용함.RxSwift는 bind를 위해 Binder, ObserverType이 필요함. |
| buffer | buffer |  |
| catchError | catch |  |
| catchErrorJustReturn | replaceError(with:) |  |
| combineLatest | combineLatest, tryCombineLatest |  |
| compactMap | compactMap, tryCompactMap |  |
| concat | append, prepend |  |
| concatMap | ❌ |  |
| create | ❌ | Apple removed AnyPublisher with a closure in Xcode 11 beta 3 :-( |
| debounce | debounce |  |
| debug | print |  |
| deferred | Deferred |  |
| delay | delay |  |
| delaySubscription | ❌ |  |
| dematerialize | ❌ |  |
| distinctUntilChanged | removeDuplicates, tryRemoveDuplicates |  |
| do | handleEvents |  |
| elementAt | output(at:) |  |
| empty | Empty(completeImmediately: true) |  |
| enumerated | ❌ |  |
| error | Fail |  |
| filter | filter, tryFilter |  |
| first | first, tryFirst |  |
| flatMap | flatMap |  |
| flatMapFirst | ❌ |  |
| flatMapLatest | switchToLatest |  |
| from(optional:) | Optional.Publisher(_ output:) |  |
| groupBy | ❌ |  |
| ifEmpty(default:) | replaceEmpty(with:) |  |
| ifEmpty(switchTo:) | ❌ | Could be achieved with composition - replaceEmpty(with: publisher).switchToLatest() |
| ignoreElements | ignoreOutput |  |
| interval | ❌ |  |
| just | Just |  |
| map | map, tryMap |  |
| materialize | ❌ |  |
| merge | merge, tryMerge |  |
| merge(maxConcurrent:) | flatMap(maxPublishers:) |  |
| multicast | multicast |  |
| never | Empty(completeImmediately: false) |  |
| observeOn | receive(on:) |  |
| of | Sequence.publisher | 모든 시퀀스의 publisher property 혹은 'Publishers.Sequence(sequence:)'를 직접 사용할 수 있음. |
| publish | makeConnectable |  |
| range | ❌ |  |
| reduce | reduce, tryReduce |  |
| refCount | autoconnect |  |
| repeatElement | ❌ |  |
| retry, retry(3) | retry, retry(3) |  |
| retryWhen | ❌ |  |
| sample | ❌ |  |
| scan | scan, tryScan |  |
| share | share | There’s no replay or scope in Combine. Could be “faked” with multicast. |
| skip(3) | dropFirst(3) |  |
| skipUntil | drop(untilOutputFrom:) |  |
| skipWhile | drop(while:), tryDrop(while:) |  |
| startWith | prepend |  |
| subscribe | sink |  |
| subscribeOn | subscribe(on:) | RxSwift uses Schedulers. Combine uses RunLoop, DispatchQueue, and OperationQueue. |
| take(1) | prefix(1) |  |
| takeLast | last |  |
| takeUntil | prefix(untilOutputFrom:) |  |
| throttle | throttle |  |
| timeout | timeout |  |
| timer | Timer.publish |  |
| toArray() | collect() |  |
| window | collect(Publishers.TimeGroupingStrategy) | Combine has a TimeGroupingStrategy.byTimeOrCount that could be used as a window. |
| withLatestFrom | ❌ |  |
| zip | zip |  |

---

### Combine에만 있는 Operator들 (try Operator)

반면에 `Combine`에만 있는 `try` `Operator`들도 있습니다.

> `tryMap`
> 
> `tryContains(where:)`
> 
> `tryScan`
> 
> `tryAllSatisfy`
> 
> `tryFilter`
> 
> `tryDrop(while:)`
> 
> `tryCompactMap`
> 
> `tryPrefix(while:)`
> 
> `tryRemoveDuplicates(by:)`
> 
> `tryFirst(where:)`
> 
> `tryReduce`
> 
> `tryLast(where:)`
> 
> `tryMax(by:)`
> 
> `tryCatch`
> 
> `tryMin(by:)`
> 

`try` operators는 일반 operators보다 `error`을 좀 더 쉽게 핸들링할 수 있도록 도와줍니다.

아래 `tryMap`은 `Data type`뿐만 아니라 `map` 내부에서 발생할 수 있는 `Error`를 함께 핸들링할 수 있도록 `Result type`으로 방출시켜 줍니다.

```swift
func map<T>(_ transform: (Output) -> T) -> Just<T>

func tryMap<T>(_ transform: (Output) throw -> T) -> Result<T, Error>.Publisher
```

---

### Combining Operators(결합 연산자)의 차이

2개 이상의 이벤트를 묶어서 처리를 해야 할 때 사용하는 결합 연산자입니다.

`Combine`과 `RxSwift` 모두 `merge`, `combineLatest`, `zip`이 있지만,

`Combine`은 결합할 개수가 늘어나면 `Merge3`, `Merge4`, `Merge5`, ... 이렇게 별도의 연산자를 사용합니다.

`RxSwift`는 `CombineLatest`가 `최대 8개`인데, `Combine`은 `최대 4개`까지 되기 때문에,

5개 이상부터는 몇 개의 결합으로 나눠서 결합한 것을 다시 결합하는 방식으로 처리해야 합니다.

`Zip`도 마찬가지로 `RxSwift`는 `최대 8개`, `Combine`은 `최대 4개`까지 지원됩니다.

| Combine | RxSwift |
| --- | --- |
| Merge, Merge3, Merge4, Merge5, Merge6, Merge7, Merge8, MergeMany | merge |
| CombineLatest, CombineLatest3, CombineLatest4 | combineLatest |
| Zip, Zip3, Zip4 | zip |

### AnyCancellable

`RxSwift`의 `Disposable`에 해당하는 요소입니다.

`AnyPublisher`를 `subscribe` 하기 위해 `.sink`를 하면 `AnyCancellable`을 생성시킵니다.

`AnyCancellable`은 `Cancellable` `protocol`을 따르는 클래스입니다.

`RxSwift`에서는 ``Observable``을 `subscribe` 했을 때 `Disposable`이 생성되고 메모리 해제를 위해서 `dispose(by: disposeBag)`을 해주었죠?

하지만 `Combine`에서는 `DisposeBag`에 해당하는 개념은 없고, 대신 `Set<AnyCancellable>`에 넣어줍니다.

`RxSwift`의 `Disposable`과 `Combine`의 `AnyCancellable`은 `deinit 될` 때 동작이 취소가 됩니다.

`AnyCancellable`과는 다르게 그냥 `Cancellable`은 `deinit`시 동작이 취소되지 않기 때문에, 의도한 대로 동작하지 않을 수 있으니 주의해야 합니다.

| Combine | RxSwift |
| --- | --- |
| AnyCancellable | Disposable |
| - | DisposeBag |

```swift
// Combine
var cancellables = Set<AnyCancellable>()
Just(1)
     .sink {
        print($0)
    }
    .store(in: &cancellables)
```
```swift
// RxSwift
let disposeBag = DisposeBag() 
Observable.just(1)
    .subscribe(onNext: {
        print($0)
    }
    .disposed(by:disposeBag)
```

(전수열 님이 만드신 CancelBag을 사용할 수도 있습니다: https://github.com/devxoul/CancelBag)

---

### Thread Handling

`Combine`에도 `subscribe(on:)`이라는 동일한 이름으로 `Thread Handling`을 지원하지만,

`RxSwift`와는 다르게 동작을 합니다.

`Combine`에서는 `subscribe(on:)` 업스트림(코드 위쪽)에서만 해당 `Thread`에서 동작을 하게 됩니다.

`subscribe(on:)`의 위치에 주의해줘야 합니다.

```swift
// Combine
Just(1)
    .subscribe(on: DispatchQueue.main) // 위까지 main에서 동작
    .map { _ in
    	implements()
    }
    .sink { ... }
```

아래는 `RxSwift`에서의 `observeOn`, `subscribeOn`의 동작을 표현한 마블다이어그램입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/da78e0d1-5a2b-487e-88ed-d5e47c018aac)

`observeOn` 다운스트림(코드 아래)부터 해당 스레드에서 동작하고(위치 상관있음),

`subscribeOn`에서 설정한 스레드에서 `subscribe`하게 되어 있습니다(위치 상관없음).

---

### 성능

(https://medium.com/@M0rtyMerr/will-combine-kill-rxswift-64780a150d89)

1천만개의 element를 처리하는데에 드는 시간 및 Allocation을 비교한 아티클입니다.

Apple에서 Combine 발표시 성능에 중점을 두고 개발했다고 한 만큼

압도적으로 속도가 빠르고, 메모리 할당도 매우 적습니다.
| Time | Allocation |
|--|--|
|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/68e56ee9-1bf8-46e6-b254-4dffe42ddaf6)|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/724625f4-2312-4eb7-ab41-6b3b441cab97)|

---

`Combine`의 주요요소를 `RxSwift`와 비교해봤습니다.

양쪽의 장단점이 확실하기 때문에 현업에서는 이를 잘 따져서 사용을 해봐야겠습니다.