---
title: (Refactoring) 6-6 여러 함수를 클래스로 묶기
author: Cody
date: 2023-05-14 00:34:00 +0800
categories: [iOS, Refactoring]
tags: [Swift, Refactoring]
pin: false
mermaid: true
---
클래스로 묶으면

- 함수들이 공유하는 **공통 환경을 더 명확하게 표현**할 수 있고,
- 각 함수에 전달되는 인수를 줄여서 **객체 안에서의 함수 호출을 간결**하게 만들 수 있습니다.
- 그리고 이런 객체를 **시스템의 다른 부분에 전달하기 위한 참조를 제공**할 수 있습니다.
- **클라이언트가 객체의 핵심 데이터를 변경할 수 있고, 파생 객체들을 일관되게 관리**할 수도 있습니다.

(1) 함수들이 공유하는 공통 데이터 레코드를 캡슐화: 공통 데이터가 레코드 구조로 묶여 있지 않다면 '[매개변수 객체 만들기](https://www.notion.so/Refactoring-6-5-a86122a05d824f76b3cbdb9104a8f79b?pvs=21)'로 데이터를 하나로 묶는 레코드를 만들기

(2) 공통 레코드를 사용하는 함수 각각을 새 클래스로 옮기기: 공통 레코드의 멤버는 함수 호출문의 인수 목록에서 제거

(3) 데이터를 조작하는 로직들을 '[함수로 추출](https://www.notion.so/Refactoring-6-1-3fbba6f486284146b426b6f19534b8fb?pvs=21)'해서 새 클래스로 옮기기

아래는 '클래스로 묶기'가 필요한 예시코드입니다. 계량기를 읽어 세금을 부과하는 예시입니다.

(...라고 책에 되어 있지만. Swift로 예시를 전환하면서 Reading의 성격이 class보다는 struct가 맞다고 생각하여 struct로 작성했습니다)

```swift
let reading = Reading(customer: "ivan", quantity: 10, month: 5, year: 2017) // struct

func acquireReading() -> Reading {
    return reading
}

func baseRate(month: Int, year: Int) -> Double {
    if year == 2017 && month == 5 {
        return 0.1
    }

    return 0.2
}
```

위 코드는 클라이언트에서 아래처럼 사용하고 있습니다.

데이터와 데이터를 사용하는 곳이 여기저기 흩어져 있는 상황입니다.

```swift
// client1. 기본요금 계산
let aReading = acquireReading()
let baseCharge = baseRate(month: aReading.month, year: aReading.year) * aReading.quantity.double
print(baseCharge)
```

```swift
// client2. 세금을 부과. 기본 소비량만큼은 면세.
let aReading = acquireReading()
let base = baseRate(month: aReading.month, year: aReading.year) * Double(aReading.quantity)

func taxThreshold(year: Int) -> Double {
    return year > 2000 ? 0.1 : 0.2
}

let taxableCharge = max(0, base - taxThreshold(year: aReading.year))
```

```swift
// client3. 기본요금 계산 함수를 만들어둔 클라이언트
let aReading = acquireReading()

func calculateBaseCharge(_ aReading: Reading) -> Double {
    return baseRate(month: aReading.month, year: aReading.year) * Double(aReading.quantity)
}

let basicChargeAmount = calculateBaseCharge(aReading)
```

하나의 책임을 가진 클래스(or 구조체)가 필요해보입니다.

우선 데이터를 가진 **`Reading`** 구조체로 만들어줍니다.

그리고 이미 만들어진 **`baseRate(month:year:)`**와 **`calculateBaseCharge(_:)`**를 옮겨줍니다.

연산 프로퍼티로 옮겨준 후, 그에 맞게 이름도 변경시켜줍니다.

```swift
struct Reading {
    let customer: String
    let quantity: Int
    let month: Int
    let year: Int

    var baseRate: Double { // 연산 프로퍼티로 이동
		if year == 2017 && month == 5 {
            return 0.1
        }

        return 0.2
    }

    var baseCharge: Double { // 연산 프로퍼티로 이동하고, 이름을 변경
				return baseRate(month: month, year: year) * Double(quantity)
    }
}
```

각 클라이언트들에서 제각각 `baseCharge`를 계산하던 부분을 **`Reading`**클래스의 **`baseCharge`** 호출로 변경시켜줍니다.

```swift
// client1. 기본요금 계산
let aReading = acquireReading()
let baseCharge = aReading.baseCharge // 변경된 baseCharge 호출
print(baseCharge)
```

```swift
// client2. 세금을 부과. 기본 소비량만큼은 면세.
let aReading = acquireReading()
let base = aReading.baseCharge // 변경된 baseCharge 호출
func taxThreshold(year: Int) -> Double {
    return year > 2000 ? 0.1 : 0.2
}

let taxableCharge = max(0, base - taxThreshold(year: aReading.year))
```

```swift
// client3. 기본요금 계산 함수를 만들어둔 클라이언트
let aReading = acquireReading()

let basicChargeAmount = aReading.baseCharge // 변경된 baseCharge 호출
```

이어서 client2에서 **`taxThreshold`**에 따라 **`taxableCharge`**를 계산하는 부분도 옮겨옵니다.

```swift
struct Reading {
    let customer: String
    let quantity: Int
    let month: Int
    let year: Int

    var baseRate: Double {
        if year == 2017 && month == 5 {
            return 0.1
        }

        return 0.2
    }

    var baseCharge: Double {
        return baseRate * Double(quantity)
    }

    var taxThreshold: Double { // 연산 프로퍼티로 이동
				return year > 2000 ? 0.1 : 0.2
    }

    var taxableCharge: Double { // 연산 프로퍼티로 이동
				return max(0, baseCharge - taxThreshold)
    }
}
```

client2를 **`Reading`**의 **`taxableCharge`**를 사용하도록 수정합니다.

`base`값도 **`taxableCharge`**를 계산하기 위한 값이었기 때문에 제거해주어 간결하게 만들어줄 수 있습니다.

```swift
// client2. 세금을 부과. 기본 소비량만큼은 면세.
let aReading = acquireReading()
let taxableCharge = aReading.taxableCharge // 이동시킨 연산 프로퍼티 호출
```

관련데이터와 계산들을 한 곳으로 모으면, 함수 호출시 불필요한 매개변수가 필요없어져 함수 혹은 연산 프로퍼티로 만들 수 있습니다.

위에서는 `Reading`의 프로퍼티들을 `let`으로 작성했지만, 상황에 따라 `var`로 바꾸더라도 파생데이터들(`baseCharge`, `taxableCharge`)이 프로퍼티 변경에 따라 바로 계산되도록 연산 프로퍼티로 변경하여 호출부를 간결하게 만들고, 이들을 일관되게 관리할 수 있게 되었습니다.