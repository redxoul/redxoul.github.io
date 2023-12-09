---
title: (Refactoring) 6-2 변수 추출하기, 변수 인라인하기
author: Cody
date: 2023-05-01 00:34:00 +0800
categories: [Refactoring]
tags: [Swift, Refactoring]
pin: false
mermaid: true
---
## 변수 추출하기 Extract Variable

표현식이 복잡하여 이해하기 어려울 땐, 지역 변수로 표현식을 쪼개어 관리하기 쉽게 만들수 있습니다.

추가한 변수는 디버거에 breakpoint를 지정하거나, 상태를 출력하거나, 상태를 임의로 변경할 수 있어 디버깅에도 많은 도움이 됩니다.

(1) 추출하려는 표현식에 부작용은 없는지 확인

(2) 상수를 하나 선언하고 이름을 붙일 표현식의 복제본을 대입

(3) 원본 표현식을 만든 변수로 교체

(4) 테스트

(5) 표현식을 여러 곳에서 사용한다면 각각을 새로 만든 변수로 교체. 하나 교체할 때마다 테스트.

아래에 딱 봐도 복잡한 주석이 꼭 필요해보이는 계산식의 예시가 있습니다.

```swift
func price(order: Order) -> Double {
    // 가격(price) = 기본가격 - 수량할인 + 배송비
    return (order.quantity.double * order.itemPrice) - (max(0, order.quantity - 500).double * order.itemPrice * 0.05) + (min(order.quantity.double * order.itemPrice * 0.1, 100))
}
```

위 코드는 아래와 같이 리팩터링이 가능합니다.

```swift
func price(order: Order) -> Double {
    let basePrice: Double = order.quantity.double * order.itemPrice
    let quantityDiscount: Double = max(0, order.quantity - 500).double * order.itemPrice * 0.05
    let shippingCost: Double = min(basePrice * 0.1, 100)

    return basePrice - quantityDiscount + shippingCost
}
```

아래는 클래스 내에서의 변수 추출하기 예제입니다.

```swift
class Order {
    private let record: Record

    init(record: Record) {
        self.record = record
    }

    var quantity: Int {
        return record.quantity
    }

    var itemPrice: Double {
        return record.itemPrice
    }

    var price: Double {
        return Double(quantity) * itemPrice -
            max(0, Double(quantity - 500)) * itemPrice * 0.05 +
            min(Double(quantity) * itemPrice * 0.1, 100)
    }
}
```

지역 변수로 추출하는 것 대신에 연산프로퍼티로 추출한다는 것만 빼면 동일합니다.

price 프로퍼티를 아래처럼 리팩터링할 수 있습니다.

```swift
class Order {
    private let record: Record

    init(record: Record) {
        self.record = record
    }

    var quantity: Int {
        return record.quantity
    }

    var itemPrice: Double {
        return record.itemPrice
    }

    var price: Double { // 아래 작성한 연산 프로퍼티로 리팩터링
        return basePrice - quantityDiscount + shippingCost
    }

    // 추가
    var basePrice: Double {
        return Double(quantity) * itemPrice
    }

    var quantityDiscount: Double {
        return max(0, Double(quantity - 500)) * itemPrice * 0.05
    }

    var shippingCost: Double {
        return min(Double(quantity) * itemPrice * 0.1, 100)
    }
}
```

## 변수 인라인하기 Inline Variable

이름이 원래의 표현식과 다를바가 없을 때는 불필요한 변수가 될 수 있고,

이런 변수는 주변 코드를 리팩터링하는데 방해가 될 수도 있으니 인라인시켜주는 것이 좋습니다.

(1) 대입문의 표현식에서 부작용이 생기지는 않는지 확인

(2) 변수가 상수로 선언되지 않았다면 상수로 만든 후 테스트 -> 이렇게 하면 값이 단 한 번만 대입되는지 확인 가능

(3) 테스트

(4) 변수를 사용하는 부분을 모두 교체할 때까지 과정을 반복

(5) 변수 선언문과 대입문을 제거

(6) 테스트

예시입니다.

```swift
func isDeliveryFree(order: Order) -> Bool {
    let basePrice = order.basePrice
    return basePrice > 1000.0
}
```

함수 인라인하기와 유사하게 아래처럼 리팩터링해주면 됩니다.

```swift
func isDeliveryFree(order: Order) -> Bool {
    return order.basePrice > 1000.0
}
```