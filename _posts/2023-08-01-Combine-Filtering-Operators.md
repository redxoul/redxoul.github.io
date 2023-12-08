---
title: (Combine) Filtering Operators
author: Cody
date: 2023-08-01 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---
`Publisher`가 내보내는 값이나 이벤트를 제한하고 그 중 일부만 소비하고 싶을 때 유용한 `Filtering Operator`입니다.

> 💡 Filtering Operator에는 try 접두사가 붙은 유사 Operator(예: filter와 tryFilter)가 있습니다.
> 유일한 차이점은 끝에서 throw하는 클로저를 제공한다는 것. 클로저 내에서 던지는 모든 오류는 던진 오류와 함께 Publisher를 종료합니다. 여기서는 non-throwing Operator에 대해서만 정리합니다.


### `filter`

(= `RxSwift`의 `filter`)

`Swift` 표준 라이브러리에도 있고 모두가 익숙한 `Filtering Operator`의 기본입니다.

`Bool`을 반환하는 클로저에 일치하는 값만 전달시킵니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5fe92f4f-7b4f-4f5b-b4cf-4f72be8d582f)

`filter`의 예시입니다.

```swift
// 1.시퀀스 타입에서 publisher 프로퍼티을 사용하여 1부터 10까지 한정된 수의 값을 방출하는 새 publisher를 생성
let numbers = (1...10).publisher

// 2. filter operator를 사용하여 3의 배수인 숫자만 통과하도록 허용하는 클로저를 전달
numbers
    .filter { $0.isMultiple(of: 3) }
    .sink(receiveValue: { n in
        print("\\(n) is a multiple of 3!")
    })
    .store(in: &subscriptions)
```

결과

```
3 is a multiple of 3!
6 is a multiple of 3!
9 is a multiple of 3!
```

### `removeDuplicates`

(= `RxSwift`의 `distinctUntilChanged`)

`동일한 값을 연속으로 방출`하는 `Publisher`가 있을 수 있는데, `이를 무시하고 싶을 때` 유용한 `Operator`입니다.

이 `Operator`는 `Equatable을 준수하는 모든 값에 대해 자동으로 동작`하기 때문에 따로 넣어줄 매개변수는 없습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0741b76c-e4d9-4f76-9910-51492162fcc4)

예시 코드입니다.

```swift
// 1. 문장을 단어 배열(예: [문자열])로 분리한 다음 이 단어들을 방출할 새 Publisher를 생성
let words = "hey hey there! want to listen to mister mister ?"
                .components(separatedBy: " ")
                .publisher
// 2. words Publisher에 removeDuplicates()를 적용
words
  .removeDuplicates()
  .sink(receiveValue: { print($0) })
  .store(in: &subscriptions)
```

결과

```
hey
there!
want
to
listen
to
mister
?
```

### `compactMap`

(= `RxSwift`의 `compactMap`)

`optional` 값을 반환하는 `Publisher`를 처리해야 할 때 필요한 `Swift`표준 라이브러리에서 지원하는`compactMap`과 같은 맥락의`Operator`입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d7f23634-6b98-4f7a-b226-8e7f495c7804)

아래는 `compactMap`의 사용 예시입니다.

```swift
// 1. 문자열 목록을 방출하는 퍼블리셔를 생성합니다.
let strings = ["a", "1.24", "3",
               "def", "45", "0.23"].publisher

// 2. compactMap을 사용하여 각 String에서 Float를 초기화하려고 시도.
// Float의 이니셜라이저가 제공된 String을 변환할 수 없는 경우 nil을 반환.
// 이러한 nil 값은 compactMap 연산자에 의해 자동으로 필터링.
strings
    .compactMap { Float($0) }
    .sink(receiveValue: {
        // 3. Float로 성공적으로 변환된 String만 출력
        print($0)
    })
    .store(in: &subscriptions)
```

출력

```
1.24
3.0
45.0
0.23
```

### `ignoreOutput`

(= `RxSwift`의 `ignoreElement`)

`Publisher`가 방출한 실제 값은 무시한 채 `값을 모두 출력했다는 사실(completed)만 알고 싶을 때`가 있는데, 이런 케이스에 사용할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/26decbf7-c3cb-4367-bbec-4e10b596f4ab)

아래는 예시코드 입니다.

```swift
// 1. 1부터 10,000까지 10,000개의 값을 출력하는 publisher를 생성
let numbers = (1...10_000).publisher

// 2. 모든 값을 생략하고 completed 이벤트만 consumer에게 전송하는 ignoreOutput를 추가
numbers
  .ignoreOutput()
  .sink(
    receiveCompletion: { print("Completed with: \($0)") },
    receiveValue: { print($0) }
  )
  .store(in: &subscriptions)
```

출력

```
Completed with: finished
```

### `First(where:)`

`Swift` 표준라이브러리에 있는 그것과 동일한 맥락의 `Operator`입니다.

`where` 클로저를 만족하는 첫번째 값만 찾아서 필터링시켜줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/07a62332-18c5-478b-830a-bfc80e8fd31b)


> 💡 이 Operator는 lazy하게 동작합니다.
> 그래서 where클로저와 일치하는 값을 찾을 때까지 필요한 만큼만 값을 불러오고,
> 일치하는 값을 찾으면 바로 subscription은 cancel되고 completed됩니다.

아래는 `first(where:)`의 예시입니다.

```swift
// 1. 1부터 9까지의 Int를 방출하는 새 publisher를 생성
let numbers = (1...9).publisher

// 2. first(where:) operator를 사용하여 첫 번째로 방출된 짝수 값을 탐색
numbers
    .print("numbers") // 3. lazy한 동작을 확인하기 위한 print
    .first(where: { $0 % 2 == 0 })
    .sink(
        receiveCompletion: { print("Completed with: \($0)") },
        receiveValue: { print($0) }
    )
    .store(in: &subscriptions)
```

출력.

첫 번째 짝수(2)를 찾자마자 subscription이 취소되고 completed되는 것이 확인됩니다.

```
numbers: receive subscription: (1...9)
numbers: request unlimited
numbers: receive value: (1)
numbers: receive value: (2)
numbers: receive cancel
2
Completed with: finished
```

`first()`를 사용해서 `where`조건없이 첫번째 값을 가져올 수도 있습니다.

(= `RxSwift`의 `first`)

### `last(where:)`

바로 전에 보았던 `first(where:)`와 반대로 `where`클로저를 만족하는 마지막 값만 찾아서 필터링시켜줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e64d2189-6ea5-4428-a099-115e4bf021aa)

> ⚠️ `first(where:)`와 달리 이 operator는 publisher가 값 전송을 완료할 때까지 기다려야 일치하는 값을 찾았는지 알 수 있으므로 greedy하게 동작하며 Upstream은 반드시 유한해야 합니다.

아래는 `last(where:)`의 예시입니다.

```swift
// 1. 1에서 9 사이의 Int를 방출하는 Publisher 생성
let numbers = (1...9).publisher

// 2. last(where:) operator로 마지막으로 방출된 짝수 값을 탐색
numbers
    .last(where: { $0 % 2 == 0 })
    .sink(
        receiveCompletion: { print("Completed with: \($0)") },
        receiveValue: { print($0) }
    )
    .store(in: &subscriptions)
```

출력

```
8
Completed with: finished
```

`Upstream`이 무한한 `Publisher`에서 `last(where:)`를 사용하게 되면, 방출된 값이 마지막 값인지 알 수가 없기 때문에 `DownStream`에서 값을 받을 수 없게 됩니다.

아래는 `PassthroughSubject`에 ``last(where:)``를 사용한 예시입니다.

```swift
let numbers = PassthroughSubject<Int, Never>()

numbers
    .last(where: { $0 % 2 == 0 })
    .sink(
        receiveCompletion: { print("Completed with: \($0)") },
        receiveValue: { print($0) }
    )
    .store(in: &subscriptions)

numbers.send(1)
numbers.send(2)
numbers.send(3)
numbers.send(4)
numbers.send(5)
```

계속 `send`를 해도 값은 방출되지 않습니다.

`complete`을 시켜주면 그 때 값이 방출됩니다.

```swift
numbers.send(completion: .finished)
```

출력

```
4
Completed with: finished
```

`last()`를 사용하여 `where`조건없이 마지막값을 가져올 수도 있습니다.

(= `RxSwift`의 `takeLast`)

### `dropFirst`

(= `RxSwift`의 `skip`)

`count` 매개변수를 받아서(기본값 1) `publisher`가 `count` 값 갯수만큼 방출한 값을 무시합니다.

`count` 값을 건너뛴 후에 `publisher`가 값을 전달하기 시작합니다.

예를 들어 `count`가 3이면 4번째 값부터 값을 받습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/30906a7e-88fe-4c26-a9e7-b190088a73b2)

``dropFirst()``의 예시입니다.

```swift
// 1. 1에서 10 사이의 Int를 출력하는 publisher 생성
let numbers = (1...10).publisher

// 2. dropFirst(8)로 처음 8개의 값을 무시하고 9와 10만 출력
numbers
    .dropFirst(8)
    .sink(receiveValue: { print($0) })
.store(in: &subscriptions)
```

출력

```
9
10
```

### `drop(while:)`

`while`클로저의 조건문이 true인 동안 값을 무시하다가, false가 되는 시점부터 값을 방출시키도록 해줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6bbbfe14-a975-4271-b7d5-ec1533537ea0)

``drop(while:)``의 예시입니다.

```swift
// 1. 1에서 10 사이의 Int를 방출하는 publisher 생성
let numbers = (1...10).publisher

// 2. drop(while:)을 사용하여 5로 나눌 수 있는 첫 번째 값이 나올 때까지 drop하고,
// 조건이 끝나는 즉시 drop이 끝나고 값을 방출시키기 시작
numbers
    .drop(while: { $0 % 5 != 0 })
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

출력

```
5
6
7
8
9
10
```

`filter`와 `drop(while:)` 동작의 차이점

| filter | drop(while:) |
| --- | --- |
| 클로저 조건을 만족하는 값들만 방출 | 클로저 조건을 만족하는 값들을 무시 |
| 클로저 조건을 만족하는 값이 나와도 filter를 계속 적용 | 클로저 조건을 만족할 때까지만 drop을 적용 |

### `drop(untilOutputFrom:)`

(= `RxSwift`의 `skipUntil`)

두 번째 `publisher`가 값을 방출할 때까지, 첫 번째 `publisher`의 값을 무시할 수 있습니다.

예를 들어서 사용자가 어떤 버튼을 누르면 `isReady publisher`가 값을 방출하고, 그 때부터 첫 번째 `publisher`가 값을 방출하도록 하는 경우입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2eba9f42-b4c7-4d4f-b170-eae9ef5d5c42)

아래는 ``drop(untilOutputFrom:)``의 예시입니다.

```swift
// 1. 수동으로 값을 전송할 수 있는 두 개의 PassthroughSubject를 생성
// 첫 번째는 isReady. 두 번째는 사용자의 tap
let isReady = PassthroughSubject<Void, Never>()
let taps = PassthroughSubject<Int, Never>()

// 2. drop(untilOutputFrom: isReady)를 사용하여 isReady가 적어도 하나의 값을 내보낼 때까지 사용자의 tap을 무시
taps
    .drop(untilOutputFrom: isReady)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

// 3. subject로 5번의 tap을 전달.
(1...5).forEach { n in
    taps.send(n)

    // 4. 세 번째 tap 후에는 isReady에 값을 전달
    if n == 3 {
        isReady.send()
    }
}
```

출력

```
4
5
```

### `prefix`

(= `RxSwift`의 `take`)

주어진 수 만큼의 값만 받고 완료시키는 `operator`입니다.

`dropfirst`와 반대되는 매커니즘입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6d5a4c54-d458-48b1-b09f-3d3dbc8f9efd)

`prefix`의 예제입니다.

```swift
// 1. 1에서 10 사이의 Int를 방출하는 publisher 생성
let numbers = (1...10).publisher

// 2. prefix(2)를 사용하여 처음 두 값만 방출하도록 함. 두 개의 값이 방출되는 즉시 publisher가 completed
numbers
    .prefix(2)
    .sink(
        receiveCompletion: { print("Completed with: \($0)") },
        receiveValue: { print($0) }
    )
    .store(in: &subscriptions)
```

출력

```
1
2
Completed with: finished
```

`prefix`는 `lazy`하게 동작하기 때문에, 필요한 만큼만 값을 받고 종료시켜 더 추가적인 값을 생성하지 않도록 해줍니다.

### `prefix(while:)`

`while`클로저의 결과가 `true`일 때까지 `Upstream Publisher`의 값을 통과시킵니다. 결과가 `false`이면 바로 `Publisher`가 `completed`됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2d0b137c-f8f1-48b9-b3dd-fdf770497013)

``prefix(while:)``의 예시입니다.

```swift
// 1. 1에서 10 사이의 Int를 방출하는 publisher 생성
let numbers = (1...10).publisher

// 2. prefix(while:)를 사용하여 값이 3보다 작으면 통과. 3보다 크거나 같은 값이 나오면 Publisher가 Completed
numbers
    .prefix(while: { $0 < 3 })
    .sink(
        receiveCompletion: { print("Completed with: \($0)") },
        receiveValue: { print($0) }
    )
    .store(in: &subscriptions)
```

결과.

```
1
2
Completed with: finished
```

### `prefix(untilOutputFrom:)`

(= `RxSwift`의 `takeUntil`)

`prefix(untilOutputFrom:)`는 두 번째 `publisher`가 값을 방출할 때까지 값을 받습니다.

예를 들어 사용자가 두 번만 탭할 수 있는 버튼이 있다고 했을 때,

두 번의 탭이 발생하자마자 버튼에 대한 3번 이상의 탭 이벤트는 생략되도록 할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/07276ee9-dcc5-4e06-9b7a-883e168f02ec)

``prefix(untilOutputFrom:)``의 예시입니다.

```swift
// 1. 수동으로 값을 전송할 수 있는 두 개의 PassthroughSubject를 생성
// 첫 번째는 isReady. 두 번째는 사용자의 tap
let isReady = PassthroughSubject<Void, Never>()
let taps = PassthroughSubject<Int, Never>()

// 2. prefix(untilOutputFrom: isReady)를 사용하여 isReady가 값을 내보낼 때까지 tap 이벤트를 통과
taps
    .prefix(untilOutputFrom: isReady)
    .sink(
        receiveCompletion: { print("Completed with: \($0)") },
        receiveValue: { print($0) }
    )
    .store(in: &subscriptions)

// 3. Subject을 통해 5번의 tap을 보내고
(1...5).forEach { n in
    taps.send(n)

    // 4. 두 번째 탭 후에는 isReady에 값을 보냄
    if n == 2 {
        isReady.send()
    }
}
```

출력

```
1
2
Completed with: finished
```