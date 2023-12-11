---
title: (Combine) Time Manipulation Operators
author: Cody
date: 2023-10-20 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---

반응형 프로그래밍의 핵심 아이디어는 시간에 따른 비동기 이벤트 흐름을 모델링하는 것.

Combine에서는 시퀀스가 시간에 따라 값에 반응하고 변환하는 다양한 연산자들을 제공.

---

## Time shifting

### delay(for:tolerance:scheduler:options)

(Rx의 delay)  
`Upstream Publisher`가 값을 내보낼 때마다 `delay` 연산자는 잠시 동안 값을 유지한 다음 사용자가 지정한 스케줄러에서 요청한 지연 시간 후에 값을 내보냄.

초마다 하나의 값을 내보내는 퍼블리셔를 만든 다음 1.5초씩 지연시키고 두 타임라인을 동시에 표시하여 비교.

```swift
import Combine
import SwiftUI
import PlaygroundSupport

var subscriptions = Set<AnyCancellable>()

let valuesPerSecond = 1.0
let delayInSeconds = 1.5

// 1. sourcePublisher는 타이머가 내보내는 날짜를 공급할 Subject
let sourcePublisher = PassthroughSubject<Date, Never>()

// 2. delayedPublisher는 sourcePublisher에서 값을 delay시켜 main scheduler에서 값을 방출
let delayedPublisher = sourcePublisher.delay(for: .seconds(delayInSeconds),
                                             scheduler: DispatchQueue.main)

// 3. main thread에서 초당 하나의 값을 전달하는 타이머를 만듦
// autoconnect()를 사용해서 즉시 시작하고 sourcePublisher를 통해 방출되는 값을 제공
let subscription = Timer
    .publish(every: 1.0 / valuesPerSecond, 
             on: .main,
             in: .common)
    .autoconnect()
    .subscribe(sourcePublisher)
```

아래는 이벤트를 시각화할 `TimelineView` 세팅.

```swift
// 4.Timer의 값을 표시할 TimelineView 생성
let sourceTimeline = TimelineView(title: "방출된 값 (\(valuesPerSecond)초마다 방출):")

// 5. Delay된 Value를 표시할 TimelineView 생성
let delayedTimeline = TimelineView(title: "Delay된 값들 (\(delayInSeconds)초 delay):")

// 6. 위 두 TimelineView를 VStack으로
let view = VStack(spacing: 50) {
    sourceTimeline
    delayedTimeline
}

// 7. PlaygroundPage에 liveView 설정
PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))
```

위 `TimeLineView`는 아래처럼 source Publisher를 Publisher extension으로 displayEvents로 넣어주면

오른쪽에서부터 순서대로 그려주는 뷰. 상세 구현은 생략.

```swift
sourcePublisher.displayEvents(in: sourceTimeline)
delayedPublisher.displayEvents(in: delayedTimeline)
```

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/fe049d91-13f2-4ba3-87fc-2620ab0cfe89)


---

## Collecting Values

### collect(\_:options:) - strategy를 byTime으로

(Rx의 toArray)  
지정된 시간간격으로 `Publisher`로부터 값을 collect해야할 때에 필요. buffering의 형태.

(Transform Operator 중 하나인 `collect`의 오버로드. 기존 `collect`는 지정된 갯수만큼 묶어 배열로 방출하지만)

이 `collect`는 지정된 시간 단위로 묶어 배열로 방출.

```swift
let valuesPerSecond = 1.0
let collectTimeStride = 4

// 1. Timer가 방출할 값을 전달할 sourcePublisher
let sourcePublisher = PassthroughSubject<Date, Never>()

// 2. colect 연산자로 'collectTimeStride의 보폭 동안 수신하는 값을 수집하는 collectedPublisher'를 생성
// 연산자는 이러한 값 그룹을 지정된 스케줄러(DispatchQueue.main)에 배열로 방출
let collectedPublisher = sourcePublisher
    .collect(.byTime(DispatchQueue.main, .seconds(collectTimeStride)))
```

방출된 값과 `collect`된 값들의 방출을 `TimelineView`로 확인.

```swift
// 방출된 값과 collect된 값들의 방출을 보여줄 TimelineView들
let sourceTimeline = TimelineView(title: "방출된 값들:")
let collectedTimeline = TimelineView(title: "Collect된 값들 (매 \(collectTimeStride)초마다):")

let view = VStack(spacing: 40) {
    sourceTimeline
    collectedTimeline
}

PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))

// TimelineView들에 각 소스들 표시
sourcePublisher.displayEvents(in: sourceTimeline)
collectedPublisher.displayEvents(in: collectedTimeline)
```

4초마다 `collect`된 배열이 방출되는 것을 확인.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/f305e69e-7836-45a2-a88b-3f4b0e2947d3)

배열내 값을 확인해보기 위해 `collectedPublisher`에 `flatmap`을 추가.

```swift
let collectedPublisher = sourcePublisher
    .collect(.byTime(DispatchQueue.main, .seconds(collectTimeStride)))
    .flatMap { dates in dates.publisher } // 추가
```

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/73c9f38d-d16f-46b7-a1be-2702445565ac)


### collect(\_:options:) - strategy를 byTimeOrCount로

`strategy`를 `byTimeOrCount`로 하면,

일정 시간 간격으로 방출된 값을 collect하면서 값의 갯수를 제한할 수 있음.

위 `collect` 예제에서 제한할 갯수(`collectMaxCount`)를 추가하고,

```swift
let collectMaxCount = 2 // 추가
```

`byTimeOrCount`로 수집할 `Publisher`도 추가하고,

```swift
let collectedPublisher2 = sourcePublisher // 추가
    .collect(.byTimeOrCount(DispatchQueue.main,
                            .seconds(collectTimeStride),
                            collectMaxCount))
    .flatMap { dates in dates.publisher }
```

`TimelineView`도 추가하고,

```swift
let collectedTimeline2 = TimelineView(title: "Collect된 값들 (최대 \(collectMaxCount)개씩 매 \(collectTimeStride)초마다):") // 추가

let view = VStack(spacing: 40) {
    sourceTimeline
    collectedTimeline
    collectedTimeline2 // 추가
}
```

`TimelineView`에 `Publisher` 추가하고 타임라인 확인.

```swift
collectedPublisher2.displayEvents(in: collectedTimeline2) // 추가
```

최대갯수(`2`)가 될 때와 Time(`4초`)가 될 때마다 값을 방출하는 것을 확인.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/9bf17b1a-1007-4619-8c65-defc967efc47)


---

## Event의 보류(Holding off)

텍스트필드에서 값이 연속으로 들어올 때마다 반응하지 않고,

한동안 값을 입력하지 않았을 때만 입력했던 값을 받아 처리하는 것과 같이 이벤트를 Holding Off시키고 싶을 때 필요한 오퍼레이터.

`Combine`에서는 `debounce`와 `throttle`을 제공.

### debounce(for:scheduler:)

(Rx의 debounce)  
받은 값을 1초마다 방출시키는 `debounce` 예시.

```swift
// 1. String을 방출할 subject
let subject = PassthroughSubject<String, Never>()

// 2. debounce로 1초간 기다렸다가 값을 전달.
// 1초 간격으로 전달된 마지막 값이 있는 경우 해당값을 방출. 초당 1번만 방출하게 됨.
let debounced = subject
    .debounce(for: .seconds(1.0), scheduler: DispatchQueue.main)
    .share() // 3. debounce된 Publisher를 여러번 subscribe하더라도 일관된 값을 받도록 하기 위해 share 연산자 사용.
```

타이핑을 대신하기 위한 `TimeInterval`과 `String` 배열.

0.6초간 입력 후 멈추었다가 2.0초부터 다시 입력을 시작.

```swift
public let typingHelloWorld: [(TimeInterval, String)] = [
  (0.0, "H"),
  (0.1, "He"),
  (0.2, "Hel"),
  (0.3, "Hell"),
  (0.5, "Hello"),
  (0.6, "Hello "),
  (2.0, "Hello W"),
  (2.1, "Hello Wo"),
  (2.2, "Hello Wor"),
  (2.4, "Hello Worl"),
  (2.5, "Hello World")
]
```

`TimelineView`에 방출된 값과 `Debounce`된 값을 설정하고,

시간 간격을 `print` 하도록 작성.

```swift
let subjectTimeline = TimelineView(title: "값 방출")
let debouncedTimeline = TimelineView(title: "Debounce된 값")

let view = VStack(spacing: 100) {
    subjectTimeline
    debouncedTimeline
}

PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))

subject.displayEvents(in: subjectTimeline)
debounced.displayEvents(in: debouncedTimeline)

let subscription1 = subject
    .sink { string in
        print("+\(deltaTime)초: Subject 방출됨: \(string)")
    }

let subscription2 = debounced
    .sink { string in
        print("+\(deltaTime)초: Debounce후 방출됨: \(string)")
    }

subject.feed(with: typingHelloWorld)
```

입력이 멈추었을 때, `debounce`된 값이 들어오는 것을 확인.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/b257751e-087c-4835-917b-e46ed546bf71)


```
+0.0초: Subject 방출됨: H
+0.1초: Subject 방출됨: He
+0.2초: Subject 방출됨: Hel
+0.3초: Subject 방출됨: Hell
+0.5초: Subject 방출됨: Hello
+0.6초: Subject 방출됨: Hello 
+1.6초: Debounce후 방출됨: Hello 
+2.0초: Subject 방출됨: Hello W
+2.0초: Subject 방출됨: Hello Wo
+2.3초: Subject 방출됨: Hello Wor
+2.5초: Subject 방출됨: Hello Worl
+2.5초: Subject 방출됨: Hello World
+3.5초: Debounce후 방출됨: Hello World
```

### throttle(for:scheduler:latest:)

(Rx의 throttle)  
`throttle`은 지정된 시간 간격만큼 기다린 다음, 해당 간격 중 받았던 가장 첫번째 값(`latest: false`)이나 최신값(`latest: true`)를 받음.

아래는 `throttle`의 예시.

```swift
import Combine
import SwiftUI
import PlaygroundSupport

let throttleDelay = 1.0

// 1. String을 방출할 Subject
let subject = PassthroughSubject<String, Never>()

// 2. latest를 false로 하여 throttled subject는
// 각 1초 간격으로 subject로부터 받은 첫번째값만 방출
let throttled = subject
    .throttle(for: .seconds(throttleDelay), 
              scheduler: DispatchQueue.main,
              latest: false)
    .share() // 3. 모든 subscriber가 throttled subject에서 동일한 값을 share받음
```

그리고 이를 확인하기 위한 `TimelineView` 작성.

```swift
let subjectTimeline = TimelineView(title: "방출된 값")
let throttledTimeline = TimelineView(title: "throttle된 값")

let view = VStack(spacing: 100) {
    subjectTimeline
    throttledTimeline
}

PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))

subject.displayEvents(in: subjectTimeline)
throttled.displayEvents(in: throttledTimeline)

let subscription1 = subject
    .sink { string in
        print("+\(deltaTime)초: Subject 방출됨: \(string)")
    }

let subscription2 = throttled
    .sink { string in
        print("+\(deltaTime)초: Throttled 방출됨: \(string)")
    }

subject.feed(with: typingHelloWorld)
```

방출된 값과 `throttle`된 값 확인.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/e54595bb-36be-4a30-84c7-64da306f47ce)


로그를 통해 `throttle`의 동작을 확인. 1초 간격으로 기다렸다가 그 사이에 제일 먼저 받았던 값(`latest: false`)을 방출.

```swift
+0.0초: Subject 방출됨: H
+0.0초: Throttled 방출됨: H
+0.1초: Subject 방출됨: He
+0.2초: Subject 방출됨: Hel
+0.3초: Subject 방출됨: Hell
+0.5초: Subject 방출됨: Hello
+0.6초: Subject 방출됨: Hello 
+1.1초: Throttled 방출됨: He
+2.0초: Subject 방출됨: Hello W
+2.0초: Subject 방출됨: Hello Wo
+2.1초: Throttled 방출됨: Hello W
+2.3초: Subject 방출됨: Hello Wor
+2.5초: Subject 방출됨: Hello Worl
+2.5초: Subject 방출됨: Hello World
+3.1초: Throttled 방출됨: Hello Wor
```

`throttle`의 `latest`를 `true`로 수정하면

```swift
let throttled = subject
    .throttle(for: .seconds(throttleDelay), 
              scheduler: DispatchQueue.main,
              latest: true)
    .share()
```

로그를 통해 1초 간격으로 기다렸다가 그 사이에 제일 최신값(`latest: true`)을 방출하는 것을 확인.

```
+0.0초: Subject 방출됨: H
+0.0초: Throttled 방출됨: H
+0.1초: Subject 방출됨: He
+0.2초: Subject 방출됨: Hel
+0.3초: Subject 방출됨: Hell
+0.5초: Subject 방출됨: Hello
+0.6초: Subject 방출됨: Hello 
+1.0초: Throttled 방출됨: Hello 
+2.1초: Subject 방출됨: Hello W
+2.1초: Throttled 방출됨: Hello W
+2.2초: Subject 방출됨: Hello Wo
+2.2초: Subject 방출됨: Hello Wor
+2.5초: Subject 방출됨: Hello Worl
+2.5초: Subject 방출됨: Hello World
+3.1초: Throttled 방출됨: Hello World
```

---

## 시간 초과 처리

### timeout

(Rx의 timeout)  
`timeout` 연산자가 실행이 되면 `publisher`가 `complete`되거나 정의된 `error`를 발생시키고 종료시킴.

아래는 `subject`가 5초간 아무값도 발생시키지 않았을 때 `timeout`되는 예시.

`subject`가 `error`가 `Never`이기 때문에 failure없이 complete.

```swift
import Combine
import SwiftUI
import PlaygroundSupport

let subject = PassthroughSubject<Void, Never>()

// 1. upstream publisher가 5초간 아무값을 방출하지 않으면 시간 초과가 됩니다.
let timedOutSubject = subject.timeout(.seconds(5), scheduler: DispatchQueue.main)
```

버튼을 누르면 `subject`가 방출하도록 설정하고 `TimelineView` 작성.

```swift
let timeline = TimelineView(title: "Button 탭")

let view = VStack(spacing: 100) {
    // 1. 버튼을 누르면 subject 방출
    Button(action: { subject.send() }) {
        Text("5초 내로 버튼 탭")
    }
    timeline
}

PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))

timedOutSubject.displayEvents(in: timeline)
```

5초 내로 탭하지 않았을 때 `subject`가 `complete`되는 것을 확인.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/3207c61f-889b-4b68-b340-6b7dce30d0c6)


시간 초과 시 에러를 발생시키기 위해서는 아래와 같이 수정.

```swift
enum TimeoutError: Error { // 추가
    case timedOut
}

let subject = PassthroughSubject<Void, TimeoutError>() // TimeoutError로 수정

let timedOutSubject = subject.timeout(.seconds(5),
                                      scheduler: DispatchQueue.main,
                                      customError: { .timedOut }) // customError 추가
```

시간 초과시 에러를 방출하는 것을 확인

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/8d19c46b-94d7-49e7-ae7c-866078d8150b)


---

## 시간 측정

### measureInterval(using:)

시간 조작은 하지 않고 측정만 하는 연산자.

`measureInterval(using:)`는 `Publisher`가 2개의 연속된 값 사이의 경과한 시간을 측정해야할 때 사용할 수 있는 도구.

```swift
import Combine
import SwiftUI
import PlaygroundSupport

let subject = PassthroughSubject<String, Never>()

// 1. subject를 main 큐에서 값을 방출하게 하고, 측정하도록 지정
let measureSubject = subject.measureInterval(using: DispatchQueue.main)

let subjectTimeline = TimelineView(title: "방출된 값")
let measureTimeline = TimelineView(title: "측정된 값")

let view = VStack(spacing: 100) {
    subjectTimeline
    measureTimeline
}

PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))

subject.displayEvents(in: subjectTimeline)
measureSubject.displayEvents(in: measureTimeline)

// subject와 measure 발출된 값 print
let subscription1 = subject.sink {
    print("+\(deltaTime)초: Subject 방출됨: \($0)")
}

let subscription2 = measureSubject.sink {
    print("+\(deltaTime)초: Measure 방출됨: \($0)")
}

subject.feed(with: typingHelloWorld)
```

출력값을 확인.

`TimeInterval`은 제공된 스케줄러(여기서는 `DispatchQueue`)가 제공하는 시간 단위.

`DispatchQueue`의 경우 `TimeInterval`은 `nanoseconds` 단위로 생성된 `DispatchTimeInterval`로 정의.

```
+0.0초: Measure 방출됨: Stride(_nanoseconds: 40468500)
+0.0초: Subject 방출됨: H
+0.1초: Measure 방출됨: Stride(_nanoseconds: 64299875)
+0.1초: Subject 방출됨: He
+0.2초: Measure 방출됨: Stride(_nanoseconds: 102388250)
+0.2초: Subject 방출됨: Hel
+0.3초: Measure 방출됨: Stride(_nanoseconds: 108301875)
+0.3초: Subject 방출됨: Hell
+0.5초: Measure 방출됨: Stride(_nanoseconds: 208391209)
+0.5초: Subject 방출됨: Hello
+0.6초: Measure 방출됨: Stride(_nanoseconds: 105173166)
+0.6초: Subject 방출됨: Hello 
+2.1초: Measure 방출됨: Stride(_nanoseconds: 1469857959)
+2.1초: Subject 방출됨: Hello W
+2.2초: Measure 방출됨: Stride(_nanoseconds: 104921375)
+2.2초: Subject 방출됨: Hello Wo
+2.2초: Measure 방출됨: Stride(_nanoseconds: 785583)
+2.2초: Subject 방출됨: Hello Wor
+2.5초: Measure 방출됨: Stride(_nanoseconds: 315478292)
+2.5초: Subject 방출됨: Hello Worl
+2.5초: Measure 방출됨: Stride(_nanoseconds: 139916)
+2.5초: Subject 방출됨: Hello World
```

보기 좋게 출력하기 위해 아래처럼 수정이 필요.

```swift
let subscription2 = measureSubject.sink {
    print("+\(deltaTime)초: Measure 방출됨: \(Double($0.magnitude) / 1_000_000_000.0)") // 수정
}
```

출력을 확인.

```
+0.0초: Measure 방출됨: 0.024960375
+0.0초: Subject 방출됨: H
+0.1초: Measure 방출됨: 0.077610083
+0.1초: Subject 방출됨: He
+0.2초: Measure 방출됨: 0.107842584
+0.2초: Subject 방출됨: Hel
+0.3초: Measure 방출됨: 0.104186958
+0.3초: Subject 방출됨: Hell
+0.5초: Measure 방출됨: 0.2083225
+0.5초: Subject 방출됨: Hello
+0.6초: Measure 방출됨: 0.10464775
+0.6초: Subject 방출됨: Hello 
+2.1초: Measure 방출됨: 1.471396875
+2.1초: Subject 방출됨: Hello W
+2.2초: Measure 방출됨: 0.103586167
+2.2초: Subject 방출됨: Hello Wo
+2.2초: Measure 방출됨: 0.000129
+2.2초: Subject 방출됨: Hello Wor
+2.5초: Measure 방출됨: 0.316555708
+2.5초: Subject 방출됨: Hello Worl
+2.5초: Measure 방출됨: 0.000130167
+2.5초: Subject 방출됨: Hello World
```

`DispatchQueue`가 아닌 `Runloop` 메인으로 바꿨을 때는?

```swift
let measureSubject2 = subject.measureInterval(using: RunLoop.main) // 추가

let subscription3 = measureSubject2.sink { // 추가
    print("+\(deltaTime)s: Measure2 emitted: \($0)")
}
```

`Runloop`에서는 초단위로 방출이 측정되는 것을 확인.

```
+0.0초: Measure 방출됨: 0.029805917
+0.0s: Measure2 방출: Stride(magnitude: 0.030153989791870117)
+0.0초: Subject 방출됨: H
+0.1초: Measure 방출됨: 0.074119875
+0.1s: Measure2 방출: Stride(magnitude: 0.07328498363494873)
+0.1초: Subject 방출됨: He
+0.2초: Measure 방출됨: 0.103644875
+0.2s: Measure2 방출: Stride(magnitude: 0.1036679744720459)
+0.2초: Subject 방출됨: Hel
+0.3초: Measure 방출됨: 0.108386083
+0.3s: Measure2 방출: Stride(magnitude: 0.1084979772567749)
+0.3초: Subject 방출됨: Hell
+0.5초: Measure 방출됨: 0.208423542
+0.5s: Measure2 방출: Stride(magnitude: 0.20862603187561035)
+0.5초: Subject 방출됨: Hello
+0.6초: Measure 방출됨: 0.10648925
+0.6s: Measure2 방출: Stride(magnitude: 0.10615503787994385)
+0.6초: Subject 방출됨: Hello 
+2.1초: Measure 방출됨: 1.471713875
+2.1s: Measure2 방출: Stride(magnitude: 1.4720739126205444)
+2.1초: Subject 방출됨: Hello W
+2.2초: Measure 방출됨: 0.102555375
+2.2s: Measure2 방출: Stride(magnitude: 0.10231101512908936)
+2.2초: Subject 방출됨: Hello Wo
+2.2초: Measure 방출됨: 0.000357041
+2.2s: Measure2 방출: Stride(magnitude: 0.0001990795135498047)
+2.2초: Subject 방출됨: Hello Wor
+2.5초: Measure 방출됨: 0.315149417
+2.5s: Measure2 방출: Stride(magnitude: 0.3153599500656128)
+2.5초: Subject 방출됨: Hello Worl
+2.5초: Measure 방출됨: 0.000406625
+2.5s: Measure2 방출: Stride(magnitude: 0.00017905235290527344)
+2.5초: Subject 방출됨: Hello World
```

---

## 정리

1. delay(for:tolerance:scheduler:options)
- 스케줄러에 지정한 시간만큼 방출한 값을 유지했다가 해당 시간이 지나면 방출시켜줌.

2. collect(\_:options:)는 strategy에 따라
- byTime를 strategy으로 했을 때, 지정된 시간간격으로 Publisher로부터 값을 collect시켜줌. buffering의 형태.
- byTimeOrCount로 strategy으로 했을 때, 일정 시간 간격으로 방출된 값을 collect하면서 값의 갯수를 제한시켜줌.

3. debounce(for:scheduler:)
- 스케줄러에 지정한 시간만큼 값이 들어오지 않기를 기다렸다가, 값 방출이 멈추기 전까지 받았던 값들을 배열로 한꺼번에 방출시켜줌.

4. throttle(for:scheduler:latest:)
- 지정된 시간 간격만큼 기다린 다음, 해당 간격 중 받았던 가장 첫번째 값(latest: false)이나 최신값(latest: true)를 받음.

5. timeout
- timeout 연산자가 실행이 되면 publisher가 complete되거나 정의된 error를 발생시키고 종료

6. measureInterval(using:)
- Publisher가 2개의 연속된 값 사이의 경과한 시간을 측정해야할 때 사용할 수 있는 도구.
- TimeInterval은 제공된 스케줄러(여기서는 DispatchQueue)가 제공하는 시간 단위(DispatchQueue는 nanoseconds로 runloop는 초단위로 제공)