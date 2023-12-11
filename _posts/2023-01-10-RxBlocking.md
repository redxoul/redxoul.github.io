---
title: (RxCocoa) RxBlocking
author: Cody
date: 2023-01-10 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift, RxCocoa]
pin: false
mermaid: true
---

`RxBlocking`은

바로 전 포스팅에서 정리했던 `RxTest`와 같은 `TestScheduler`는 없이

`Observable`의 `Event`방출을 검증하는 방법입니다.

단순히 `Observable`이 특정값들을 방출하는 것을 검증할 때는 `RxTest`가 아니라 `RxBlocking`만으로 충분할 것입니다.

`RxBlocking`은 `Observable`의 `next`이벤트를 배열로 변환하는 방법을 제공하여,

해당 값을 `TestAssertion`으로 테스팅할 수 있도록 해줍니다.

`Observable`에 `.toBlocking()`을 사용하면

`Observable`을 `BlockingObservable`로 전환해주고,

`BlockingObservable`은 `.toArray()`를 통해 `next`이벤트들을 `complete`이벤트가 발생할 때까지 배열로 전환시켜줍니다.

아래는 예시입니다.

```swift
// Observable -> BlockingObservable
let observable = Observable.of("A", "B", "C").toBlocking()

// Observable의 next이벤트를 Array로 전환
let values = try! observable.toArray()

// Nimble로 Test
expect(values).to(equal("A", "B", "C"))
```

`complete`이벤트가 보장되지 않는 `Observable`은 `.toBlocking(timeout:)`을 통해 `timeout`을 설정하여 전환을 시켜주면 됩니다.

```swift
// Observable -> BlockingObservable
let observable = Observable.of("non", "complete", "event").toBlocking(timeout: 2)

// Observable의 next이벤트를 Array로 전환
let values = try! observable.toArray()

// Nimble로 Test
expect(values).to(equal("non", "complete", "event"))
```