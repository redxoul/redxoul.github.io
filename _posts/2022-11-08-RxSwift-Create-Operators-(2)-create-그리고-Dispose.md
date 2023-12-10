---
title: (RxSwift) Create Operators(2) create 그리고 Dispose
author: Cody
date: 2022-11-08 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---
### Create

[RxSwift의 Lifecycle을 정리한 글](https://swiftycody.github.io/posts/RxSwift-RxSwift%EC%9D%98-%EA%B8%B0%EB%B3%B8-%EC%9A%94%EC%86%8C-%EB%B0%8F-LifeCycle/)에서
`Observable`이 `next`, `completed`, `error` 이벤트를 방출시키는 타입이라고 했죠.
`Create`는 각 이벤트들을 방출시키는 `onNext`, `onCompleted`, `onError` 를 직접 구현하는 방식입니다.

```swift
print("----- create (1) -----")
Observable.create { observer -> Disposable in
    observer.on(Event.next(1)) 	// observer.on은 Event.next로 1을 방출받음
    observer.on(.next(2))		// 2를 방출받음
    observer.onNext(3)			// onNext로 줄일 수 있음.
    return Disposables.create()
}
.subscribe(onNext: { // next이벤트만 받아서 출력
    print($0)
})
```

위 예제의 `Create` 는 `subscribe`하는 `observer`가

`next`이벤트로 `1`, `2`, `3`을 받는 `Observable`을 생성합니다.

`observer.on(Event.next(1))`과 같이 긴 과정을 `observer.onNext(1)`로 축약할 수 있는

`SugarAPI`를 제공합니다. 

Lifecycle을 정리한 글에서 썼듯이 `Disposable`을 객체를 만들어 반환합니다.

예제 출력은 아래와 같습니다.

```
----- create (1) -----
1
2
3
```

위에서 작성한 `Create`는 이전글에서 작성한 `of`의 예제와 같습니다.

```swift
Observable<Int>.of(1, 2, 3)
    .subscribe(onNext: {
        print($0)
    })
```

사실 알고보면 다른 생성 `Operator`들은 `Create`의 `SugarAPI`였던거죠.

`RxSwift`는 이렇게 복잡해질 수 있는 코드를 간략히 표현할 수 있는 `SugarAPI`를 많이 제공합니다.

다음은 `onCompleted()`예제입니다.

```swift
print("----- create (2) -----")
Observable.create { observer -> Disposable in
    observer.onNext(1)
    observer.onCompleted()
    observer.onNext(2) // completed 이벤트 후에는 Observable 이벤트가 종료되어 실행되지 않음
    return Disposables.create()
}
.subscribe {
    print($0)
}
```

출력은 아래와 같습니다.
``complete`이벤트를 방출한 뒤로는 `next`이벤트가 방출되지 않는 것`을 확인할 수 있습니다.

```
----- create (2) -----
next(1)
completed
```

다음은 `onError()`의 예제입니다.

```swift
let disposeBag = DisposeBag()

print("----- create (3) -----")
enum MyError: Error {
    case anError
}

Observable<Int>.create { observer -> Disposable in
    observer.onNext(1)
    observer.onError(MyError.anError)
    observer.onNext(2)
    return Disposables.create()
}
.subscribe(onNext: {
    print($0)
}, onError: {
    print($0.localizedDescription)
}, onCompleted: {
    print("completed")
}, onDisposed: {
    print("disposed")
})
.disposed(by: disposeBag)
```

출력을 확인해보면 (`completed`와 마찬가지로)
`Error`를 방출한 후 `next`이벤트가 방출되지 않는 것을 확인할 수 있습니다.

```
----- create (3) -----
1
The operation couldn’t be completed. (__lldb_expr_56.MyError error 0.)
disposed
```

### Dispose

그리고 이번 예제에서는 이전에 작성하지 않았던 `DisposeBag`을 작성했습니다.

사실 모든 `Observable`은 `subscribe`를 하고나면 `Disposable`(폐기가능)을 반환하고,
모두 사용하고 난 `Observable`을 폐기하기 위해서 반드시 `dispose()`를 시켜주어야 합니다.
이는 명시적으로 `subscribe`를 종료, 취소시키는 행위로 이것을 하지 않았을 때 `메모리누수가 발생할 수 있`습니다.

이전 예제들에서는 생략했지만 실제 사용할 때는 반드시 `dispose`를 시켜주어야 합니다.

`dispose`에는 2가지 방식이 있는데,

첫번째는 더이상 `subscription`을 사용하지 않는다고 판단되는 타이밍에 `dispose()`를 호출하는 것 입니다.

```swift
// observable 생성
let observable = Observable.of("A", "B", "C")

// subscribe
let subscription = observable.subscribe { event in
    print(event)
}

// 더이상 사용하지 않을 때 dispose
subscription.dispose()
```

두번째는 `subscribe`와 동시에 `DisposeBag 에 넣는 방법`입니다.

보통 `observable`을 `subscribe`하는 값을 가지는 `ViewController`나 `ViewModel`등에 `disposeBag`을 만들어놓고, `subscribe`할 때 이 `disposeBag`에 넣어놓으면

`disposeBag`이 `dealloacate`될 때 알아서 넣어놓은 `subscription`들의 `dispose()`를 호출시켜 폐기시켜줍니다.

우리가 `RxSwift`를 쓰는 목적 중 많은 부분이

`RxCocoa`를 쓰기 위함이고, UI에 값을 바인딩해서 사용하기 위함인데, UI 바인딩을 끝내는 타이밍은 곧 UI가 없어지는 타이밍이기도 하고, 정확하게 `dispose()`를 처리할 타이밍을 잡기 어려운 경우도 많기 때문에 이 두번째 방식을 많이 사용합니다.

```swift
// DisposeBag 생성
let disposeBag = DisposeBag()

// Observable create
Observable.of("A", "B", "C")
    .subscribe { // subscribe
        print($0)
    }
    .disposed(by: disposeBag) // disposeBag에 subscription을 추가
```