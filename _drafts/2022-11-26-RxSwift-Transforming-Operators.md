---
title: (RxSwift) Transforming Operators
author: Cody
date: 2022-11-26 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---

이번 글에서는 받은 값을 `가공`하기 위한 `Transforming Operators`를 정리해봅니다.

## toArray

`Observable`의 모든 `Element`들을 하나의 `Array`로 묶인 `Single`로 변환시켜줍니다.

[Single 정리글](https://swiftycody.github.io/posts/RxSwift-Traits-Single-Completable-Maybe/)에서도 썼지만, `Single`로 변환시 `Observable`이 `complete`이벤트를 방출하는지 반드시 확인을 해야합니다. 그렇지 않으면 `observer`가 절대로 `success`이벤트를 받을 수가 없게 됩니다

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/894e0131-694b-48c9-abf1-6e4233372beb)

마블다이어그램에도 볼수 있듯이 `complete`이벤트(세로선)이 발생해야 `Array`로 변환시켜줍니다.

```swift
print("- - - - - toArray - - - - -")
Observable.of("HoneyButter", "Wasabi", "Buldak", "MintChoco")
    .toArray()
    .subscribe(onSuccess: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - toArray - - - - -
["HoneyButter", "Wasabi", "Buldak", "MintChoco"]
```

## map

`Swift`에서 제공하는 `map`과 동일한 기능의 Operator입니다.

`Date()`를 받아서 `DateFormatter`로 변환하는 `map`의 예시입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6a1b8f6d-c64b-4489-bae0-7a098fb41172)

```swift
print("- - - - - map - - - - -")
Observable.of(Date())
    .map { date -> String in
        let dateFormatter = DateFormatter()
        dateFormatter.dateFormat = "yyyy.MM.dd"
        dateFormatter.locale = Locale(identifier: "ko_KR")
        return dateFormatter.string(from: date)
    }
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - map - - - - -
2022.11.24
```

## flatMap

`Observable` 내부의 `Observable`(`inner Observable`)을 다루기 위한 `Operator`입니다.

중첩된 `Observable` 내부의 `Observable`을 꺼내서 쓸 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b9d3c9e3-3f3d-4098-94f6-a6dbc4991b84)

```swift
print("- - - - - flatmap - - - - -")
protocol Almond {
    var tasteScore: BehaviorSubject<Int> { get }
}

struct Tester: Almond {
    var tasteScore: BehaviorSubject<Int>
}

let wasabiTester = Tester(tasteScore: BehaviorSubject<Int>(value: 10))
let mintChocoTester = Tester(tasteScore: BehaviorSubject<Int>(value: 1))

let testing = PublishSubject<Tester>()

testing
    .flatMap { tester in
        tester.tasteScore
    }
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

testing.onNext(wasabiTester)
wasabiTaster.tasteScore.onNext(10)

tasting.onNext(mintChocoTaster)
wasabiTaster.tasteScore.onNext(10)
wasabiTaster.tasteScore.onNext(10)
mintChocoTaster.tasteScore.onNext(1)
```

(출력)

```
- - - - - flatmap - - - - -
10
10
1
10
10
1
```

## flatMapLatest

`flatMap`과 유사하지만, 상위 `Observable`에서 마지막으로 `next`된 `내부 Observable`의 값만 `subscribe`하게 됩니다.

`map`과 `switchLatest`의 조합.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9671a0ec-5f5d-4046-a074-177b5ab3af94)

```swift
print("- - - - - flatmapLatest - - - - -")
let buldakTester = Tester(tasteScore: BehaviorSubject<Int>(value: 9))
let cornTester = Tester(tasteScore: BehaviorSubject<Int>(value: 8))

let testing2 = PublishSubject<Tester>()

testing2
    .flatMapLatest { tester in
        tester.tasteScore
    }
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

testing2.onNext(buldakTester)
buldakTaster.tasteScore.onNext(9)

tasting2.onNext(cornTaster)
buldakTaster.tasteScore.onNext(9)
cornTaster.tasteScore.onNext(8)

tasting2.onNext(buldakTaster)
cornTaster.tasteScore.onNext(8)
```

(출력)

`내부 Observable`이 `BehaviorSubject`기 때문에

최신이전에 `next`되었던 `내부 BehaviorSubject`값은 방출되지 않고 있다가,

다시 `next`되자마자 다시 `subscribe`되어 방출되지 않았던 마지막 값을 다시 방출하며 시작하는 것을 확인할 수 있습니다.

```
- - - - - flatmapLatest - - - - -
9
9
8
8
9
```

---

위에서 `map`, `flatMap`, `flatMapLatest`는 내부 `Observable`의 값만 받을 수 있었는데요.

위 예제들은 `내부 Observable`에서 `error`가 발생했을 때

처리가 되지 않아서 `외부 Observable`이 종료가 될 수 있다는 문제점이 있습니다.

```swift
enum hate: Error {
    case MintChoco
}

buldakTester.tasteScore.onError(hate.MintChoco) // Unhandled error happened: MintChoco
```

이 문제를 해결하기 위해서

아래에서 설명할 `materialize`와 `dematerialize`가 필요해집니다.

---

## materialize & dematerialize

`materialize`는 구체화한다는 뜻입니다.

`materialize` Operator를 사용하면 어떤 이벤트를 받았는지 알 수 있게 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8ccec053-2263-4413-9b0b-7cec4b21f1f0)

반대로 `dematerialize`는 다시 이벤트 값만 받을 수 있도록 해주는 Operator입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b70edc79-be8b-4620-80d8-90a9b7b0345d)

아래는 `materialize`, `dematerialize`의 예제입니다.

```swift
print("- - - - - materialize, dematerialize - - - - -")
enum hate: Error {
    case MintChoco
}

let me = Taster(tasteScore: BehaviorSubject<Int>(value: 10))
let you = Taster(tasteScore: BehaviorSubject<Int>(value: 8))

let almondEating = PublishSubject<Taster>()

almondEating
    .flatMapLatest { Taster in
        Taster.tasteScore
            .materialize()
    }
    .filter {
        guard let error = $0.error else { return true }
        print("Error: ", error) // error는 출력만 해주고 필터링
        return false
    }
    .dematerialize()
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

almondEating.onNext(me)
me.tasteScore.onNext(10)
me.tasteScore.onError(hate.MintChoco) // me Observable은 종료
almondEating.onNext(you) // 외부 Observable은 종료되지 않음
you.tasteScore.onNext(8)
you.tasteScore.onNext(8)
```

(출력)

`materialize`로 `error`를 처리해주었기 때문에,

`외부 Observable`이 종료되지 않고,

다른 `내부 Observable`의 값을 계속해서 받을 수 있는 것이 확인됩니다.

```
- - - - - materialize, dematerialize - - - - -
10
10
Error:  MintChoco
8
8
8
```