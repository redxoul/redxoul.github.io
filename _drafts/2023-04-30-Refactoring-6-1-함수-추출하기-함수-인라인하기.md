---
title: (Refactoring) 6-1 함수 추출하기, 함수 인라인하기
author: Cody
date: 2023-04-30 00:34:00 +0800
categories: [Refactoring]
tags: [Swift, Refactoring]
pin: false
mermaid: true
---
## 함수 추출하기 Extract Function

`코드 조각을 찾아 무슨 일을 하는지 파악한 다음, 독립된 함수로 추출하고 목적에 맞는 이름을 붙이는 작업`입니다.

`'목적과 구현을 분리'`하는 방식이 가장 합리적인 기준. 코드를 보고 무슨 일을 하는지 파악하는 데 한참이 걸린다면 그 부분을 함수로 추출한 뒤 '무슨 일'에 걸 맞는 이름을 짓습니다.

(1) 함수를 새로 만들고, 목적을 잘 드러내는 이름을 붙임('어떻게'가 아닌 '무엇을' 하는지가 드러나야 함): 대상 코드가 함수 호출문 하나처럼 간단하더라도 함수로 뽑아서 목적이 더 잘 드러나는 이름을 붙일수 있다면 추출할 것(이런 이름이 떠오르지 않는다면 추출하면 안 된다는 신호)

(2) 추출한 코드를 원본 함수에서 복사하여 새 함수에 붙여넣음

(3) 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는지 검사. 있다면 매개변수로 전달.

(4) 변수를 다 처리했다면 컴파일

(5) 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 바꿈

(6) 테스트

(7) 다른 코드에 방금 추출한 것과 똑같거나 비슷한 코드가 없는지 살핌. 있다면 방금 추출한 새 함수를 호출하도록 바꿀지 검토.

아래는 예시 코드입니다.

```swift
func printOwing(invoice: inout Invoice) {
    var outstanding = 0.0

    print("```````````*")
    print("`` Customer Owes ``")
    print("```````````*")

    // calculate outstanding
    for order in invoice.orders {
        outstanding += order.amount
    }

    // record due date
    let today = Date()
    invoice.dueDate = Calendar.current.date(byAdding: .day, value: 30, to: today)

    //print details
    print("name: \(invoice.customer)")
    print("amount: \(outstanding)")

    if let dueDate = invoice.dueDate {
        print("due: \(dueDate)")
    }
}

var invoice = Invoice(orders: [Order(amount: 2.0), Order(amount: 5.0)], customer: "엘리")
printOwing(invoice: &invoice)
```

위 코드의 문제점: `printOwing` 함수의 길이가 매우 길고, 하는 일이 많아서 파악이 어려움. 그리고 코드 리딩을 돕기 위한 주석이 달려 있음 → 한 단계씩 함수를 추출해 나가야 합니다.

(inout 매개변수의 문제점은 여기서 짚지는 않습니다)

우선 함수 제일 위의 지역변수 정의: 예전에는 각 함수에서 사용할 지역변수를 가장 상단에 정의하는 것이 관행이었지만 요즘은 그렇지 않습니다. `사용하는 곳과 최대한 가까운 곳에 정의`해주는 것이 좋습니다.

각 단계마다 주석으로 정리해줍니다.

```swift
 func printOwing(invoice: inout Invoice) {
    // 배너를 출력
    print("```````````*")
    print("`` Customer Owes ``")
    print("```````````*")

    // outstanding(미해결 채무)을 계산
    var outstanding = 0.0 // 사용하는 곳 가까운 곳으로 이동
    for order in invoice.orders {
        outstanding += order.amount
    }

    // 지급 날짜를 계산, 기존 객체에 dueDate를 수정 -> 이것 또한 나쁜 냄새. 불변의 데이터를 유지하는 것이 좋음(여기서는 일단 패스)
    let today = Date()
    invoice.dueDate = Calendar.current.date(byAdding: .day, value: 30, to: today)

    // 세부사항을 출력
    print("name: \(invoice.customer)")
    print("amount: \(outstanding)")

    if let dueDate = invoice.dueDate {
        print("due: \(dueDate)")
    }
}

var invoice = Invoice(orders: [Order(amount: 2.0), Order(amount: 5.0)], customer: "엘리")
printOwing(invoice: &invoice)
```

위 단계를 통해 아래와 같이 4개의 함수로 정리가 가능해 보입니다.

```swift
func printOwing(invoice: inout Invoice) {
    printBanner()

    var outstanding = calculateOutstanding(invoice) // 지역변수를 사용하는 경우 매개변수로 넘김

    updateDueDate(invoice: &invoice)

    printDetails(invoice: invoice, outstanding: outstanding) // 지역변수를 사용하는 경우 매개변수로 넘김
}

func printBanner() {
    print("***********************")
    print("**** Customer Owes ****")
    print("***********************")
}

func calculateOutstanding(_ invoice: Invoice) -> Double {
    var result = 0.0 // 함수 내에서 반환을 목적으로 하는 변수의 경우 result를 사용

    for order in invoice.orders {
        result += order.amount
    }

    return result
}

func updateDueDate(invoice: inout Invoice) {
    let today = Date()
    invoice.dueDate = Calendar.current.date(byAdding: .day, value: 30, to: today)
}

func printDetails(invoice: Invoice, outstanding: Double) {
    print("name: \(invoice.customer)")
    print("amount: \(outstanding)")

    if let dueDate = invoice.dueDate {
        print("due: \(dueDate)")
    }
}
```

위와 같이 정리를 하고 나면

`printOwing` 함수는 4줄의 함수로 바뀌고 각 줄이 의미와 목적이 있는 이름으로 대체되어

세부내용을 확인하지 않고도 바로 함수를 이해할 수 있게 되었습니다.

만약 후에 버그가 생겼다면 빠르게 해당 함수를 읽고 수정하기도 좋습니다.

추가로 `calculateOutstanding` 메서드의 경우, for문을 도는 절차지향 코드를 함수형 프로그래밍으로 전환.

```swift
func calculateOutstanding(_ invoice: Invoice) -> Double {
    return invoice.orders.reduce(0.0) { sum, order in
        sum + order.amount
    }
}
```

## 함수 인라인하기 Inline Function

위에서 함수 추출을 통해 목적이 드러나는 이름의 짧은 함수를 사용하기를 권하지만,

때로는 함수 본문이 이름만큼 명확한 경우가 있어 이럴 경우 그 함수를 다시 inline하는 과정이 필요합니다.

(1) 다형 메서드인지 확인: 서브 클래스에서 오버라이드하는 메서드는 인라인하면 안 됨

(2) 인라인할 함수를 호출하는 곳을 모두 찾기

(3) 각 호출문을 함수 본문으로 교체

(4) 하나씩 교체할 때마다 테스트: 한번에 할 필요는 없음. 인라인하기 까다로운 부분이 있다면 남겨두고 틈틈이 처리

(5) 함수 정의를 삭제

아래는 매우 간단한 예시1

```swift
func rating(driver: Driver) -> Int {
    return moreThanFiveLateDeliveries(driver: driver) ? 2 : 1
}

func moreThanFiveLateDeliveries(driver: Driver) -> Bool {
    return driver.numberOfLateDeliveries > 5 // 함수 이름이 그대로 구현된 짧은 코드
}
```

위와 같은 경우

함수 이름과 동일한 기능이며 재사용성이 없어보이는 코드가 짧게 구현되어 있습니다.

만약 단순히 늦은 배달이 5번 초과되는지를 체크하는 함수가 아닌, 5번의 기준만이 아닌 다른 비즈니스 로직에 따라 배달원을 평가할 수 있거나 재사용성이 있는 함수의 내용이라면 inline을 할 필요는 없지만 위와 같은 케이스라면 inline이 필요합니다.

```swift
func rating(driver: Driver) -> Int {
    return driver.numberOfLateDeliveries > 5 ? 2 : 1
}
```

함수 인라인하기 두번째 예시입니다.

```swift
func reportLines(customer: Customer) -> [[String]] {
    var lines = [[String]]()
    gatherCustomerData(out: &lines, customer: customer) // 불필요한 간접호출
    return lines
}

func gatherCustomerData(out: inout [[String]], customer: Customer) {
    out.append(["name", customer.name])
    out.append(["location", customer.location])
}
```

`reportLines` 함수내에서 불필요한 간접호출이 있는 경우입니다.

아래와 같이 인라인을 해줍니다.

```swift
func reportLines(customer: Customer) -> [[String]] {
    var result = [[String]]() // 함수의 결과를 반환하는 목적의 변수
    result.append(["name", customer.name]) // 인라인 해준 후, 변수명 체크
    result.append(["location", customer.location])
    return result
}
```