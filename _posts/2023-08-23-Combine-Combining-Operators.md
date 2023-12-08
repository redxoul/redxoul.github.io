---
title: (Combine) Combining Operators
author: Cody
date: 2023-08-23 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---
`Combining Operator`을 사용하면 서로 다른 `Publisher`가 방출한 이벤트를 결합(combine)하고

결합 코드에서 의미 있는 데이터 조합을 만들 수 있습니다.

예를 들어서 사용자 이름, 비밀번호, 체크박스 등의 입력이 필요한 양식의 경우 각각의 `Publisher`를 조합해서 동작하도록 만들 수 있습니다.

먼저 `prepend` 연산자 시리즈입니다.

`prepend`는 '앞에 붙이다'라는 뜻처럼 `Publisher`에서 나오는 값보다 먼저 나오는 값을 지정해줄 수 있습니다.

(Rx의 `concat`과 유사)

# prepend 연산자 시리즈

## prepend(Output...)

`prepend(Output)`는 `...` 구문을 사용하여 다양한 값 리스트를 받습니다.

즉, 원래 `Publisher`와 동일한 출력 타입인 값을 얼마든지 받을 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0cfdf639-9f7d-415d-ba28-3f955c16ac6e)

간단한 예시입니다.

```swift
// 1. 숫자 3 4를 방출하는 Publisher를 생성
let publisher = [3, 4].publisher

// 2. prepend를 사용하여 Publisher의 자체 값 앞에 숫자 1과 2를 추가
publisher
    .prepend(1, 2)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

출력.

```
1
2
3
4
```

`prepend` 연산자를 여러개를 사용할 수도 있습니다.

```swift
let publisher = [3, 4].publisher

publisher
    .prepend(1, 2)
    .prepend(-1, 0) // 추가
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

출력

```
-1
0
1
2
3
4
```

마지막 prepend가 업스트림에 먼저 영향을 미치기 때문에,

`-1`, `0`이 먼저 출력되고, `1`, `2`가 출력, 그리고 `3`, `4`가 출력되는 것을 확인.

## prepend(sequence)

이 `prepend` 변형은 이전 것과 유사하지만 모든 `Sequence` 개체를 입력으로 사용한다는 차이점이 있습니다.

예를 들어 `Array` 또는 `Set`을 사용할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3c808332-aee2-431b-afb1-30f3debd5575)


예시.

```swift
// 1. 숫자 5, 6, 7을 방출하는 Publisher를 생성
let publisher = [5, 6, 7].publisher

// 2. prepend(Sequence)를 원래 publisher에게 두 번 추가
// 한 번은 Array의 값을 앞에 추가하고 두 번째는 Set의 값을 앞에 추가
publisher
    .prepend([3, 4])
    .prepend(Set(1...2))
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

출력.

```
1
2
3
4
5
6
7
```

> ⚠️ Set은 Array와 달리 순서가 보장되지 않기 때문에 방출되는 순서가 달라질 수 있음.  
> 위 예시에서는 1, 2가 2, 1이 될 수도 있음.

`stride` Sequence를 더 추가.

```swift
let publisher = [5, 6, 7].publisher

publisher
    .prepend([3, 4])
    .prepend(Set(1...2))
    .prepend(stride(from: 6, to: 11, by: 2)) // 추가
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

출력.

```
6 // 추가
8 // 추가
10 // 추가
2
1
3
4
5
6
7
```

## prepend(Publisher)

두 개의 다른 `publisher`가 있고 그들의 값을 함께 붙이고 싶을 땐

`prepend(Publisher)`를 사용하여 원래 `publisher`의 값 앞에 두 번째 `publisher`가 내보낸 값을 추가 가능합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e4baee9d-b69b-44f9-80c6-593d39bea2fe)

prepend(publisher)의 첫번째 예시.

```swift
// 1. 숫자 3과 4를 방출하는 publisher와 1과 2를 방출하는 publisher를 생성
let publisher1 = [3, 4].publisher
let publisher2 = [1, 2].publisher

// 2. Publisher1의 시작 부분에 Publisher2를 추가.
// Publisher1은 작업 수행을 시작하고 Publisher2가 finished completion 이벤트를 보낸 후에만 이벤트를 방출.
publisher1
    .prepend(publisher2)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

출력.

```
1
2
3
4
```

prepend(publisher)의 두번째 예시.

이번에는 `publisher2`를 `subject`로 바꿉니다.

```swift
// 1. 첫 번째 publisher는 값 3과 4를 방출.
// 두 번째 publisher는 값을 동적으로 받아들일 수 있는 PassthroughSubject
let publisher1 = [3, 4].publisher
let publisher2 = PassthroughSubject<Int, Never>()

// 2. Publisher1 앞에 subject를 prepend
publisher1
    .prepend(publisher2)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

// 3. subject로 값 1과 2를 방출
publisher2.send(1)
publisher2.send(2)
```

출력.

```
1
2
```

예상대로 `subject`가 완료되지 않았기 때문에 `publisher1`이 값을 방출할 수 없게 됩니다.

`finish`를 방출시키면, `publisher1`이 값을 방출할 수 있게 됩니다.

```swift
let publisher1 = [3, 4].publisher
let publisher2 = PassthroughSubject<Int, Never>()

publisher1
    .prepend(publisher2)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

publisher2.send(1)
publisher2.send(2)
publisher2.send(completion: .finished) // 추가
```

출력.

```
1
2
3
4
```

---

# append 연산자 시리즈

`append`는 `prepend`와 반대로 첫번째 `publisher`가 값을 모두 방출했을 때, 추가로 방출할 값을 설정해 줄 수 있습니다.

## append(Output…)

append(Output...)은 prepend와 유사하게 작동합니다. 또한 `Output` 유형의 가변 목록을 취하지만 원래 `publisher`가 `.finished` 이벤트로 `completed`된 후에 해당 항목을 추가합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/15ebb1e2-5853-4267-bba5-ea85c5ea0b4a)

아래는 간단한 예제입니다.

```swift
// 1. 단일값 1을 내보내는 publisher
let publisher = [1].publisher

// 2. 2, 3을 append하고 4를 append
publisher
    .append(2, 3)
    .append(4)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

출력.

```
1
2
3
4
```

각 `Append`는 작업을 추가하기 전에 업스트림이 완료될 때까지 기다렸다가 완료 후 동작합니다.

```swift
// 1. publisher는 수동으로 값을 보낼 수 있는 PassthroughSubject로
let publisher = PassthroughSubject<Int, Never>()

publisher
    .append(3, 4)
    .append(5)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

// 2. PassthroughSubject에 1과 2를 방출
publisher.send(1)
publisher.send(2)
```

출력.

`publisher`로 방출한 값만 출력됨을 확인.

```
1
2
```

아래처럼 코드를 추가하면 `append`가 동작함을 확인.

```swift
let publisher = PassthroughSubject<Int, Never>()

publisher
    .append(3, 4)
    .append(5)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

publisher.send(1)
publisher.send(2)
publisher.send(completion: .finished) // 추가
```

출력.

```
1
2
3
4
5
```

> ⚠️ 업스트림의 publisher가 .finished 완료를 보내지않으면 모든 append는 동작하지 않음.

## append(Sequence)

`Sequence`를 개체를 가져와 원래 `publisher`의 모든 값이 내보낸 후에 해당 값을 추가합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ad7a60a1-6459-4b07-a2a4-6cf2d59c647d)


예시입니다.

```swift
// (1) 1, 2, 3을 방출하는 publisher 생성
let publisher = [1, 2, 3].publisher

publisher
    .append([4, 5]) // (2) 4와 5(순서대로)를 사용하여 배열을 추가
    .append(Set([6, 7])) // (3) 6과 7(순서 없음)을 사용하여 Set를 추가
    .append(stride(from: 8, to: 11, by: 2)) // (4) 8에서 11 사이를 2씩 증가하는 Strideable을 추가
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

출력.

```
1
2
3
4
5
7
6
8
10
```

## append(Publisher)

append(Publisher)는 publisher를 가져와서 publisher에서 방출한 모든 값을 원래 publisher의 끝에 추가합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2433442d-564c-4372-b949-c37d586c08ee)


예시입니다.

```swift
// (1) 첫 번째 publisher는 1과 2를 방출
// 두 번째 publisher는 3과 4를 방출
let publisher1 = [1, 2].publisher
let publisher2 = [3, 4].publisher

// (2) publisher2를 publisher1에 appen
// publisher2의 모든 값이 완료되면 publisher1의 끝에 추가
publisher1
    .append(publisher2)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

출력.

```
1
2
3
4
```

# Advanced combining

`prepend`, `append`보다는 복잡하지만, `publisher` `composition`에 유용한 연산자들입니다.

## switchToLatest
(= Rx의 flatMapLatest)

pending 중인 `publisher`의 `subscription`을 취소하는 동시에 전체 `publisher` `subscription`을 즉시 전환하여 최신 `subscription`으로 전환할 수 있습니다.  
자체적으로 `publisher`를 내보내는 `publisher`에서만 사용할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6222a395-6dcf-471a-be9b-e21a514deec0)

위 마블다이어그램을 설명하는 예시입니다.

```swift
// (1) Int를 방출하고, Error가 없는 3개의 publisher 생성
let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<Int, Never>()
let publisher3 = PassthroughSubject<Int, Never>()

// (2) 다른 PassthroughSubject를 방출하는 두 번째 PassthroughSubject를 생성
// publisher1, publisher2, publisher3를 방출시킬 수 있음.
let publishers = PassthroughSubject<PassthroughSubject<Int, Never>, Never>()

// (3) 게시자에서 switchToLatest를 사용.
// publishers를 통해 다른 publisher를 보낼 때마다 새 publisher로 전환하고 이전 subscription을 취소.
publishers
    .switchToLatest()
    .sink(
        receiveCompletion: { _ in print("Completed!") },
        receiveValue: { print($0) }
    )
    .store(in: &subscriptions)

// (4) publisher1을 publishers에게 보낸 다음 1과 2를 publisher1에게 방출
publishers.send(publisher1)
publisher1.send(1)
publisher1.send(2)

// (5) publisher1에 대한 subscription을 취소하는 publisher2를 보냄.
// 그리고 publisher1에게 3을 방출시키지만 무시되고
// publisher2에 대한 활성화된 subscription이 있기 때문에 publisher2에서 4와 5를 방출
publishers.send(publisher2)
publisher1.send(3)
publisher2.send(4)
publisher2.send(5)

// (6) publisher2에게 subscription을 취소하는 publisher3을 보냄.
// 6을 Publisher2에 보내면 무시됨.
// 그런 다음 7, 8, 9를 보내면 Publisher3에 대한 subscription을 통해 방출
publishers.send(publisher3)
publisher2.send(6)
publisher3.send(7)
publisher3.send(8)
publisher3.send(9)

// (7) 현재 Publisher(Publisher3)에게 completion 이벤트를 보내고
// publishers에게 또 다른 completion 이벤트를 보냄 -> 이로써 모든 활성화된 subscription이 완료
publisher3.send(completion: .finished)
publishers.send(completion: .finished)
```

출력.

```
1
2
4
5
7
8
9
Completed!
```

아래 예시와 같이

동일한 네트워크 리퀘스트를 여러번 실행했을 때, 이전에 실행한 리퀘스트를 취소하려는 케이스에 유용합니다.

```swift
let url = URL(string: "https://source.unsplash.com/random")!

// (1) Unsplash의 공개 API에서 임의의 이미지를 가져오기 위해 네트워크 요청을 수행하는 함수 getImage()를 정의
func getImage() -> AnyPublisher<UIImage?, Never> {
    URLSession.shared
        .dataTaskPublisher(for: url)
        .map { data, _ in UIImage(data: data) }
        .print("image")
        .replaceError(with: nil)
        .eraseToAnyPublisher()
}

// (2) PassthroughSubject를 생성하여 사용자가 버튼을 탭하는 것을 대체
let taps = PassthroughSubject<Void, Never>()

// (3) 버튼을 탭하면 getImage()를 호출하여 탭을 임의의 이미지에 대한 새 네트워크 요청에 매핑
// -> Publisher<Void, Never> 를 Publisher<Publisher<UIImage?, Never>, Never>로 매핑
taps
    .map { _ in getImage() }
    .switchToLatest() // (4) Publisher의 Publisher 형태로 매핑되어 switchToLatest() 사용 가능.
    .sink(receiveValue: { _ in })
    .store(in: &subscriptions)

// (5) DispatchQueue를 사용하여 세 번의 지연된 버튼 탭을 재현
taps.send()

DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
    taps.send()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 3.1) {
    taps.send()
}
```

출력 확인.

```
image: receive subscription: (DataTaskPublisher)
image: request unlimited
image: receive value: (Optional(<UIImage:0x600000268000 anonymous {1080, 608} renderingMode=automatic(original)>))
image: receive finished
image: receive subscription: (DataTaskPublisher)
image: request unlimited
image: receive cancel
image: receive subscription: (DataTaskPublisher)
image: request unlimited
image: receive value: (Optional(<UIImage:0x6000002658c0 anonymous {1080, 1620} renderingMode=automatic(original)>))
image: receive finished
```

2개의 이미지를 받은 value history가 확인됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/35c4285a-593e-4ce3-b1e9-d1e0567f9385)


## merge(with:)
(= `Rx`의 `merge`)

merge(with:)는 아래 마블다이어그램과 같이

동일한 타입의 여러 `publisher`의 방출을 interleave(상호배치)시켜줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/25ca440d-8031-4dbd-973f-eba3135e1b38)

위 마블다이어그램의 예시입니다.

```swift
// (1) Int 값을 방출하고 Error가 없는 두 개의 PassthroughSubject
let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<Int, Never>()

// (2) publisher1을 publisher2와 merge하여 두 publisher 모두에서 방출한 값을 interleave시킴(최대 8명의 다른 게시자를 병합 가능)
publisher1
    .merge(with: publisher2)
    .sink(
        receiveCompletion: { _ in print("Completed") },
        receiveValue: { print($0) }
    )
    .store(in: &subscriptions)

// (3) publisher1에 1과 2를 추가하고
publisher1.send(1)
publisher1.send(2)

// publisher2에 3을 추가한 다음
publisher2.send(3)

// 다시 publisher1에 4를 추가하고
publisher1.send(4)

// 마지막으로 publisher2에 5를 추가
publisher2.send(5)

// (4) publisher1과 publisher2 모두에게 completion 이벤트
publisher1.send(completion: .finished)
publisher2.send(completion: .finished)
```

출력.

```
1
2
3
4
5
Completed
```

## combineLatest
(= `Rx`의 `combineLatest`와 유사)

`combineLatest`는 다양한 타입의 `publisher`를 튜플로 묶을 수 있게 해주는 연산자입니다.

`publisher` 중 하나에서 값을 방출할 때마다, 모든 `publisher`의 최신값이 포함된 튜플을 방출시켜줍니다.

> ⚠️ combineLatest로 묶인 모든 publisher들이 최소 1개 이상의 값을 방출하고부터 튜플이 방출됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1e3043b8-41b3-4db4-abe9-0beb86b9b2e5)

위 마블다이어그램의 예시입니다.

```swift
// (1) 2개의 다른 타입의 publisher 생성
let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<String, Never>()

// (2) Publisher2의 최신 방출값을 Publisher1과 결합.
// (CombineLatest의 여러 오버로드를 사용하여 최대 4개의 게시자를 결합 가능)
publisher1
    .combineLatest(publisher2)
    .sink(
        receiveCompletion: { _ in print("Completed") },
        receiveValue: { print("P1: \($0), P2: \($1)") }
    )
    .store(in: &subscriptions)

// (3) 1과 2를 publisher1에 방출하고
publisher1.send(1)
publisher1.send(2)

// "a"와 "b"를 publisher2로 방출
publisher2.send("a")
publisher2.send("b")

// 3을 publisher1로 방출
publisher1.send(3)

// "c"를 publisher2로 방출
publisher2.send("c")

// (4) 두 publisher 모두 completion
publisher1.send(completion: .finished)
publisher2.send(completion: .finished)
```

출력.

`publisher2`가 최초로 `a`를 방출하고 `publisher1`의 최신값 `2`와 튜플로 방출된 것이 확인됩니다.

```
P1: 2, P2: a
P1: 2, P2: b
P1: 3, P2: b
P1: 3, P2: c
Completed
```

## zip
(= `Rx`의 `zip`)

Swift 표준 라이브러리의 `zip`과 유사합니다.

위 `combineLatest`와 달리, `zip`으로 묶인 `publisher`들의 방출값들을 각 인덱스 순서대로 튜플로 묶어 방출시켜줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6a6b45c1-cfa2-4c68-b71d-0f846149d68e)

아래는 예시입니다.

```swift
// (1) 2개의 다른 타입의 publisher 생성
let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<String, Never>()

// (2) 2개의 publisher를 zip으로 묶음
publisher1
    .zip(publisher2)
    .sink(
        receiveCompletion: { _ in print("Completed") },
        receiveValue: { print("P1: \($0), P2: \($1)") }
    )
    .store(in: &subscriptions)

// (3) 1과 2를 publisher1에서 방출
publisher1.send(1)
publisher1.send(2)
// "a"와 "b"를 publisher2에서 방출
publisher2.send("a")
publisher2.send("b")
// 3을 다시 publisher1에서 방출
publisher1.send(3)
// "c"와 "d"를 publisher2에서 방출
publisher2.send("c")
publisher2.send("d")

// (4) 두 publisher를 모두 completion
publisher1.send(completion: .finished)
publisher2.send(completion: .finished)
```

출력.

방출한 값들이 방출한 순서에 맞춰서 튜플로 방출되는 것이 확인됩니다.

```
P1: 1, P2: a
P1: 2, P2: b
P1: 3, P2: c
Completed
```

