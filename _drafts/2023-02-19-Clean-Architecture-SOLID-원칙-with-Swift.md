---
title: (Clean Architecture) SOLID 원칙 with Swift
author: Cody
date: 2023-02-19 00:34:00 +0800
categories: [iOS, Dev Terms]
tags: [Swift, Clean Architecture]
pin: false
mermaid: true
---
(이 글은 '클린 아키텍처 - 소프트웨어 구조와 설계 원칙'을 읽고 정리한 글입니다. 예시는 책의 것과 다를 수도 있습니다)

좋은 소프트웨어는 깔끔한 코드(Clean Code)에서 시작합니다.

좋은 벽돌을 사용하지 않으면 빌딩의 아키텍처가 좋고 나쁨은 크게 의미가 없고,

반대로 좋은 벽돌을 사용하더라도 빌딩의 아키텍처를 엉망으로 만들 수 있습니다.

그래서 좋은 벽돌로 좋은 아키텍처를 정의하는 원칙이 필요 -> **`SOLID`**

## **SOLID원칙의 목적**

중간 수준의 소프트웨어 구조가 변경에 유연하고, 이해하기 쉽고 많은 소프트웨어 시스템에 사용될 수 있는 컴포넌트의 기반이 되도록 하는 데 있습니다.

(여기서 중간 수준 소프트웨어는 코드 수준보다는 조금 상위이며 모듈과 컴포넌트 내부에서 사용되는 소프트웨어를 말합니다)

## **SOLID원칙**

아래 다섯 가지 원칙의 머리글자로 줄여서 **SOLID**원칙이라고 합니다.

- **SRP**(단일 책임 원칙, **Single Responsibility Principle**)
- **OCP**(개방-폐쇄 원칙, **Open-Closed Principle**)
- **LSP**(리스코프 치환 원칙, **Liskov Substitution Principle**)
- **ISP**(인터페이스 분리 원칙, **Interface Segregation Principle**)
- **DIP**(의존성 역전 원칙, **Dependancy Inversion Principle**)

각 원칙에 대한 설명은 아래에 하나씩 정리해 봅니다.

---

## **`SRP`**(단일 책임 원칙, **Single Responsibility Principle**)

: (콘웨이 Conway 법칙에 따른 따름 정리) 소프트웨어 시스템이 가질 수 있는 최적의 구조는 시스템을 만드는 조직의 사회적 구조에 커다란 영향을 받습니다.

따라서 **각 소프트웨어 모듈은 변경의 이유가 하나, 단 하나여야**만 합니다.

(이름만으로 '함수는 단 하나의 일만 해야 한다는 원칙'으로 오해할 수 있음. 이는 중요하지만 SRP는 아님)

**SRP**를 지키게 되면 **모듈의 재사용성과 응집도가 높아지며, 모듈 간 의존성을 감소**시킬 수 있습니다.

아래는 **SRP**가 지켜지지 않은 '급여 **App**의 **Employee** 클래스' 예시이며 아래와 같은 기능을 합니다.

```swift
class Employee {
    var name: String
    var id: String
    var salary: Double

    func calculatePay() -> Double {
            // 급여 계산 코드
            // 회계팀에서 기능을 정의하고, CFO에게 보고하기 위해 사용
    }

    func saveEmployee() {
            // 직원 정보 저장 코드
            // DB관리자가 기능을 정의하고, CTO에게 보고하기 위해 사용
    }

    func reportHours() {
            // 급여 출력 코드
            // 인사팀에서 기능을 정의하고 사용, COO에게 보고하기 위해 사용
    }
}
```

서로 다른 **Actor**들(회계팀, DB관리자, 인사팀)이 기능을 정의하는 메서드들이 단일 클래스에 모여,

한 **Actor**가 수정한 사항이 다른 **Actor**들에게도 영향이 갈 수 있는 상태입니다.

이를 **SRP** 원칙을 따라서 수정을 한다면 아래와 같이 할 수 있습니다.

**SRP**에 따라 각 클래스들이 하나의 책임만을 가지고, 각 클래스를 수정할 **Actor**들이 서로 영향을 받지 않게 되었습니다.

```swift
class Employee {
    var name: String
    var id: String
    var salary: Double

    func calculatePay() -> Double {
            // 급여 계산 코드
    }
}

class EmployeeDatabase {
    func saveEmployee(employee: Employee) {
            // 직원 정보 저장 코드
    }
}

class PayrollSystem {
    func reportHours(employee: Employee) {
            // 급여 출력 코드
    }
}
```

---

## **`OCP`**(개방-폐쇄 원칙, **Open-Closed Principle**)

: (버트란트 마이어 Bertland Meyer에 의해 유명해진 원칙)

**'소프트웨어 개체는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다'**는 원칙으로

기존 코드를 수정하기보다는 반드시 새로운 코드를 추가하는 방식으로 시스템의 행위를 변경할 수 있도록 설계해야만 소프트웨어 시스템을 쉽게 변경할 수 있다는 것이 이 원칙의 요지입니다.

**OCP**을 지키면 **기존의 코드를 변경하지 않고 새로운 동작을 추가**할 수 있게 됩니다.

아래는 **OCP**가 지켜지지 않은 '결제 프로세스' 예시입니다.

```swift
class Payment {
    var amount: Double
    var method: String

    init(amount: Double, method: String) {
        self.amount = amount
        self.method = method
    }

    func process() -> Bool {
        if method == "credit" {
                    // 신용카드 결제 프로세스
        } else if method == "paypal" {
                    // 페이팔 결제 프로세스
        } else {
            return false
        }

        return true
    }
}
```

위 코드도 얼마든지 새로운 결제 프로세스를 추가할 수는 있지만,

기능이 확장될 때마다 **Payment** 클래스에 변경이 일어나 **OCP**에 위반이 됩니다.

우리는 **Swift**의 **Protocol**을 통해 이 원칙을 지키기 쉽게 작성할 수 있습니다.

위 코드를 **OCP**에 맞춰 수정을 하면 아래와 같이 됩니다.

```swift
protocol PaymentMethod {
    func process(amount: Double) -> Bool
}

class CreditCardPayment: PaymentMethod {
    func process(amount: Double) -> Bool {
                // 신용카드 결제 프로세스
        return true
    }
}

class PayPalPayment: PaymentMethod {
    func process(amount: Double) -> Bool {
                // 페이팔 결제 프로세스
        return true
    }
}

class Payment {
    var amount: Double
    var method: PaymentMethod

    init(amount: Double, method: PaymentMethod) {
        self.amount = amount
        self.method = method
    }

    func process() -> Bool {
        return method.process(amount: amount)
    }
}
```

새로운 결제 프로세스가 생겼을 때는 **PaymentMethod** 프로토콜을 따르는 새 결제 프로세스를 작성하면 되고,

**Payment** 클래스에 주입시켜 주면 됩니다.

이제 **Payment**클래스는 확장에는 열려있고, 변경에는 닫혀있게 되었습니다.

---

## **`LSP`**(리스코프 치환 원칙, **Liskov Substitution Principle**)

: (바바라 리스코프 Babara Liskov가 정의한 하위 타입에 관한 원칙) 상호 대체 가능한 구성요소를 이용해 소프트웨어 시스템을 만들 수 있으려면, 이들 구성요소는 반드시 서로 치환 가능해야 한다는 계약을 반드시 지켜야 합니다.

쉽게 말해 **서브타입은 기본타입을 대체할 수 있어야 할 수 있어야 한다**는 원칙입니다.

**'서브타입을 기본타입 대신 사용하더라도 이들을 사용하던 클라이언트의 기존 동작은 동일한 동작을 해야'**합니다.

**LSP**을 지키면 **상속 구조의 유지보수성이 높아지며, 다형성을 제공**할 수 있습니다.

아래는 **LSP**를 위반하는 예시입니다.

```swift
class Rectangle { // 사각형 클래스
    var width: Double
    var height: Double

    init(width: Double, height: Double) {
        self.width = width
        self.height = height
    }

    func area() -> Double {
        return width * height
    }
}

class Square: Rectangle { // 정사각형 클래스
    override var width: Double {
        didSet {
            height = width
        }
    }

    override var height: Double {
        didSet {
            width = height
        }
    }
}
```

**Square** 클래스는 **Rectangle** 클래스를 상속받고 있습니다.

하지만 **Square** 클래스는 **width**와 **height**가 항상 같아야 하기 때문에, **width**와 **height**를 따로 설정하는 것이 불가능합니다.

따라서 **Square** 클래스의 인스턴스는 **Rectangle**클래스의 인스턴스처럼 동작하지 않습니다.

이는 **LSP**를 위반하는 것입니다.

이를 해결하기 위해, **LSP**를 지키는 방법으로 **Rectangle** 클래스와 **Square** 클래스를 다음과 같이 수정할 수 있습니다.

```swift
class Rectangle { *// 사각형 클래스*
        var width: Double
    var height: Double

    init(width: Double, height: Double) {
        self.width = width
        self.height = height
    }

    func area() -> Double {
        return width * height
    }
}

class Square: Rectangle { *// 정사각형 클래스*
        override var width: Double {
        didSet {
            height = width
        }
    }

    override var height: Double {
        didSet {
            width = height
        }
    }

    init(side: Double) {
        super.init(width: side, height: side)
    }
}
```

위와 같이 정사각형 변의 길이(**side**)만 받아서 초기화하도록 해주면,

기본 **Rectangle**클래스와 동일한 동작을 하도록 유지하면서 **Square**클래스 인스턴스를 사용할 수 있게 되어 **LSP**를 준수할 수 있게 됩니다.

---

## **`ISP`**(인터페이스 분리 원칙, **Interface Segregation Principle**)

: 소프트웨어 설계자는 사용하지 않은 것에 의존하지 않아야 한다.

**Swift**로 설명하면

**프로토콜(인터페이스)은 클라이언트가 사용하지 않는 것을 구현하도록 하면 안 된다**는 의미입니다.

**ISP**을 지키면 **인터페이스의 의도가 분명해지고, 유연성과 재사용성이 높아**집니다.

아래는 **ISP**를 위반한 예시입니다.

저 프린터는 **scan**기능도 **fax**기능도 없지만 잘못 선언된 **Machine** 프로토콜의 인터페이스를 따라야 하기 때문에 엉뚱한 메서드들을 구현해야만 하며, 나중에 이러한 사정을 모르는 신입 개발자는 이를 사용하여 또 다른 엉뚱한 코드를 만들어낼 수도 있습니다.

```swift
protocol Machine {
    func print()
    func scan()
    func fax()
}
```

```swift
class SomePrinter: Machine {
    func print() {
            // 프린트 동작 구현
    }

    func scan() {
            // 스캔 기능 없지만 구현은 해놓음
    }

    func fax() {
            // 팩스 기능 없지만 구현은 해놓음
    }
}
```

위 코드를 **ISP**에 맞춰 작성하면 아래와 같이 할 수 있습니다.

```swift
protocol Printer {
    func print()
}

protocol Scanner {
    func scan()
}

protocol Faxer {
    func fax()
}
```

```swift
class HPPrinter: Printer {
    func print() {
            // 프린트 기능 구현
    }
}

class BrotherScanner: Scanner {
    func scan() {
            // 스캔 기능 구현
    }
}

class CannonAllInOne: Printer, Scanner {
    func print() {
            // 프린트 기능 구현
    }

    func scan() {
            // 스캔 기능 구현
    }
}
```

---

## **`DIP`**(의존성 역전 원칙, **Dependancy Inversion Principle**)

: 고수준 정책을 구현하는 코드는 저수준 세부사항을 구현하는 코드에 절대로 의존해서는 안 됩니다. 대신 세부사항이 정책에 의존해야 합니다.

**Swift**로 설명을 하자면 **'구체적인 것(Class, Struct)에 의존하면 안 되고, 구체적인 것이 추상화된 것(Protocol)에 의존해야 한다.'**는 의미입니다.

**DIP**를 지키면 **결합도를 낮출 수 있으며, 유연성과 확장성**이 높아집니다. **테스트 코드 작성에도 유리**해집니다.

([의존성, 의존성 주입, 의존성 역전의 원칙](https://www.notion.so/A-dependency-Dependency-Injection-Dependency-Inversion-Principle-0049752fcf81427385a713c534e5af6a?pvs=21)에 대해 더 자세히 써놓은 글이 있으니 이를 참조하셔도 좋습니다)

아래는 **DIP**를 위반한 예제입니다.(위 글에서 사용한 동일한 예제)

**Moving** 클래스에 의존성을 가지는 **Human클래스입니다.**

```swift
class Moving {
    func walking() {
        print("🚶🏻👣👣👣👣")
    }

    func running() {
        print("🏃🏻‍♂️💨💨💨💨")
    }
}

class Human {
    let moving: Moving

    init(move: Moving) {
        self.moving = move
    }

    func walking() {
        moving.walking()
    }

    func running() {
        moving.running()
    }
}
```

이와 같은 상황에서 **Moving**을 **Human**이 아닌 다른 동물에도 적용을 하고 싶다면, 혹은 다른 방법으로 걷거나 달리는 **Human**을 원하게 되면 어떻게 할까요?

**Moving**에 다른 메서드를 추가하거나? 다른 **Moving**을 만들어야 할까요? 그러고 나면 **Human**도 그에 맞춰 수정해야겠죠?

위와 같은 의존관계에서는 유연하게 변화를 주기가 어렵습니다. 변화에 취약합니다.

위 코드를 **DIP**에 맞춰 수정해 보면 아래와 같습니다.

```swift
// Moving을 protocol로 추상화
protocol Moving {
    func walking()
    func running()
}

// 추상화된 Moving으로 다양한 Moving을 작성
class HumanMoving: Moving {
    func walking() {
        print("🚶🏻👣👣👣👣")
    }

    func running() {
        print("🏃🏻‍♂️💨💨💨💨")
    }
}

class HumanMoving2: Moving {
    func walking() {
        print("🧑🏻‍🦯👣👣👣👣")
    }

    func running() {
        print("🧑🏻‍🦽☽☽☽☽")
    }
}

class Human {
    var moving: Moving

    init(move: Moving) {
        self.moving = move
    }

    func walking() {
        moving.walking()
    }

    func running() {
        moving.running()
    }
}
```

위와 같이 작성이 되면 **Human**은 다양한 방법으로 **Moving**을 할 수 있게 되었고,

앱이 동작하는 중에도 얼마든지 새로운 **Moving**을 주입해서 다른 동작을 할 수 있도록 유연해지게 되었습니다.

```swift
let human = Human(move: HumanMoving())
human.walking() // 🚶🏻👣👣👣👣
human.running() // 🏃🏻‍♂️💨💨💨💨

human.moving = HumanMoving2()
human.walking() // 🧑🏻‍🦯👣👣👣👣
human.running() // 🧑🏻‍🦽☽☽☽☽
```

저 **Moving**은 다른 동물에도 사용할 수 있습니다.

```swift
class CatMoving: Moving {
    func walking() {
        print("🐈🐾🐾🐾🐾")
    }

    func running() {
        print("🐈💨💨💨💨")
    }
}

class Cat {
    let moving: Moving

    init(move: Moving) {
        self.moving = move
    }

    func walking() {
        moving.walking()
    }

    func running() {
        moving.running()
    }
}

let cat = Cat(move: CatMoving())
cat.walking()// 🐈🐾🐾🐾🐾
cat.running()// 🐈💨💨💨💨
```

---

## 정리

**SOLID** 원칙을 지키지 않으면

1. 코드의 복잡성과 결합도가 높아져 유지보수성이 떨어지며,
2. 새로운 기능을 추가하기 어려워집니다.
3. 또한, 코드 변경 시 예기치 않은 부작용이 발생할 가능성이 높아집니다.

**Swift** 2.0이 발표됐을 때 프로토콜 지향 언어라고 발표가 되기도 했습니다.

위 예시들에서처럼

**Swift**에서는 **Protocol**(과 **Extension**)을 잘 사용하는 것만으로도

**SOLID**원칙을 지키며 소스를 작성하기가 쉬운 까닭이지 않을까 싶습니다.