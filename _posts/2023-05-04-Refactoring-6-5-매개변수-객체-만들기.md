---
title: (Refactoring) 6-5 매개변수 객체 만들기
author: Cody
date: 2023-05-04 00:34:00 +0800
categories: [Refactoring]
tags: [Swift, Refactoring]
pin: false
mermaid: true
---
## **매개변수 객체 만들기**

데이터 항목 여러개를 함수로 넘겨주게 되면, 데이터 구조를 묶어서 보내주면 좋습니다.

- 데이터 뭉치를 데이터 구조로 묶으면 데이터 사이의 관계가 명확해짐
- 함수는 데이터 구조를 받음으로써 매개변수 수가 줄어듦
- 같은 데이터 구조를 사용하는 모든 함수가 원소를 참조할 때 같은 이름을 사용하기 때문에 일관성을 높여줌

(1) 적당한 데이터 구조가 없으면 새로 만들기

(2) 테스트

(3) 함수 선언 바꾸기로 데이터 구조를 매개변수로 추가

(4) 테스트

(5) 함수 호출시 새로운 데이터 구조 인스턴스를 넘겨주도록 수정하고, 수정할 때마다 테스트

(6) 기존 매개변수를 사용하던 코드를 새 데이터 구조의 원소를 사용하도록 변경

(7) 기존 매개변수를 제거하고 테스트

아래는 온도측정값(**reading**) 배열에서 정상 작동 범위(**min**, **max**)를 벗어난 것이 있는지 검사하는 코드 예시입니다.

매개변수는 최대 **3개**를 넘어가지 않는 것이 좋은데 최대치군요? 그리고 **min**과 **max**는 연관성이 있어 묶어줄 수 있어 보입니다.

```swift
func readingsOutsideRange(station: Station, min: Int, max: Int) -> [Reading] {
    return station.readings.filter { $0.temp < min || $0.temp > max }
}

let station = Station(name: "ZB1", readings: [
    Reading(temp: 47, time: "2016-11-10 09:10"),
    Reading(temp: 53, time: "2016-11-10 09:20"),
    Reading(temp: 58, time: "2016-11-10 09:30"),
    Reading(temp: 53, time: "2016-11-10 09:40"),
    Reading(temp: 51, time: "2016-11-10 09:50")
])
```

```swift
// 호출
print(readingsOutsideRange(station: station, min: 51, max: 53))
// [__lldb_expr_7.Reading(temp: 47, time: "2016-11-10 09:10"), __lldb_expr_7.Reading(temp: 58, time: "2016-11-10 09:30")]
```

**min**과 **max**를 하나의 데이터 구조로 만들어줍니다.

그리고 해당 데이터 관련 처리도 모아주기 위해 함수로 만들어줍니다.

```swift
struct NumberRange<T: Comparable> {
    let min: T
    let max: T

    func contains(_ value: T) -> Bool {
        return value >= min && value <= max
    }
}
```

새로 만든 객체를 **readingOutsideRange()** 함수의 매개변수로 추가하도록 선언을 수정해주고,

```swift
func readingsOutsideRange(station: Station, min: Int, max: Int, range: NumberRange<Int>) -> [Reading] {
    return station.readings.filter { $0.temp < min || $0.temp > max }
}
```

호출부를 수정시켜줍니다. 아직은 구현부가 수정된 것이 아니라 테스트가 통과가 됩니다.

```swift
let operationPlan = NumberRange<Int>(min: 51, max: 53)

print(readingsOutsideRange(station: station, min: 51, max: 53, range: operationPlan))
```

기존 매개변수를 사용하는 곳을 하나씩(min, max, contains()) 변경하고, 변경할 때마다 테스트를 진행합니다.

```swift
func readingsOutsideRange(station: Station, min: Int, max: Int, range: NumberRange<Int>) -> [Reading] {
    return station.readings.filter { !range.contains($0.temp) }
}
```

선언부와 호출부의 기존 매개변수를 제거합니다.

```swift
func readingsOutsideRange(station: Station, range: NumberRange<Int>) -> [Reading] {
    return station.readings.filter { !range.contains($0.temp) }
}
```

```swift
let operationPlan = NumberRange<Int>(min: 51, max: 53)

print(readingsOutsideRange(station: station, range: operationPlan))
// [__lldb_expr_30.Reading(temp: 47, time: "2016-11-10 09:10"), __lldb_expr_30.Reading(temp: 58, time: "2016-11-10 09:30")]
```