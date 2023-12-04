---
title: (WWDC23) What's new in Swift 5.9
author: Cody
date: 2023-06-10 00:34:00 +0800
categories: [iOS]
tags: [Swift, Swift5.9, WWDC, WWDC23]
pin: false
mermaid: true
---
[What’s new in Swift - WWDC23 - Videos - Apple Developer](https://developer.apple.com/wwdc23/10164)

## if/else문의 향상

복잡한 조건을 기반으로한 **let** 변수를 초기화하려면 아래와 같은 복잡한 삼항 표현식이 나올수도 있습니다(있나!?).

```swift
let bullet =
  isRoot && (count == 0 || !willExpand) ? ""
    : count == 0    ? "- "
    : maxDepth <= 0 ? "▹ " : "▿ "
```

이를 **if 표현식**으로 좀 더 친숙하고 읽기 쉬운 **if문 체인**으로 변경할 수 있습니다.

```swift
let bullet =
    if isRoot && (count == 0 || !willExpand) { "" }
    else if count == 0 { "- " }
    else if maxDepth <= 0 { "▹ " }
    else { "▿ " }
```

전역변수나 저장 프로퍼티를 초기화하는 경우 이런 방법이 도움이 될 수 있습니다.

```swift
let attributedName = AttributedString(markdown: displayName)
```

위와 같은 단일 표현이 잘 동작할 수도 있지만, 특정 조건을 원하는 경우 아래처럼 표현을 했어야 했는데,

```swift
let attributedName = {
    if let displayName, !displayName.isEmpty {
        AttributedString(markdown: displayName)
    } else {
        "Untitled"
    }
}()
```

이제 **if 문이 표현식이 될 수 있기 때문**에 아래처럼 표현이 가능합니다.

```swift
let attributedName =
    if let displayName, !displayName.isEmpty {
        AttributedString(markdown: displayName)
    } else {
        "Untitled"
    }
```

---

## Result Builders

유효하지 않은 코드 Type 체킹 속도가 향상되었고, 유효하지 않은 코드에 대한 오류 메세지가 더 명확졌습니다.

```swift
struct ContentView: View {
    enum Destination { case one, two }

    var body: some View {
        List {
            NavigationLink(value: .one) { // 실제로 문제가 되는 코드
                Text("one")
            }
            NavigationLink(value: .two) {
                Text("two")
            }
        }.navigationDestination(for: Destination.self) {
            $0.view // Swift5.7에서 에러가 표시되던 부분
        }
    }
}
```

이전에는 위처럼 일부 유효하지 않은 코드로 인해 Result Builder의 완전 다른 부분에서 잘못된 오류가 발생하는 문제가 있었습니다. 이제는 좀 더 정확한 위치에 컴파일러 진단을 받을 수 있게 됩니다.

---

## Generic with Parameter Pack

이제는 **매개변수의 타입**뿐만 아니라, **매개변수의 갯수**에 대해서도 추상화할 수 있습니다.

예를 들어 아래처럼 여러개의 제네릭 매개변수를 받아, 여러개의 제네릭을 반환하고 싶을 수 있습니다.

```swift
func evaluate<Result>(_:) -> (Result)

func evaluate<R1, R2>(_:_:) -> (R1, R2)

func evaluate<R1, R2, R3>(_:_:_:) -> (R1, R2, R3)

func evaluate<R1, R2, R3, R4>(_:_:_:_:)-> (R1, R2, R3, R4)

func evaluate<R1, R2, R3, R4, R5>(_:_:_:_:_:) -> (R1, R2, R3, R4, R5)

func evaluate<R1, R2, R3, R4, R5, R6>(_:_:_:_:_:_:) -> (R1, R2, R3, R4, R5, R6)
```

하지만 위처럼 하면 6개 이상 매개변수를 전달하려고 할 때마다, 또 새로운 오버로드가 필요해집니다.

**Swift 5.9**부터는 **each** 를 통해서 **매개변수의 갯수를 추상화**할 수 있게 됩니다.

이 컨셉을 'Parameter Pack'이라고 부릅니다.

**Parameter Pack**을 사용하면 위처럼 각 고정 매개변수 길이에 대한 오버로드 함수들을 아래처럼 단일 함수로 나타내는 것이 가능해집니다.

```swift
func evaluate<each Result>(_: repeat Request<each Result>) -> (repeat each Result)
```

타입추론은 **Parameter Pack**을 사용하는 API를 API가 사용하는지 알 필요없이 자연스럽게 사용할 수 있게 해줍니다.

```swift
struct Request<Result> { ... }

struct RequestEvaluator {
  func evaluate<each Result>(_: repeat Request<each Result>) -> (repeat each Result)
}

let results = RequestEvaluator.evaluate(r1, r2, r3) // Type Parameter Pack의 사용
```

(더 자세한 내용은 Generalize APIs using parameter packs 세션 참조)

---

## Macro

**boilerplate** 코드를 제거하고 Swift의 표현력을 올리기 위해 제공.

아래는 **assert**문. 거짓일 때는 프로그램을 중지하지만, 파일과 줄 번호만 얻을 수 있습니다. 

```swift
assert(max(a, b) == c)
```

XCTest는 두 값을 개별적으로 사용하는 assert-equal operation을 제공하므로 문제가 발생시 두 값이 같지 않은 것을 볼 수 있지만 max(a, b)가 잘못되었는지, c가 잘못되었는지, max(a, b)는 어떤 값이었는지, 어떤 값이 잘못되었는지는 알수는 없습니다.

```swift
XCAssertEqual(max(a, b), c) //XCTAssertEqual failed: ("10") is not equal to ("17")
```

**#assert** 매크로를 이용하면 **assertion**이 실패했을 때 더 풍부하게 알려줄 수 있습니다.

```swift
import PowerAssert

#assert(max(a, b) == c)
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/df29ad76-f467-4715-8ba0-0a57bdf173d1)

**assertion** 결과에 영향을 준 각 값과 실패한 **assertion**에 대한 코드를 표시해줍니다.

```swift
public macro assert(_ condition: Bool)
```

**macro** 패키지에서 **assert** 선언을 보면 앞에 **macro** 키워드가 붙은 것 외엔 함수처럼 보여집니다.

```swift
import PowerAssert
#assert(max(a, b)) //Type 'Int' cannot be a used as a boolean; test for '!= 0' instead
```

**macro**를 사용할 때 매개변수 타입이 검사되므로, 잘못 사용한 경우 위와 같이 유용한 **error** 메세지를 받을 수 있게 됩니다.

```swift
public macro assert(_ condition: Bool) = #externalMacro(
  module: “PowerAssertPlugin”,
  type: “PowerAssertMacro"
)
```

대부분의 매크로는 문자열을 통해 **macro** 구현을 위한 모듈 및 유형을 지정하는 **#externalMacro**로 정의됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6447df9f-df00-41e2-9eff-e1b44d79ea7a)

**#externalMacro** 타입은 컴파일러 플러그인 역할을 하는 별도의 프로그램에서 정의됩니다.  
**Swift** 컴파일러는 매크로 사용을 위한 소스 코드를 플러그인에 전달합니다.

플러그인은 새로운 소스 코드를 생성한 다음 **Swift** 프로그램에 다시 통합됩니다.  
여기에서 **macro**는 **assertion**을 개별 값을 캡처하는 코드와 소스 코드에서 표시되어야 하는 위치로 확장합니다.

상용구를 직접 작성하고 싶지는 않겠지만 매크로가 자동으로 작성합니다.

```swift
// Freestanding macro roles

@freestanding(expression)
public macro assert(_ condition: Bool) = #externalMacro(
  module: “PowerAssertPlugin”,
  type: “PowerAssertMacro"
)
```

추가적으로 **assert macro**는 @**freestanding macro**입니다.

이렇게 되면 assert는 `#`구문을 사용하고 해당 구문에서 직접 동작하여 새 코드를 생성시켜주기 때문에 **`freestanding`**(독립형)이라고 불립니다.

```swift
let pred = #Predicate<Person> {
  $0.favoriteColor == .blue
}

let blueLovers = people.filter(pred)
```

새로운 **Foundation Predicate API**가 제공하는 **#Predicate** 예시.

이 **macro**를 사용하면 클로저를 사용하여 **type-safe** 방식으로 조건자를 작성할 수 있습니다.  
결과 조건자 값은 **Swift** 컬렉션 작업인 **SwiftUI** 및 **SwiftData**를 포함하여 여러 다른 **API**와 함께 사용할 수 있습니다.

```swift
// Predicate expression macro

@freestanding(expression) 
public macro Predicate<each Input>(
  _ body: (repeat each Input) -> Bool
) -> Predicate<repeat each Input>
```

이 macro는 프로그램 다른 곳에서 사용할 수 있는 새 Predicate 타입 인스턴스를 반환시켜줍니다.

```swift
enum Path {
  case relative(String)
  case absolute(String)
  
  // boilerplate codes
  var isAbsolute: Bool {
    if case .absolute = self { true }
    else { false }
  }

  var isRelative: Bool {
    if case .relative = self { true }
    else { false }
  }
}
```

위와 같은 boilerplate 코드들은

```swift
@CaseDetection
enum Path {
  case relative(String)
  case absolute(String)
}

let absPaths = paths.filter { $0.isAbsolute }
```

**@CaseDetection macro**로 줄여줄 수 있습니다. 이 macro는 적용한 선언구문(enum문 자체)를 입력으로 받아서 새 코드를 생성시킵니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/74f25789-5434-44d2-bb9b-f5c227cb8ad9)
Expand Macro

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/06d05b90-4ef8-40f3-8ec5-9da1a06a715e)

위 **@CaseDetection** macro는 **@attached(member)** macro인데, 이는 타입이나 extension에서 새 멤버를 생성한다는 의미입니다.

**@attached(peer)** macro는 예를 들어 비동기 메서드의 complete handler 버전을 생성하거나 그 반대로 생성하기 위해 연결된 선언과 함께 새 선언을 추가합니다.

**@attached(accessor)** macro는 저장 프로퍼티를 연산 프로퍼티로 전환할 수 있으며 프로퍼티 액세스에 대한 특정 작업을 수행하거나 프로퍼티 래퍼와 비슷하지만 더 유연한 방식으로 실제 저장소를 추상화하는 데 사용할 수 있습니다.

이처럼 **@attached** 매크로는 유형의 특정 멤버에 프로퍼티를 도입할 수 있을 뿐만 아니라 새로운 프로토콜 적합성을 추가할 수 있습니다.

```swift
final class Person: ObservableObject {
    @Published var name: String
    @Published var age: Int
    @Published var isFavorite: Bool
}

struct ContentView: View {
    @ObservedObject var person: Person
    
    var body: some View {
        Text("Hello, \(person.name)")
    }
}
```

또 한가지 예는 ObservableObject입니다.

기존에 SwiftUI에서 특정 프로퍼티를 Observing하려면 타입을 **ObservableObject**를 준수하도록 만들고 모든 프로퍼티를 **@Published**로 표시하고 View에서 **@ObservedObject** 프로퍼티 래퍼를 사용하기만 하면 됩니다.  
이는 많은 단계이며 단계가 누락되면 UI가 예상대로 업데이트되지 않을 수 있습니다.

```swift
@Observable final class Person {
    var name: String
    var age: Int
    var isFavorite: Bool
}

struct ContentView: View {
    var person: Person
    
    var body: some View {
        Text("Hello, \(person.name)")
    }
}
```

이를 **@Observable** macro를 이용하면 위처럼 간편해질 수 있습니다.

프로퍼티마다 **@Published**를 달 필요가 없고, 사용하는 View에서도 **@Observed**를 달 필요가 없어졌습니다.

```swift
@attached(member, names: ...)
@attached(memberAttribute)
@attached(conformance)
public macro Observable() = #externalMacro(...).
```

**@Observable** macro는 위 세가지 macro 역할의 조합으로 만들어집니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d5e12bed-d86c-4430-b454-75b10ae244a4)

**@attached(member, names: ...)**는 새 프로퍼티와 메서드를 추가하고,

**@attached(memberAttribute)**는 observed 클래스의 저장프로퍼티에 **@ObservationTracked** macro를 추가하여 observation 이벤트를 트리거하는 getter, setter로 확장합니다.

**@attached(conformance)**는 Observable 프로토콜에 대한 준수를 도입(?)합니다.

위처럼 복잡해보일 수 있는 코드들이 @Observable macro 뒤에 숨어져있습니다.

(더 자세한 내용은 'Expand on Swift Macros', 'Write Swift Macros' 세션 참조)

---

## Swift Foundation

이전에 공개했던 Swift Foundation이 MacOS Sonoma, iOS 17, iPadOS 17 부터 도입된다는 내용으로 기존 Foundation보다 성능향상이 크다는 점을 이야기 했습니다.

[Swift로 작성한 Foundation Package Preview가 공개](https://swifty-cody.tistory.com/105)

---

## C++과 Swift의 상호운용성

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8bc3da2c-bba5-427d-aa18-6ba62f55c166)

C++과 상호운용성 세팅을 할 수 있다는 내용입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4706958a-6564-4bf0-8991-7ce74e767475)

위처럼 C++로 작성한 코드를 Swift에서 액세스하고 직접 사용할 수 있게 됩니다.

(자세한 내용은 'Mix Swift and C++' 세션 참조)

---

## 동시성 Concurrency

추상 동시성 모델에는 2가지가 있습니다.

- **Tasks**: 어디에서나 실행할 수 있는 순차적 작업 단위. 프로그램에 **await**가 있을 때마다 작업을 일시중지하고 작업을 계속할 수 있게 되면 다시 시작할 수 있음.
- **Actors**: 격리된 상태에 대한 상호 배타적인 액세스를 제공하는 동기화 메커니즘. 외부에서 액터를 입력하면 작업을 일시 중지할 수 있으므로 **await**가 필요.

**Tasks**와 **Actors**는 추상 언어 모델에 통합되지만 해당 모델 내에서 다양한 환경에 맞게 다양한 방식으로 구현될 수 있습니다.

**Tasks**는 **Global concurrent pool**에서 실행됩니다.

**Global concurrent pool**이 작업 일정을 결정하는 방법은 환경에 달려 있습니다.  
Apple 플랫폼의 경우 **Dispatch** 라이브러리는 전체 운영 체제에 최적화된 일정을 제공하며 각 플랫폼에 대해 광범위하게 조정되었습니다.  
보다 제한적인 환경에서는 다중 스레드 스케줄러의 오버헤드가 허용되지 않을 수 있습니다.

**Swift**의 동시성 모델은 Single-Thread 협력 Queue로 구현됩니다.  
추상 모델은 다양한 런타임 환경에 매핑할 수 있을 만큼 유연하기 때문에 동일한 **Swift** 코드가 두 환경에서 모두 작동합니다.  
또한 콜백 기반 라이브러리와의 상호 운용성은 처음부터 **Swift**의 **async/await** 지원에 내장되었습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/cd80debb-baff-41e4-830c-61cd372e87b3)

**withCheckedContinuation** 작업을 통해 **Task**를 일시정시했다가, 나중에 콜백에 대한 **response**로 다시 시작할 수 있습니다.

이를 통해 **Task** 자체를 관리하는 기존 라이브러리와의 통합이 가능합니다.  
**Swift** 동시성 런타임에서 **Actor**의 표준 구현은 **Actor**에서 실행할 **Task**의 잠금 없는 대기열이지만 가능한 유일한 구현은 아닙니다.  
보다 제한된 환경에서는 **Atomic**하지 않을 수 있고 대신 **spinlock**과 같은 다른 동시성 **primitive**를 사용할 수 있습니다.

해당 환경이 **Single-thread**인 경우 **synchronization**이 필요하지 않지만 **Actor** 모델은 관계없이 프로그램에 대한 추상 동시성 모델을 유지합니다.  
동일한 코드를 **multi-thread**인 다른 환경으로 계속 가져올 수 있습니다.  
**Swift 5.9**에서는 커스텀 **Actor** **excutor**를 통해 특정 **Actor**가 자체 **synchronization** 메커니즘을 구현할 수 있습니다.

이를 통해 **Actor**는 기존 환경에 더 유연하고 적응할 수 있습니다.

```swift
// Custom actor executors

actor MyConnection {
  private var database: UnsafeMutablePointer<sqlite3>
  
  init(filename: String) throws { … }
  
  func pruneOldEntries() { … }
  func fetchEntry<Entry>(named: String, type: Entry.Type) -> Entry? { … }
}

await connection.pruneOldEntries()
```

위는 데이터베이스 연결을 관리하는 **Actor**의 예시.

**Swift**는 이 **Actor**의 **storage**에 대한 상호 배타적 액세스를 보장하므로 데이터베이스에 대한 동시 액세스가 없습니다.  
그러나 동기화가 수행되는 특정 방식에 대해 더 많은 제어가 필요한 경우에는? 예를 들어 데이터베이스 연결을 위해 특정 **Dispatch Queue**를 사용하려는 경우 아마도 해당 큐가 **Actor**를 채택하지 않은 다른 코드와 공유되기 때문일 것입니다.

```swift
actor MyConnection {
  private var database: UnsafeMutablePointer<sqlite3>
  private let queue: DispatchSerialQueue // 추가
  
  nonisolated var unownedExecutor: UnownedSerialExecutor { queue.asUnownedSerialExecutor() } // 추가

  init(filename: String, queue: DispatchSerialQueue) throws { … }
  
  func pruneOldEntries() { … }
  func fetchEntry<Entry>(named: String, type: Entry.Type) -> Entry? { … }
}

await connection.pruneOldEntries()​
```

커스텀 **Actor Excutor**를 사용하면 가능합니다.

여기서 **Actor**에 **DispatchSerialQueue**를 추가하고 해당 **Dispatch Queue**에 해당하는 **Excutor**를 생성하는 **UnownedSerialExcutor** 프로퍼티의 구현을 추가했습니다.

이 변경으로 **Actor** 인스턴스에 대한 모든 **synchronization**는 해당 **Queue**을 통해 발생합니다.  
**Actor** 외부에서 **pruneOldEntries**에 대한 호출을 **await**하면 이제 해당 **Queue**에서 **dispatch-async**를 수행합니다.

이렇게 하면 개별 **Actor**가 **synchronization**를 제공하는 방법을 더 잘 제어할 수 있으며, Objective-C 또는 C++로 작성되었기 때문에 아직 **Actor**를 사용하지 않는 다른 코드와 **Actor**를 동기화할 수도 있습니다.

```swift
// Executor protocols

protocol Executor: AnyObject, Sendable {
  func enqueue(_ job: consuming ExecutorJob)
}

protocol SerialExecutor: Executor {
  func asUnownedSerialExecutor() -> UnownedSerialExecutor
  func isSameExclusiveExecutionContext(other executor: Self) -> Bool
}

extension DispatchSerialQueue: SerialExecutor { … }
```

**DispatchQueue**가 새로운 **SerialExecutor** **protocol**을 준수하기 때문에 **DispatchQueue**를 통한 **Actor**의 **synchronization**가 가능합니다.

몇 가지 핵심 작업만 있는 이 **protocol**을 준수하는 새 type을 정의하여 **Actor**와 함께 사용할 고유한 **synchronization** 메커니즘을 제공할 수 있습니다. **Excutor**의 컨텍스트에서 코드가 이미 실행되고 있는지 확인합니다.  
예를 들어 메인 스레드에서 실행하고 있으면? 과도한 **reference-counting** 트래픽 없이 액세스할 수 있도록 **Excutor**에 대한 **unowned-reference**를 추출합니다.

그리고 가장 핵심적인 작업인 **enqueue**는 **Excutor** '**Job(작업)'**의 소유권을 가져옵니다. **Job**은 **Excutor**에서 **synchronous**하게 실행해야 하는 **asynchronous** **task**의 일부입니다.  
**enqueue**가 호출되는 시점에서 **Serial** **Excutor**에서 실행 중인 다른 코드가 없을 때 해당 **Job**을 실행하는 것은 **Excutor**의 책임입니다.  
예를 들어 **Dispatch Queue**에 대한 **enqueue**는 해당 **Queue**에서 **dispatch async**를 호출합니다.

(자세한 내용은 "Behind the Scene(WWDC21)", "Beyond the basics of Structured Concurrency(WWDC23)"를 참조)

---

## FoundationDB

상용 하드웨어에서 실행되고 **MacOS**, **Linux**, **Windows**를 포함한 다양한 플랫폼을 지원하는 매우 큰 key-value 저장소를 위한 확장가능한 솔루션을 제공하는 분산형 **DB**. **C++**로 작성된 대규모 코드베이스가 포함된 오픈소스 프로젝트

(..인데, 비동기처리에 대한 코드가 C++보다 Swift로 작성하는 것이 더 깔끔하고 좋다는 내용인 듯합니다)

(자세한 내용은 생략합니다!)