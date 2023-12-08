---
title: (Refactoring) 6-3 함수 선언 바꾸기
author: Cody
date: 2023-05-02 00:34:00 +0800
categories: [Refactoring]
tags: [Swift, Refactoring]
pin: false
mermaid: true
---
함수의 이름이 좋으면 함수의 구현 코드를 볼 필요도 없이 호출문만 보고도 무슨일을 하는지 파악이 가능합니다.

> 💡 좋은 이름을 떠올리는 효과적인 방법은 '주석으로 함수의 목적을 설명해보기'. 주석이 멋진 이름으로 바뀌어 되돌아 올 때가 있음.

매개변수도 마찬가지. 함수를 사용하는 문맥을 설정하는 것.

함수 선언 바꾸기를 할 때는 변경사항을 보고 함수 선언과 호출문들을 단번에 고칠 수 있을지 가늠해보고 가능해 보이면 간단한 절차로 수행할 수 있습니다.

## **간단한 절차**

(1) 매개변수를 제거하려거든 먼저 함수 본문에서 제거 대상 매개변수를 대상 매개변수를 참조하는 곳은 없는지 확인

(2) 메서드 선언을 원하는 형태로 변경

(3) 기존 메서드 선언을 참조하는 부분을 모두 찾아서 바뀐 형태로 수정

(4) 테스트

만약 이름과 매개변수 모두를 수정하고 싶다면 각각 독립적으로 처리하는 것이 좋습니다.

독립적으로 하다가 문제가 있다면 롤백한 후 아래 절차로 수행합니다.

## **마이그레이션 절차**

(1) 이어지는 추출단계를 수월하게 만들어야 한다면 함수 본문을 적절히 리팩터링

(2) 함수 본문을 새로운 함수로 추출(새로 만들 함수 이름이 기존 함수와 같다면 일단 검색하기 쉬운 이름을 임시로 붙여두기

(3) 추출한 함수에 매개변수를 추가해야 한다면 [간단한 절차]를 따라 추가

(4) 테스트

(5) 기존 함수를 인라인하기

(6) 이름을 임시로 붙여뒀다면 함수 선언 바꾸기를 한번 더 적용하여 원래 이름으로 되돌리기

(7) 테스트

아래는 함수 이름 바꾸기의 예시입니다.

```swift
func circum(radius: Double) -> Double {
    return 2 * Double.pi * radius
}
```

함수 이름이 너무 축약이 되어 본래의 의미를 찾을 수 있도록 **[간단한 절차]**로 변경해줍니다.

(한국어로 치면 '원둘레'가 아니라 '원둘'로 표기한 셈)

(**Xcode**에서는 rename을 사용하면 사용된 곳까지 한번에 쉽게 바꿀 수 있죠. **Swift**를 리팩터링할 때 **Objective-C**에서 사용된 코드는 못 찾거나 찾아도 rename이 적용 안 되는 경우가 있으니 주의해야 합니다)

```swift
func circumference(radius: Double) -> Double { // circumference: 원의 둘레
        return 2 * Double.pi * radius
}
```

동일한 이름의 메서드가 여러 클래스에 정의되어 있는데 특정 클래스의 메서드만 변경하고 싶을 때는 **[마이그레이션 절차]**로 진행할 수 있습니다(..만 사실 **Xcode**는 이런 경우라도 잘 구분해서 renaming해주긴 합니다. 아래는 해당 기법을 알아두기 위한 정리입니다).

**마이그레이션 절차**로 진행하기 위해 대상 함수 본문 전체를 새로운 함수로 추출합니다.

```swift
func circum(radius: Double) -> Double {
    return circumference(radius: radius)
}

func circumference(radius: Double) -> Double {
    return 2 * Double.pi * radius
}
```

수정한 코드를 테스트한 뒤, 예전 함수를 인라인합니다. 그러면 예전 함수를 호출하는 부분이 모두 새 함수를 호출하도록 바뀌게 됩니다. 하나를 변경할 때마다 테스트하면서 하나씩 처리합니다. 모두 바꿨다면 예전 함수를 제거합니다.

만약 공개된 **API**를 수정해야 한다면? `circumference(radius:)` 함수를 만든 후 리팩터링을 멈춥니다. 가능하면 `circum(radius:)`가 **deprecated**임을 표시해줍니다. 그리고 클라이언트 모두가 `circumference(radius:)`를 사용한다고 판단이 될 때 `circum(radius:)`를 제거해줍니다.

## **매개변수 추가하기**

아래는 도서 관리 프로그램의 `Book`클래스의 예시입니다.

```swift
class Book {
    private var reservations: [Customer]

    init() {
        self.reservations = []
    }

    func addReservation(customer: Customer) { // 예약 기능
        reservations.append(customer)
    }

    func hasReservation(customer: Customer) -> Bool {
        return reservations.contains { $0.id == customer.id }
    }
}
```

위와 같은 상황에서 예약 기능에 우선순위 큐를 지원하라는 요구가 추가되는 상황입니다(이건 리팩터링이라기 보다 그냥 기능 추가?).

아래와 같이 새로운 매개변수의 기본값을 설정해두면, 기존 예약은 `customer`만 전달하여 그대로 사용하고 새롭게 추가되는 우선순위 예약의 경우 추가된 매개변수를 사용하면 됩니다.

```swift
func addReservation(customer: Customer, isPriority: Bool = false) {
    if isPriority {
        // 우선 순위 로직
    }
    else {
        reservations.append(customer) // 기존 로직
    }
}
```

(위처럼 **flag**값으로 동작을 구분하는 함수는 좋지 않지만 여기서는 넘어갑니다)

## **매개변수를 속성으로 바꾸기**

아래는 손님이 `NewEngland`에 거주하는지 여부를 반환해주는 함수인데, 이렇게 `Customer`를 받아서 처리하면, `Customer`에 국한되어서 처리를 하는 함수가 되어버리기 때문에, `Customer` 대신 `state`만 받아오도록 수정하는 것이 좋습니다.

```swift
func inNewEngland(_ aCustomer: Customer) -> Bool {
    let newEnglandStates = ["MA", "CT", "ME", "VT", "NH", "RI"]
    return newEnglandStates.contains(aCustomer.address.state)
}
```

호출되는 부분이 몇 없다면 찾아서 바로 수정해줘도 되지만, 복잡한 상황이라면 아래와 같이 바꾸어줍니다.

먼저 함수 추출하기로 새 함수를 만들어주고, 기존 함수에서 매개변수로 필요한 값(`stateCode`)을 추출하여 인라인하고 매개변수로 넣어줍니다.

```swift
func inNewEngland(_ aCustomer: Customer) -> Bool {
    let stateCode = aCustomer.address.state
    return xxNewEngland(stateCode)
}

func xxNewEngland(_ stateCode: String) -> Bool {
    return ["MA", "CT", "ME", "VT", "NH", "RI"].contains(stateCode)
}
```

그리고 호출문들을 찾아 하나씩 함수 인라인하기로 기존 함수 본문을 넣어줍니다.

위 작업이 다 되면 **[함수 선언 바꾸기]**로 새 함수의 이름(`xxNewEngland(_:)`)을 기존 함수의 이름(`inNewEngland(_:)`)으로 다시 변경하는 작업을 합니다.

```swift
func inNewEngland(_ stateCode: String) -> Bool {
    return ["MA", "CT", "ME", "VT", "NH", "RI"].contains(stateCode)
}
```

이렇게 수정되면 함수에서 필요로 하는 값만 받아옴으로써,

**외부 다른 객체의 의존도를 낮추고 재사용성을 높일 수 있게** 됩니다.