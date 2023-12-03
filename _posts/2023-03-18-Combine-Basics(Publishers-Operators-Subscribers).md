---
title: (Combine) Basics(Publishers, Operators, Subscribers)
author: Cody
date: 2023-03-18 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---
**Combine**의 핵심 기본요소는 **`Publishers`**, **`Operators`**, **`Subscribers`**.

다른 요소들도 있지만 위 세가지는 필수입니다.

## **Publishers**

하나 이상의 **`Subscriber`**에게 시간이 지남에 따라 값을 내보낼 수 있는 타입입니다.

수학 연산, 네트워킹, User event 처리를 포함한 거의 대부분의 Publisher의 내부 로직과 관계없이 3가지 타입의 이벤트를 내보낼 수 있습니다.

- **`Publisher`**의 **Generic**타입의 **`Output`**
- **`Success`** 완료
- **`Publisher`**의 **`Failure`**타입의 **`error`**가 있는 완료

**`Publisher`**는 0개 이상의 **`Output`**값을 내보낼 수 있고, 성공 혹은 실패로 완료된 경우 다른 이벤트를 내보낼 수 없습니다.

아래는 시간이 지남에 따라 **Int**값을 내보내는(emit) **`Publisher`**를 타임라인에서 시각화한 그림 -> 마블 다이어그램

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b86eb4bb-95ec-4cfc-b4bc-7b6f230ab4d8)

- **파란 박스**: 타임라인에서 지정된 시간에 방출된 값을 나타내고 숫자는 방출된 값을 나타냅니다.
- 다이어그램의 오른쪽에 **수직선**: 성공적인 **Stream**완료를 의미합니다.
- 

마블 다이어그램은 소프트웨어 내의 모든 종류의 동적 데이터를 나타낼 수 있습니다. 그래서 숫자 계산, 네트워크 호출, 사용자 제스처 반응 또는 화면상의 데이터 표시에 관계없이 **Combine** **Publisher**를 사용하여 앱 내의 모든 작업을 처리할 수 있습니다.

**Publisher Protocol**은 두가지 유형이 일반적입니다.

- **`Publisher.Output`**은 **Publisher**의 출력 값의 타입입니다. 
`Int publisher`라면 `String`이나 `Date`값을 내보낼 수 없습니다.
- **`Publisher.Failure`**는 **Publisher**가 실패할 경우 발생할 수 있는 **`Error`** 타입입니다.
**Publisher**가 절대 실패할 수 없는 경우 **`Never` failure**타입을 지정해줍니다.

특정 **`Publisher`**를 **`Subscribe`**하면 예상되는 값과 실패할 수 있는 **Error**를 알 수 있습니다.

## **Operators**

**Operators**는 **동일한 Publisher** 혹은 **새 Publisher**를 반환하는 **Publisher protocol**에서 선언된 메서드입니다.

Operators는 Subscription 중 복잡한 처리를 구현하기 위해 결합(combine)시켜서 사용할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/32015906-5b32-49b3-9d94-74d82835727a)

**Operators**는 항상 **업스트림(Upstream)**과 **다운스트림(Downstream)**이라고 하는 입력과 출력을 항상 가지고 있습니다.→ 이를 통해 ‘공유상태’를 피할 수 있습니다.

**Operators**는 이전 **Operator**로부터 받은 데이터로 작업하는 데 중점을 두고 체인의 다음 **Operator**에게 출력을 제공합니다.

즉 비동기적으로 실행되는 다른 어떤 코드도 작업중인 데이터에 끼어들어 변경할 수 없습니다.

## **Subscribers**

모든 **Subscription**은 **`Subscriber`**와 함께 끝납니다.

**`Subscriber`**는 보통 내보낸 출력 혹은 completion 이벤트로 ‘무언가’를 수행합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9cf76b0a-06f0-4e96-9dd7-05362613af45)

**Combine**은 데이터 스트림 작업을 간단하게 만드는 두개의 기본 제공 **`Subscriber`**를 제공합니다.

- **`sink`** subscriber를 사용하면 **`Output` 값 및 completion을 수신할 코드와 함께 closure를 제공**할 수 있습니다. 거기서 받은 이벤트로 원하는대로 할 수 있습니다.
- **`assign`** subscriber를 사용하면 custom 코드 필요없이 결과 출력을 **data model 또는 UI control의 property로 바인딩하여 `keypath`를 통해 데이터를 화면에 직접 표시**할 수 있습니다.

## **Subscription**

**`Publishers`, `Operators`, `Subscribers`의 전체 체인**을 설명하는 용어입니다.

**subscription**이 끝날 때 **subscriber**를 추가하면 체인 시작부분에서 **publisher**를 ‘활성화’합니다.

**잠재적으로 출력을 받을 subscriber가 없으면 publisher는 값을 내보내지 않습니다** → ⭐️기억해야 할 중요 사항!

**Subscription**은 **custom** 코드와 **Error**처리가 있는 비동기 이벤트 체인을 한 번만 선언할 수 있습니다.

코드가 Combine으로 이루어져 있을 경우, **Subscription**을 통해 앱의 전체 로직을 설명할 수 있고 작업이 완료되면 데이터를 push 혹은 pull하거나 이 개체 혹은 다른 개체를 콜백할 필요없이 시스템이 모든 것을 실행하도록 할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2d799709-1b8a-4cb4-9ed9-40141028bad2)

설계된 대로 **subscription**은 사용자의 제스처, 타이머 꺼짐 혹은 다른 무언가가 **publisher 중 하나를 깨울 때마다 비동기적으로 ‘실행(fire)’**됩니다.

## **Cancellable**

시스템에서 제공하는 두 **Subscriber(`sink`, `assign`)** 모두 **Cancellable** 프로토콜을 준수하고, 이는 **subscription**코드(전체 publisher, operators, subscriber 체인)가 **AnyCancellable** 객체를 반환합니다.

메모리에서 해당 객체를 해제할 때마다, 전체 **subscription**이 취소되고, 해당 리소스가 메모리에서 해제됩니다.