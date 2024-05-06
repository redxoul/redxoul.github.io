---
title: (Combine) Type Erasure
author: Cody
date: 2023-05-07 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---
( = RxSwift의 `.asObservable()`)

`Subscriber`가 `Publisher`의 어떤 추가적인 세부정보를 알 필요가 없이 `Publisher`로부터 값만 수신하도록 하고 싶을 수 있습니다.

이럴 때는 `.eraseToAnyPublisher()` `operator`를 통해 `Subscriber`가 `AnyPublisher`로 인식하도록 만들어줄 수 있습니다.

아래는 예시입니다.

```swift
var subscriptions = Set<AnyCancellable>()

// 1. PassthroughSubject를 생성
let subject = PassthroughSubject<Int, Never>()

// 2. subject에서 type-erase된 publisher를 생성
let publisher = subject.eraseToAnyPublisher()

// 3. type-erase된 publisher를 subscribe
publisher
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

// 4. subject를 통해 새 값을 전달
subject.send(0)
```

출력.

```
0
```

`publisher`를 `option-click`해보면 `PassthroughSubject<Int, Never>` 타입이 아닌, `AnyPublisher<Int, Never>`로 바뀌어 있는 것을 볼 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/53d0ea27-4d7a-459d-9abd-3eb1f08ef658)

이처럼 `type-erasure`를 사용하면

`Subscriber`나 `downstream Publisher`에게 노출되지 않길 원하는 `Publisher`에 대한 상세 정보를 숨길 수가 있습니다.

언제 이런 `type-erasure`가 필요할까요?

`Private Subject`로 어떤 값들을 처리하고, 
`Public Publisher`로는 값을 내보내기만 하고 싶을 때, 해당 `Public Publisher`로는 값을 받지 못하도록 만들고 싶을 때
`Private Subject`를 `type-erase`하여 `Publisher`를 만들어줄 수 있습니다.