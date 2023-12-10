---
title: (RxSwift) TimeBased Operators
author: Cody
date: 2022-11-28 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---

이번 글에서는 `Time base`의 Operator를 정리해봅니다.

---

### replay

`subscribe`를 했을 때, `replay`대상이 이전에 방출했던 `element`를 `buffer`만큼 다시 방출시켜주는 Operator입니다.

`replay`를 지정해주고 `connect()`를 호출해주어야 합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/976c9d76-61db-4ca2-a951-aa7fb51e5bf0)

```swift
print("- - - - - replay - - - - -")
let shinee = PublishSubject<String>()
let replay = shinee.replay(2)
replay.connect() // replay와 같은 연산자들은 connect를 해주어야 함
shinee.onNext("링딩동 링딩동")
shinee.onNext("링디기딩디기딩딩딩")

replay
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - replay - - - - -
링딩동 링딩동
링디기딩디기딩딩딩
```

### replayAll

`replay`와 같지만 `buffer`가 무한한 버전입니다. `subscribe`이전에 방출되었던 모든 `Element`를 방출합니다.

마찬가지로 `connect()`를 해주어야 합니다.

```swift
print("- - - - - replayAll - - - - -")
let teajina = PublishSubject<String>()
let replayAll = teajina.replayAll()
replayAll.connect()

teajina.onNext("진진자라")
teajina.onNext("지리지리자")
teajina.onNext("진진자라")
teajina.onNext("지리지리자")

replayAll
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - replayAll - - - - -
진진자라
지리지리자
진진자라
지리지리자
```

### buffer

`buffer`는 `timeSpan`, `count`, `scheduler` 세가지 요소를 받습니다.

지정한 `scheduler`에서 `timeSpan` 시간동안 `count`만큼의 버퍼만큼 쌓아서 `배열`로 방출시킵니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/55c7e123-6611-4b43-85c1-0cdc2ed9bd99)

```swift
print("- - - - - buffer - - - - -")
let source = PublishSubject<String>()
var count = 0
let timer = DispatchSource.makeTimerSource()
timer.schedule(deadline: .now() + 2, repeating: .seconds(1))
timer.setEventHandler {
    count += 1
    source.onNext("\(count)")
}
timer.resume()

source
    .buffer(
        timeSpan: .seconds(2),
        count: 2,
        scheduler: MainScheduler.instance
    )
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - buffer - - - - -
[]
["1", "2"]
["3", "4"]
["5", "6"]
["7", "8"]
["9", "10"]
["11", "12"]
 ...
```

### window

`window`는 `buffer`와 동작이 유사하지만 배열대신 `Observable`로 방출을 합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3d1ac5ba-8410-4a11-a3ee-8f54cbb9c9fb)

```swift
print("- - - - - window - - - - -")
let maxCreatingObservable = 2

let window = PublishSubject<String>()

var windowCount = 0
let windowTimerSource = DispatchSource.makeTimerSource()
windowTimerSource.schedule(deadline: .now()+2, repeating: .seconds(1))
windowTimerSource.setEventHandler {
    windowCount += 1
    window.onNext("\(windowCount)")
}
windowTimerSource.resume()

window
    .window(timeSpan: .seconds(2),
            count: maxCreatingObservable,
            scheduler: MainScheduler.instance)
    .flatMap { windowObservable -> Observable<(index: Int, element: String)> in
        return windowObservable.enumerated()
    }
    .subscribe(onNext: {
        print("\($0.index)번째 Observable element: \($0.element)")
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - window - - - - -
0번째 Observable element: 1
0번째 Observable element: 2
1번째 Observable element: 3
0번째 Observable element: 4
1번째 Observable element: 5
0번째 Observable element: 6
1번째 Observable element: 7
...
```

### delaySubscription

`delaySubscription`은 `subscription`을 지정한 시간만큼 `delay`시킵니다.

```swift
print("- - - - - delaySubscription - - - - -")
let delaySource = PublishSubject<String>()

var delayCount = 0
let delayTimesource = DispatchSource.makeTimerSource()
delayTimesource.schedule(deadline: .now()+2, repeating: .seconds(1))
delayTimesource.setEventHandler {
    delayCount += 1
    delaySource.onNext("\(delayCount)")
}
delayTimesource.resume()

delaySource
    .delaySubscription(.seconds(5), scheduler: MainScheduler.instance)
    .subscribe(onNext: { // subscribe를 지연
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - delaySubscription - - - - -
4
5
6
7
8
9
10
...
```

### delay

`delay`는 시퀀스를 지정한 시간만큼 지연시킵니다. 이벤트 방출이 지연됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/88eecb0e-33ba-4380-9d35-5fbffd21fb86)

```swift
print("- - - - - delay - - - - -")
let delaySubject = PublishSubject<Int>()

var delayCount = 0
let delayTimesouce = DispatchSource.makeTimerSource()
delayTimesouce.schedule(deadline: .now(), repeating: .seconds(1))
delayTimesouce.setEventHandler {
    delayCount += 1
    delaySubject.onNext(delayCount)
}
delayTimesouce.resume()

delaySubject
    .delay(.seconds(5), scheduler: MainScheduler.instance) // 이벤트 방출을 지연
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - delay - - - - -
2
3
4
5
6
7
8
...
```

### interval

`interval`은 `DispatchSource`, `Timer`와 같은 역할을 `Rx`로 구현한 Operator입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/799764e8-89b5-4a44-ab50-493b6177fe65)

```swift
print("- - - - - interval - - - - -")
Observable<Int>
    .interval(.seconds(3), scheduler: MainScheduler.instance)
    .subscribe(onNext: {
        print($0) // of 등으로 시퀀스를 구현하지 않아도 타입추론으로 Int타입의 숫자들을 방출
    })
    .disposed(by: disposeBag)
```

(출력)

`Create` `Operator`로 시퀀스를 구현하지 않아도, 타입추론으로 `Int`
타입의 숫자들을 순서대로 방출되는 것을 확인할 수 있습니다.

```
- - - - - interval - - - - -
0
1
2
3
4
...
```

### timer

`timer`는 `interval`과 유사하지만. 좀더 디테일하게 설정이 가능합니다.

`dueTime`으로 첫 `Element`를 내보내는 시간을, `period`로 그 다음 `Element`들을 방출할 주기를 설정하고 `scheduler`를 설정할 수 있습니다.

`DispatchSource`, `Timer`보다 좀 더 직관적으로 작성하기 좋습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/128afea6-30fe-401e-b3ad-48c38c8063e7)

```swift
print("- - - - - timer - - - - -")
Observable<Int>
    .timer(.seconds(5),
           period: .seconds(2),
           scheduler: MainScheduler.instance)
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - timer - - - - -
0
1
2
3
4
...
```

### timeout

`timeout`은 정해진 시간 내에 이벤트를 방출시키지 않으면 `timeout` `error`를 방출시키는 Operator입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1c736d8d-d4c9-409b-b8a2-f5c0e9e7ed7c)

아래는 5초내로 버튼을 누르지 않으면 `timeout`이 되는 예제입니다.

```swift
print("- - - - - timeout - - - - -")
import RxCocoa
import PlaygroundSupport

let pushForNoError = UIButton(type: .system)
pushForNoError.setTitle("Push!!", for: .normal)
pushForNoError.sizeToFit()

PlaygroundPage.current.liveView = pushForNoError // Playground에 UIButton을 테스트하기 위함
pushForNoError.rx.tap
    .do(onNext: {
        print("tap")
    })
    .timeout(.seconds(5), scheduler: MainScheduler.instance)
    .subscribe {
        print($0)
    }
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - timeout - - - - -
tap
next(())
tap
next(())
error(Sequence timeout.)
```