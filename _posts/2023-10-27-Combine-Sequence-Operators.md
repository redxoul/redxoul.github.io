---
title: (Combine) Sequence Operators
author: Cody
date: 2023-10-27 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---

시퀀스 연산자들은 Publisher자체가 시퀀스라고 보면됨.
시퀀스 연산자들은 개별 방출값이 아닌 Publisher 전체를 다룸.

## 값 찾기

### min()

Publisher가 Finish될 때까지 기다렸다가, 최소값만 방출시켜줌.

`Comparable 프로토콜`을 따르는 경우 별도의 파라미터없이 최소값을 찾아줌.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/26a286ce-cb8c-4c51-808b-d100450b6ccd)


```swift
var subscriptions = Set<AnyCancellable>()

// 1. 숫자를 방출하는 Publisher
let publisher = [1, -50, 246, 0].publisher

// 2. min 연산자로 최소값을 찾아서 print
publisher
    .print("publisher")
    .min()
    .sink(receiveValue: { print("최소값은 \($0)") })
    .store(in: &subscriptions)
```

출력으로 확인.

```
publisher: receive subscription: ([1, -50, 246, 0])
publisher: request unlimited
publisher: receive value: (1)
publisher: receive value: (-50)
publisher: receive value: (246)
publisher: receive value: (0)
publisher: receive finished
최소값은 -50
```

`Comparable 프로토콜`을 따르지 않는 경우에는?

`min(by:)` 연산자로 클로저를 통해 비교할 수 있음.

```swift
// 1. 문자열 배열로 생성한 Publisher<Data, Never>
let publisher = ["12345",
                 "ab",
                 "hello world"]
    .map { Data($0.utf8) } // [Data]
    .publisher // Publisher<Data, Never>

// 2. min(by:) 연산자로 byte수가 가장 작은 data를 찾은 뒤 string으로 print
publisher
    .print("publisher")
    .min(by: { $0.count < $1.count })
    .sink(receiveValue: { data in
        // 3
        let string = String(data: data, encoding: .utf8)!
        print("최소값은 \(string), \(data.count) bytes")
    })
    .store(in: &subscriptions)
```

출력으로 확인.

```
publisher: receive subscription: ([5 bytes, 2 bytes, 11 bytes])
publisher: request unlimited
publisher: receive value: (5 bytes)
publisher: receive value: (2 bytes)
publisher: receive value: (11 bytes)
publisher: receive finished
최소값은 ab, 2 bytes
```

---

### max()

`min`과 유사하게 upstream Publisher가 finish되면 최대값만 방출시켜주는 연산자.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/2e78473a-2de9-4fe9-a8f6-af81e1032bdc)


```swift
// 1. 4개의 글자를 방출하는 Publisher
let publisher = ["A", "F", "Z", "E"].publisher

// 2. max 연산자로 최대값을 찾아서 pring
publisher
    .print("publisher")
    .max()
    .sink(receiveValue: { print("최대값은 \($0)") })
    .store(in: &subscriptions)
```

출력값 확인.

```
publisher: receive subscription: (["A", "F", "Z", "E"])
publisher: request unlimited
publisher: receive value: (A)
publisher: receive value: (F)
publisher: receive value: (Z)
publisher: receive value: (E)
publisher: receive finished
최대값은 Z
```

min 연산자와 마찬가지로 `comparable 프로토콜`을 따르지 않을 때 비교할 수 있는 `max(by:)` 연산자가 있지만 생략.

---

### first()

(Rx의 `first()`)

upsteam Publisher의 첫번째 방출값을 통과시키고, 바로 subscription을 `cancel`시킴.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/c58cbd4c-fe08-4099-92a0-6f425439b9eb)


```swift
// 1. 3개의 글자를 방출하는 Publisher
let publisher = ["A", "B", "C"].publisher

// 2. first연산자로 첫번째 값을 통과시키고 cancel
publisher
    .print("publisher")
    .first()
    .sink(receiveValue: { print("첫번째 값은 \($0)") })
    .store(in: &subscriptions)
```

출력값으로 동작을 확인.

```
publisher: receive subscription: (["A", "B", "C"])
publisher: request unlimited
publisher: receive value: (A)
publisher: receive cancel
첫번째 값은 A
```

조건에 일치하는 첫번째 값을 통과시키고 싶으면 `first(where:)` 연산자를 사용.

```swift
// 1. 4개의 글자를 방출하는 Publisher
let publisher = ["J", "O", "H", "N"].publisher

// 2. first(where:)연산자로 Hello World에 포함된 첫번째 글자를 찾고 print
publisher
    .print("publisher")
    .first(where: { "Hello World".contains($0) })
    .sink(receiveValue: { print("첫번째 일치값은 \($0)") })
    .store(in: &subscriptions)
```

출력으로 동작을 확인.

```
publisher: receive subscription: (["J", "O", "H", "N"])
publisher: request unlimited
publisher: receive value: (J)
publisher: receive value: (O)
publisher: receive value: (H)
publisher: receive cancel
첫번째 일치값은 H
```

---

### last()

(Rx의 `takeLast()`)

Upstream Publisher가 complete될 때까지 기다렸다가, 마지막 값만 내보내는 연산자.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/d1c5a9f0-38e9-4963-8f39-c93d96460083)


```swift
// 1. 3개의 글자를 방출하는 Publisher
let publisher = ["A", "B", "C"].publisher

// 2. complete을 기다렸다가 마지막값만 print
publisher
    .print("publisher")
    .last()
    .sink(receiveValue: { print("마지막 값은 \($0)") })
    .store(in: &subscriptions)
```

출력으로 확인.

```
publisher: receive subscription: (["A", "B", "C"])
publisher: request unlimited
publisher: receive value: (A)
publisher: receive value: (B)
publisher: receive value: (C)
publisher: receive finished
마지막 값은 C
```

조건에 맞는 마지막 값을 방출시키는 `last(where:)` 연산자도 있음.

---

### output(at:)

(Rx의  `element(at:)`)

지정된 index에 Upstream Publisher가 방출한 값을 찾아서 통과시키고, 바로 subscription을 `cancel`시킴.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/f9e6cd33-20bc-4fe3-b254-b0e635133ecc)


```swift
// 1. 3개의 글자를 방출하는 Publisher
let publisher = ["A", "B", "C"].publisher

// 2. output(at:)으로 upstream publisher의 index 1의 방출만 통과시키고, cancel시킴
publisher
    .print("publisher")
    .output(at: 1)
    .sink(receiveValue: { print("index 1의 값: \($0)") })
    .store(in: &subscriptions)
```

출력으로 동작을 확인.

```
publisher: receive subscription: (["A", "B", "C"])
publisher: request unlimited
publisher: receive value: (A)
publisher: request max: (1) (synchronous)
publisher: receive value: (B)
index 1의 값: B
publisher: receive cancel
```

---

### output(in:)

지정된 index 범위 내의 개별 값을 통과시키고, 바로 subscription을 `cancel`시킴.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/2363bb4c-7c37-4062-9448-00f24b21cd77)


```swift
// 1. 5개의 글자를 방출하는 Publisher
let publisher = ["A", "B", "C", "D", "E"].publisher

// 2. output(in:)으로 upstream publisher의 index 1...3의 방출만 통과시키고, cancel시킴
publisher
    .output(in: 1...3)
    .sink(
        receiveCompletion: { print($0) },
        receiveValue: { print("범위내의 값: \($0)") }
    )
    .store(in: &subscriptions)
```

출력으로 동작을 확인

```
범위내의 값: B
범위내의 값: C
범위내의 값: D
finished
```

---

## Publisher의 쿼리

Publisher가 내보내는 전체 값 집합을 처리하여 쿼리한 값을 내보내줌.

### count()

Upstream Publisher가 `.finish` completed 이벤트를 보내면 방출한 값의 갯수를 내보내줌.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/1773283a-94b1-4e0e-bf4b-7fb3f45e7934)


```swift
// 1. 3개의 글자를 방출하는 Publisher
let publisher = ["A", "B", "C"].publisher

// 2. count()로 upstream publisher가 방출한 값의 갯수를 출력
publisher
    .print("publisher")
    .count()
    .sink(receiveValue: { print("\($0)개의 아이템을 받음") })
    .store(in: &subscriptions)
```

출력으로 확인

```
publisher: receive subscription: (["A", "B", "C"])
publisher: request unlimited
publisher: receive value: (A)
publisher: receive value: (B)
publisher: receive value: (C)
publisher: receive finished
3개의 아이템을 받음
```

---

### contains()

Upstream Publisher가 지정한 값을 방출하면 `true`를 반환 후 즉시 `cancel`시키고, 지정한 값이 하나도 없으면 `false`를 반환.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/e289f55f-c052-4e93-989e-1f03f8d12871)


```swift
// 1. 5개의 글자를 방출하는 Publisher
let publisher = ["A", "B", "C", "D", "E"].publisher
let letter = "C"

// 2. contains()를 사용해서 upstream publishe가 letter를 보냈는지를 체크
publisher
    .print("publisher")
    .contains(letter)
    .sink(receiveValue: { contains in
        // 3. letter가 포함되면 print
        print(contains ? "Publisher가 '\(letter)'를 방출함"
              : "Publisher가 '\(letter)'를 방출 안함")
    })
    .store(in: &subscriptions)
```

출력으로 동작을 확인.

```
publisher: receive subscription: (["A", "B", "C", "D", "E"])
publisher: request unlimited
publisher: receive value: (A)
publisher: receive value: (B)
publisher: receive value: (C)
publisher: receive cancel
Publisher가 'C'를 방출함
```

letter를 `F`로 바꾸면,

```swift
    let letter = "F"
```

finish될 때까지 기다렸다가 `false`를 받음을 확인.

```
publisher: receive subscription: (["A", "B", "C", "D", "E"])
publisher: request unlimited
publisher: receive value: (A)
publisher: receive value: (B)
publisher: receive value: (C)
publisher: receive value: (D)
publisher: receive value: (E)
publisher: receive finished
Publisher가 'F'를 방출 안함
```

이전 연산자들처럼 `comparable 프로토콜`을 따르지 않는 것을 `contains(where:)`로 찾을 수 있음.

```swift
// 1. id, name을 포함한 Almond struct
struct Almond {
    let id: Int
    let name: String
}

// 2.3개의 Almond를 방출하는 publisher
let almonds = [(123, "Honey butter"),
               (777, "Wasabi"),
               (214, "Mint choco")]
    .map(Almond.init)
    .publisher

// 3. contains(where:)로 아몬드 id가 214인지 체크
almonds
    .contains(where: { $0.id == 214 })
    .sink(receiveValue: { contains in
        // 4
        print(contains ? "민초 노노!"
              : "민초만 아니면 되!")
    })
    .store(in: &subscriptions)
```

출력으로 결과 확인.

```
민초 노노!
```

---

### allSatisfy()

Upstream Publisher가 클로저의 조건을 모두 만족하는지 여부를 방출.

Upstream이 `.finish`로 completed될 때까지 기다렸다가 동작.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/fe0421d2-8288-4ba2-8f6b-e7a745c6c70c)


```swift
// 1. 0~5사이 2씩 증가시키며 방출시키는 publisher
let publisher = stride(from: 0, to: 5, by: 2).publisher

// 2. allSatisfy()로 모든 방출된 값이 짝수인지 체크
publisher
    .print("publisher")
    .allSatisfy { $0 % 2 == 0 }
    .sink(receiveValue: { allEven in
        print(allEven ? "모든 값이 짝수!"
              : "어떤 값은 홀수!")
    })
    .store(in: &subscriptions)
```

출력으로 확인.

```
publisher: receive subscription: (Sequence)
publisher: request unlimited
publisher: receive value: (0)
publisher: receive value: (2)
publisher: receive value: (4)
publisher: receive finished
모든 값이 짝수!
```

publisher를 수정해서 하나라도 값이 일치하지 않으면,

```swift
let publisher = stride(from: 0, to: 5, by: 1).publisher // 수정
```

```
어떤 값은 홀수!
```

---

### reduce()

(Rx의 `reduce()`)

Swift에서 제공하는 고차함수인 `reduce`와 동일한 연산자.

Upstream Publisher가 방출하는 값들을 누적시키며 하나의 결과를 만들어 방출.

Upstream이 `.finished`를 전달해야 결과값을 방출.

([이전글](https://swiftycody.github.io/posts/Combine-Transforming-Operators/)에서 정리한 scan연산자와 비슷하지만, scan은 모든 단계마다 누적값을 방출시키고 reduce는 최종 누적값만 방출시킴)

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/1947e9b0-73a3-4315-ab14-ea10c1e47bb0)


```swift
// 1. 문자열을 방출하는 publisher
let publisher = ["Hel", "lo", " ", "Wor", "ld", "!"].publisher

publisher
    .print("publisher")
    .reduce("") { accumulator, value in
        // 2. reduce()로 기존 누적값과 방출된 값을 붙임
        accumulator + value
    }
    .sink(receiveValue: { print("Reduced into: \($0)") })
    .store(in: &subscriptions)
```

출력으로 확인.

```
publisher: receive subscription: (["Hel", "lo", " ", "Wor", "ld", "!"])
publisher: request unlimited
publisher: receive value: (Hel)
publisher: receive value: (lo)
publisher: receive value: ( )
publisher: receive value: (Wor)
publisher: receive value: (ld)
publisher: receive value: (!)
publisher: receive finished
Reduced into: Hello World!
```
