---
title: (RxSwift) Create Operators(1) just, of, from, range, empty, never
author: Cody
date: 2022-11-08 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---
앞에서 정리한 RxSwift의 Lifecycle 중에서  
가장 앞단계인 `Observable`의 생성을 위한 `Operator`들을 간단히 정리해봅니다.

## Just

단 하나의 `Next`이벤트로 `Element`를 방출하고 종료하는 `Observable` 시퀀스를 생성합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a3fd791f-057f-4b8d-9ae8-826c43d25949)

```swift
print("----- just(1) -----")
Observable<Int>.just(1) 	// 1을 방출하는 Observable을 생성하는 just
    .subscribe(onNext: {	// just가 방출시킨 값을 print하는 subscribe
        print($0)
    })
```

(출력)

```
----- just(1) -----
1
```

`<Int>`를 생략하여도, `just`의 타입추론으로 `Observable<Int>` 타입으로 반환됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f52a4578-2639-4ce5-9381-5907bd943156)

`Just`로 여러개의 값을 방출시키고 싶다면 배열로 내보내면 됩니다.

```swift
print("----- just(2) -----")
Observable.just([1, 2, 3, 4, 5])
    .subscribe(onNext: {
        print($0)
    })
```

(출력)

```
----- just(2) -----
[1, 2, 3, 4, 5]
```

---

## Of

여러개의 이벤트를 방출시키는 `Observable`을 생성합니다.

```swift
print("----- of (1) -----")
Observable<Int>.of(1, 2, 3)
    .subscribe(onNext: {
        print($0)
    })
```

(출력. 각각의 `Int`가 3번 방출)

```
----- of (1) -----
1
2
3
```

위 값들을 배열로 넣으면 `Observable<[Int]>`가 되고, 결과는 `Just`에 배열로 넣은 것과 동일하게 됩니다.

```swift
print("----- of (2) -----")
Observable.of([1, 2, 3, 4, 5])
    .subscribe(onNext: {
        print($0)
    })
```

(출력)

```
----- of (2) -----
[1, 2, 3, 4, 5]
```

---

## From

From은 배열(`[Element]`)로 입력을 받아서,

`배열 내 값을 하나씩 방출`시키는 `Observable<Element>`시퀀스을 생성합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b4bc5e66-8381-49f6-aa97-c8d7e45066b7)

```swift
print("----- from -----")
Observable.from([1, 2, 3, 4, 5])
    .subscribe(onNext: {
        print($0)
    })
```

(출력. 배열 내 `Element`들이 각각 방출됨)

```
----- from -----
1
2
3
4
5
```

---

## Range

`특정범위의 값이 순차적으로 증가하는 `Observable`을 생성`시킵니다.
`start`값, `count`값 외에 옵셔널로 `scheduler`를 지정해줄 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/61d50f35-ef8e-4be3-9ff2-24d6b748de93)

```swift
print("----- range -----")
Observable.range(start: 1, count: 9)
    .subscribe(onNext: {
        print("2 * \($0) = \(2*$0)")
    })
```

(출력)

```
----- range -----
2 * 1 = 2
2 * 2 = 4
2 * 3 = 6
2 * 4 = 8
2 * 5 = 10
2 * 6 = 12
2 * 7 = 14
2 * 8 = 16
2 * 9 = 18
```

---

## Empty

값이 빈 `Observable`을 생성합니다. `Completed`이벤트만 방출합니다.

`empty`일 때는 아무값도 넣지 않아 타입추론을 할 수 없어서

`Observable에 <Void>`를 명시해주어야만 `Completed` 이벤트를 발생시킵니다.

`의도적으로 값이 없는, 즉시 종료하는 Observable을 반환해야할 때` 유용합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e23fff21-1400-4dda-8d27-2fab6ee56977)

```swift
print("----- empty -----")
Observable<Void>.empty()
    .subscribe {
        print($0)
    }
```

(출력)

```
----- empty -----
completed
```

---

## Never

방출되는 값도 없고, 종료이벤트(`completed`, `error`)도 발생하지 않는 무한의 `Observable`을 생성합니다.

```swift
print("----- never -----")
Observable.never()
    .debug("never") // subscribe 출력을 위한 debug operator
    .subscribe(onNext: {
        print($0)
    }, onCompleted: {
        print("completed")
    })
```

(출력)

```
----- never -----
2022-11-06 20:45:49.779: never -> subscribed
```