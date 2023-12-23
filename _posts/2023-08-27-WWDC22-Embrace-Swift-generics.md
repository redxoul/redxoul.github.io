---
title: (WWDC22) Embrace Swift generics
author: Cody
date: 2023-08-27 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, Generics, WWDC22, WWDC]
pin: false
mermaid: true
---
[https://developer.apple.com/videos/play/wwdc2022/110352/](https://developer.apple.com/videos/play/wwdc2022/110352/)

`Generic` 모델링에 `some`/`any`를 적용하는 방법을 설명한 WWDC22 영상입니다.

`Farm`을 시뮬레이션 하기 위한 예시.

추상화 도구들을 사용하기 전의 `Concrete 타입`의 예시부터 작성.

## Concrete 타입으로 먼저 모델링

`Cow` 구조체는 `Hay`(건초) 타입의 매개변수를 받는 `eat()`이라는 메서드가 존재하고,  
`Hay` 구조체는 `Alfalfa` 종류의 작물을 재배하기 위한 `grow()`라는 static 메서드가 존재.  
`Alfalfa` 구조체는 `Alfalfa`인스턴스에서 `Hay`를 수확할 수 있는 `harvest()` 메서드가 존재.  
`Farm` 구조체는 `Cow`에게 먹이를 줄 수 있는 `feed()` 메서드가 있음.

```swift
struct Cow {
    func eat(_ food: Hay) { ... }
}

struct Hay {
    static func grow() -> Alfalfa { ... }
}

struct Alfalfa {
    func harvest() -> Hay { ... }
}

struct Farm {
    func feed(_ animal: Cow) { ... }
}
```

사료를 공급하기 위해서는 `alfalfa`를 먼저 `grow`하고, `hay`를 `harvest`한 후, `hay`를 `cow`에게 먹이는 방식으로 구현.

```swift
struct Farm {
    func feed(_ animal: Cow) {
        let alfalfa = Hay.grow()
        let hay = alfalfa.harvest()
        animal.eat(hay)
    }
}
```

하지만 더 많은 종류의 동물을 추가하고 싶다면?

아래처럼 동물들을 표현할 구조체를 추가할 수 있고, `Farm`에서는 각 동물들에게 먹이를 먹이기 위한 `feed()`메서드들을 오버로드로 추가해주어야 함.

각 `feed()` 메서드들은 아래처럼 매우 유사한 동작을 반복해서 구현하게 될 것.  
반복적인 구현으로 오버로드를 작성하는 경우 Generalize(일반화)해야한다는 신호.

```swift
struct Cow {
    func eat(_ food: Hay) { ... }
}

struct Horse { // 추가
    func eat(_ food: Carrot) { ... }
}

struct Chicken { // 추가
    func eat(_ food: Grain) { ... }
}

struct Farm {
    func feed(_ animal: Cow) {
        let alfalfa = Hay.grow()
        let hay = alfalfa.harvest()
        animal.eat(hay)
    }

    func feed(_ animal: Horse) { // 추가
        let root = Carrot.grow()
        let carrot = root.harvest()
        animal.eat(carrot)
    }

    func feed(_ animal: Chicken) { // 추가
        let wheat = Grain.grow()
        let grain = wheat.harvest()
        animal.eat(grain)
    }
}
```

## 공통 특성을 식별

위에서 특정 유형의 음식을 먹는 특성을 가진 동물 타입을 만듦.  
각 동물 타입은 먹는 방식이 다르기 때문에 `eat()` 메서드를 구현할 때마다 동작이 차이남.

→ 추상 코드가 `eat()` 메서드를 호출하고 추상코드가 작동중인 `Concrete 타입`에 따라 다르게 동작하도록 만드는 것이 목표.

### Polimorphism(다형성)

다양한 `Concrete 타입`에 대해 다르게 동작하는 특성을 `Polimorphism(다형성)`이라고 부름.

다형성을 이용하면 코드 사용방법에 따라 하나의 코드 조각이 여러 동작을 가질 수 있게 됨.

다형성은 여러가지 형태로 만들 수 있음.

1. `함수 오버로딩`: 동일한 함수 호출이 인수 유형에 따라 다른 의미를 가질 수 있도록 함. 일반적인 솔루션은 아니기 때문에 `adhoc-Polimorphism(임시 다형성)`이라고 부름.
2. `SubType 다형성`: 상위 타입에서 동작하는 코드는 코드가 런타임에 사용하는 특정 하위 타입에 따라 다른 동작을 가질 수 있음.
3. `Parametric 다형성`: `Generic`을 사용하여 구현하는 다형성. `Generic` 코드는 `타입 파라미터`를 사용해서 다양한 타입에서 동작하는 하나의 코드를 작성할 수 있도록 하며, `Concrete 타입` 자체는 인수로 사용됨.

### SubType 다형성

`SubType 다형성`의 방법은 여러가지 존재.

1. 클래스 계층 이용

`Animal`이라는 부모클래스를 만들고 각 동물을 클래스로 변경시키고 `Animal`을 상속해서 `eat()` 메서드를 재정의.

```swift
class Animal {
    func eat(_ food: ???) { fatalError("Subclass must implement 'eat'") }
}

class Cow: Animal {
    override func eat(_ food: Hay) { ... }
}

class Horse: Animal {
    override func eat(_ food: Carrot) { ... }
}

class Chicken: Animal {
    override func eat(_ food: Grain) { ... }
}
```

위 같은 경우의 문제점

- 클래스를 사용하면 서로 다른 동물 인스턴스 간에 어떤 상태도 공유할 필요가 없거나 원하지 않음에도 불구하고 참조 Semantics를 강제로 갖게 됨.
- 하위 클래스가 기본 클래스의 메서드를 재정의해야 하지만 이를 잊게 되면 런타임까지 알 수 없게 됨.
- 각 `Animal` 하위 타입들이 다른 타입의 음식을 먹으며 이런 dependency를 표현하기가 어려워짐.  
    이런 경우 `Hay`, `Carrot`, `Grain`을 `Any`같은 덜 `Concrete 타입`으로 바꾸는 해결방법이 있지만,
    - 올바른 타입이 전달되었는지 확인하는 코드가 하위 클래스에 포함되어야 하는 문제가 생김.
    - 타입을 체크하는 로직을 하위 클래스에 넣었다고 해도, 잘못된 타입의 음식을 전달해서 발생하는 문제를 런타임에만 발견할 수 있어 버그가 발생할 수 있음.

## Generic의 적용

`Animal`에 `타입 파라미터`를 적용해서 type-safe한 방식으로 동물들의 사료 타입을 표현이 가능.  
타입 파라미터는 각 하위 클래스의 특정 사료 타입에 대한 placeholder 역할을 해줌.

```swift
class Animal<Food> {
    func eat(_ food: Food) { fatalError("Subclass must implement 'eat'") }
}

class Cow: Animal<Hay> {
    override func eat(_ food: Hay) { ... }
}

class Horse: Animal<Carrot> {
    override func eat(_ food: Carrot) { ... }
}

class Chicken: Animal<Grain> {
    override func eat(_ food: Grain) { ... }
}
```

위 방식으로 작성하게 되면 `Animal`이 동작하려면 `Food`가 필요하지만,

- `Food`를 `eat`하는 것이 `Animal`의 핵심 목적이 아니며
- `Animal`과 함께 동작하는 수많은 코드가 `Food`와 연관이 없을 것이기 때문에 부자연스러움.
- `Animal` 클래스를 상속받아 사용하는 곳마다 해당되는 `Food`를 지정해주어야 하고, `Food`와 같은 `타입 파라미터`가 추가로 필요해질 때마다 모든 사용처마다 적용을 해주어야 함.  
    → `Semantics`가 올바르지 않음

```swift
class Animal<Food, Habitat, Commodity> { // 늘어난 타입 파라미터
    func eat(_ food: Food) { fatalError("Subclass must implement 'eat'") }
}
```

- 근본적인 문제는 클래스가 데이터 타입이고, 슈퍼클래스를 복잡하게 만들어 `Concrete 타입`에 대한 추상적인 아이디어를 표현하려고 한다는 것.

⇒ 기능이 어떻게 작동하는지에 대한 세부 사항 없이 타입의 기능을 나타내도록 설계된 언어 구성을 원함.

- 각 동물은 특정한 타입의 음식을 가지고 있고
- 그 음식의 일부를 섭취하는 작업도 수행됨

위 기능을 나타내는 인터페이스를 Protocol로 수행할 수 있음.

## Protocol로 인터페이스 구축

프로토콜은 적합한 타입의 기능을 설명하는 추상화 도구.  
프로토콜을 사용하면 타입이 수행하는 작업에 대한 아이디어와 구현 세부 정보를 분리할 수 있음.  
타입이 수행하는 작업에 대한 아이디어는 인터페이스를 통해 표현됨.

위에서 정리한 2가지 동물의 기능을 프로토콜로 변환하면,

```swift
protocol Animal {
    associatedtype Feed: AnimalFeed
    func eat(_ food: Feed)
}
```

- `associatedtype`은 `Concrete 타입`에 대한 placeholder역할(타입 파라미터와 같음)  
    특정 타입의 동물의 각 인스턴스에는 항상 동일한 타입의 음식이 있게 됨.
- `Feed`를 소비하는 `eat(_:)` 메서드에 매핑. `Feed` 타입을 매개변수로 받음. 프로토콜에서는 구체적인 방법의 구현은 없고, 이 구현은 구체적인 `Animal` 타입에서 구현하게 됨.

이제 위에서 정의한 프로토콜로 구체적인 동물 타입을 작성할 수 있음.

```swift
 struct Cow: Animal {
    func eat(_ food: Hay) { ... }
}

struct Horse: Animal {
    func eat(_ food: Carrot) { ... }
}

struct Chicken: Animal {
    func eat(_ food: Grain) { ... }
}
```

- 프로토콜은 클래스 뿐만 아니라 구조체, 열거형, 액터 등 원하는 Semantics에 프로토콜을 사용할 수 있음.
- 각 `Animal` 타입은 `eat()`메서드를 구현해야하고, 매개변수 목록에 사용되기 때문에 컴파일러가 `Feed`가 어떤 타입인지 추론할 수 있음.

### Generic 코드 작성

이제 `Animal` 프로토콜로 `Farm`에서 feed공급 메서드를 구현할 수 있음.

`Parametric 다형성`을 사용해서 메서드가 호출될 때, `Concrete` 타입으로 대체될 `타입 파라미터`를 적용할 수 있음.

```swift
protocol Animal {
    associatedtype Feed: AnimalFeed
    func eat(_ food: Feed)
}

struct Farm {
    func feed<A: Animal>(_ animal: A) {
        // ...
    }
}
```

- 타입 파라미터 `A`를 선언하고, `A`가 `Animal` 프로토콜을 따르도록 표시.

```swift
struct Farm {
    func feed<A>(_ animal: A) where A: Animal {
        // ...
    }
}
```

- 위처럼 후행 `where`절에서 프로토콜 적합성을 지정할 수 있음. `where`절은 디테일하게 requirement와 타입 관계를 작성할 수 있게 해줌.

### some으로 프로토콜 적합성의 추상 타입을 표현

`where`절은 강력하지만 일반적인 함수에서는 복잡해보일 수 있음  
→ `some`으로 간단하게 표현이 가능!

> Swift 5.7 이전에는 프로퍼티 타입, 리턴 타입으로만 사용 가능
{: .prompt-warning }

```swift
struct Farm {
    func feed(_ animal: some Animal) {
        // ...
    }
}
```

- `where`절도 사라지고, `타입 파라미터` 리스트도 사라짐.
- 구문상 간결하며, 매개변수 선언에 `Animal` 매개변수에 대한 Semantics를 포함하기 때문에 더 간단함.

> 여기에서 `some`은 작업중인 특정 타입이 있음을 나타냄.  
`some`뒤에는 항상 적합성 `requirement`가 붙음.  
`some`키워드는 매개변수와 결과 타입에도 사용될 수 있음.
{: .prompt-tip }

SwiftUI에서 `some` `View`를 반환하는 것이 결과 타입에 사용되는 케이스!

SwiftUI에서 `View`에서 `body` 프로퍼티는 특정 타입의 `View`를 반환하지만, `body` 프로퍼티를 사용하는 코드에서는 특정 타입이 무엇인지 알 필요가 없기 때문.

```swift
var body: some View { ... }
```

### Opaque Type(불투명한 타입)과 Underlying Type(기초 타입)

`Opaque 타입`: 특정 `Concrete 타입`에 대한 placeholder를 나타내는 추상 타입.

`Underlying 타입`: 대체되는 `Concrete 타입`.

`Opaque 타입`의 값의 경우, `Underlying Type`은 값 스코프에 대해 고정됨(?).  
이 방식으로 값을 사용하는 `Generic` 코드는 값에 접근할 때마다 동일한 `Underlying 타입`을 얻도록 보장됨.

`some Animal`과 `<T: Animal>`은 모두 `Opaque 타입`을 선언한 것.

`Opaque 타입`은 input, output 모두에 사용될 수 있기 때문에,  
매개변수, result 타입 모두에 선언할 수 있는 것.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/02786f94-28e7-47d1-a74d-65e2504505f0)
[WWDC22] Embrace Swift generics 세션

`Opaque 타입`의 위치는 ‘프로그램의 어느 부분이 추상 타입을 보는지 결정하고, 프로그램의 어느 부분이 `Concrete 타입`을 결정하는지’를 결정함.

```swift
func getValue<T>(Parameter) -> Result
```

명명된 `타입 파라미터`는 항상 input 쪽에서 선언되므로 호출하는 쪽에서 `Underlying 타입`을 결정하고, 구현에서는 추상타입을 사용.

```swift
func getValue(Parameter) -> Result
```

일반적으로, `Opaque` 파라미터나 result 타입에 대한 값을 제공하는 프로그램 부분은 `Underlying 타입`을 결정하고, 값을 사용하는 프로그램 부분은 추상 타입을 봄.

### some에 대한 Underlying Type 추론

`Underlying 타입`은 값에서 추론되기 때문에, `Underlying 타입`은 항상 값과 동일한 위치에서 제공됨.

```swift
let animal: some Animal = Horse()
```

지역 변수의 경우 `Underlying 타입`은 할당 오른쪽의 값에서 유추됨.

이는 `Opaque 타입`의 지역 변수가 항상 초기 값을 가져야 함을 의미.  
제공하지 않으면 컴파일러에서 에러가 발생됨.

```inform7
var animal: some Animal = Horse()
animal = Chicken() // Error!
```

`Underlying 타입`은 변수 범위에 대해 고정되어야 하기 때문에 `Underlying 타입`를 변경하려고 하면 에러가 발생됨.

```swift
func feed(_ animal: some Animal) { }
feed(Horse())
```

`Opaque 타입`이 있는 매개변수의 경우 `Underlying 타입`은 호출부의 인수 값에서 유추됨.

```swift
func feed(_ animal: some Animal) { }
feed(Horse())
feed(Chicken())
```

`Underlying` 타입은 매개변수 스코프에 대해서만 수정하면 되므로 각 호출은 서로 다른 인수 타입을 제공할 수 있음.

```swift
func makeView(for farm: Farm) -> some View {
    FarmView(farm)
}
```

`Opaque result 타입`의 경우 `Underlying` 타입은 구현의 return 값에서 추론됨.

`Opaque result 타입`이 있는 메서드나 `연산 프로퍼티`는 프로그램의 어느 곳에서나 호출할 수 있으므로 이 명명된 값의 스코프는 전역임.

이는 `Underlying return 타입`이 모든 return 문에서 동일해야 함을 의미.

```swift
func makeView(for farm: Farm) -> some View { // Error!!!
    if condition {
        return FarmView(farm)
    } else {
        return EmptyView()
    }
}
```

그렇지 않은 경우 컴파일러는 `Underlying return 값`의 타입이 일치하지 않는다는 에러를 발생.

`Opaque` SwiftUI의 `View`는 `ViewBuilder DSL(Domain-Specific Language)`은 제어흐름 문을 변환해서 각 분기에 대해 동일한 `Underlying return 타입`을 갖도록 해줌.

```swift
@ViewBuilder
func makeView(for farm: Farm) -> some View {
    if condition {
        FarmView(farm)
    } else {
        EmptyView()
    }
}
```

이런 경우 `ViewBuilder` DSL로 문제를 해결 가능. 위처럼 메서드에 `@ViewBuilder` annotation을 달고, `return`문을 제거하면 `ViewBuilder` 타입으로 결과를 빌드할 수 있음.

다시 `feed(_:Animal)` 메서드로.

```swift
protocol Animal {
    associatedtype Feed: AnimalFeed
    func eat(_ food: Feed)
}

struct Farm {
    func feed(_ animal: some Animal) { ... }
}
```

다른 곳에서는 `Opaque` 타입을 참조할 필요가 없기 때문에 매개변수 리스트에서 `some`을 사용할 수 있음.

```swift
protocol Animal {
    associatedtype Feed: AnimalFeed
    associatedtype Havitat
    func eat(_ food: Feed)
}

struct Farm {
    func feed(_ animal: some Animal) { ... }
        func buildHome<A>(for: animal: A) -> A.Havitat where A: Animal { ... }
}
```

함수 signature에서 `Opaque` 타입을 여러 번 참조해야 하는 경우 `Name 타입 파라미터`가 유용.

예를 들어, 위처럼 `Animal` 프로토콜에 `Habitat`(서식지)라는 또다른 `associatedtype`을 추가하면 특정 동물을 위해 농장에 habitat를 build하기를 원할 수 있음.

이런 경우, 특정 `Animal`의 종류에 따라 result 타입이 달라지기 때문에  
매개변수 타입과 return 타입에 `타입 파라미터` `A`를 사용해야 함.

```swift
struct Silo<Material> {
    private var storage: [Material]

    init(storing materials: [Material]) {
        self.storage = materials
    }
}

var hayStorage: Sile<Hay> // Material을 참조하려면 명시적으로 타입파라미터를 지정
```

`Opaque` 타입을 여러번 참조하는 일반적인 경우는 `Generic` 타입.

코드는 종종 `Generic 타입`에 대한 `타입 파라미터`를 선언하고, `저장 프로퍼티`에 대한 `타입 파라미터`를 사용하고, 다시 `memberwise 이니셜라이저`에서 사용함.

다른 컨텍스트에서 `Generic 타입`을 참조하려면 `<>`안에 `타입 파라미터`를 명시적으로 지정해야 함.

선언의 `<>`는 `Generic 타입`을 사용하는 방법을 명확히 하는 데 도움이 될 수 있으므로 `Opaque` 타입은 항상 `Generic 타입`에 대해 이름을 지정해야 함.

다시, `feed(_:)` 메서드를 작성.

```swift
protocol Animal {
    associatedtype Feed: AnimalFeed
    func eat(_ food: Feed)
}

struct Farm {
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Feed.grow() // 1
                let produce = crop.harvest() // 2
                animal.eat(produce) // 3
    }
}
```

`Animal` 매개변수의 타입을 사용하여 `Feed` `associatedtype`을 통해 성장할 작물 유형에 접근 가능.

1. `Feed.grow()`를 호출하여 이러한 타입의 `Feed`를 생성하는 작물(`crop`)의 인스턴스를 가져옴.
2. 그리고 작물의 `harvest()`를 호출해서 농작물을 수확.
3. 수확한 작물을 동물에게 `eat()`

`Underlying` animal 타입이 고정되어 있기 때문에, 컴파일러는 여러 메서드 호출을 통해 식물 타입, animal 타입 간의 관계를 알 수 있음.

이런 정적 관계는 animal에게 잘못된 타입의 먹이를 주는 실수를 방지시켜줌.

(AnimalFeed 타입과 Plant간의 관계를 표현하기 위한 프로토콜에 대한 설명은 ‘[WWDC22] [Design protocol interface in Swift](https://developer.apple.com/videos/play/wwdc2022/110353/)’ 세션 참조)

이어서 모든 동물들에게 먹이를 주는 메서드를 작성.

```swift
protocol Animal {
    associatedtype Feed: AnimalFeed
    func eat(_ food: Feed)
}

struct Farm {
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Feed.grow()
        let produce = crop.harvest()
        animal.eat(produce)
    }

    func feedAll(_ animals: [some Animal]) {

    }
}
```

`element` 타입이 `Animal` 프로토콜을 따라야한다는 것을 알지만, 배열이 다양한 타입의 `Animal`을 저장할 수 있기를 원함.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/12d1c85f-92af-4e33-8820-2aedda4bd258)


`some`에는 변경할 수 없는 특정 `Underlying 타입`이 있음. `Underlying 타입`이 고정되어 있기 때문에 배열의 모든 요소는 동일한 타입을 가져야 함.

하지만 우리가 원하는 것은 다양한 타입의 `Animal`을 저장할 수 있는 배열.

### any 키워드 (exsistential 타입) & Type Erasure 전략

`any` `Animal` 이라고 쓰면 임의의 종류의 animal을 표현할 수 있음.

`any` 키워드는 이 타입이 어떤(any) 임의 타입의 animal을 저장할 수 있으며, animal의 `Underlying`타입이 런타임에 달라질 수 있음을 나타냄.

`some` 키워드와 마찬가지로 `any` 키워드 뒤에는 항상 requirement 적합성을 따름.

`any` `Animal`은 구체적인 `Animal` 타입을 동적으로 저장할 수 있는 `single static type(단일 정적 타입)`. 이를 통해 값 타입과 함께 `Subtype 다형성`을 이용할 수 있음.

이런 유연한 저장을 허용하기 위해 모든 Animal 타입은 메모리에 특별한 표현(representation)을 가짐.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7dd9f509-c39a-4b8b-aa9c-9887833b7665)

이 표현은 상자로 비유.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a8c24f8f-2ae9-42eb-b047-9a8bceaa23ef)
어떤 값은 상자 안에 직접 들어갈 만큼 작을 수도 있고,  
어떤 값은 상자에 비해 너무 커서 값을 다른 곳에 할당하고 상자 안에는 해당 값에 대한 포인터를 저장하게 됨.

구체적인 `Animal` 타입을 동적으로 저장할 수 있는 `any` `Animal`을 공식적으로는 `existential 타입`(실존타입)이라고 부름.

그리고 서로 다른 `Concrete 타입`에 대해 동일한 표현을 사용하는 전략을 `Type Erasure`라고 부름.

`Concrete 타입`은 컴파일 타임에 지워지고, `Concrete 타입`은 런타임에만 알려짐.

`exsistential 타입`인 `any` `Animal`의 두 인스턴스는 동일한 `정적 타입`을 갖지만 `동적 타입`은 다름.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e3dea663-8bb5-43c7-840a-2948982e7feb)

`Type Erasure`는 서로 다른 Animal 값 사이의 `타입 레벨 특성`을 제거하여  
서로 다른 `동적 타입`의 값을 동일한 `정적 타입`으로 교환하여 사용할 수 있게 해줌.

`Type Erasure`를 사용해서 이질적인 값 타입 배열을 작성할 수 있게 되는데, 이를 이용해서 `feedAll()` 메서드를 작성할 수 있음.

```swift
protocol Animal {
    associatedtype Feed: AnimalFeed
    func eat(_ food: Feed)
}

struct Farm {
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Feed.grow()
        let produce = crop.harvest() 
        animal.eat(produce)
    }

    func feedAll(_ animals: [any Animal]) {
        for animal in animals {
            animal.eat(???) // Error!!: Member 'eat' cannot be used on value of type 'any Animal'
        }
    }
}
```

매개변수 타입을 `any` `Animal` 배열로 받도록 함.

배열의 각 animal들의 `eat()`을 호출하려면 `underfying` `Animal`에 대한 특정 `Feed` 타입이 필요.

`eat()`을 호출하자마자 컴파일러가 에러를 보내게 됨.(Member 'eat' cannot be used on value of type 'any Animal')

특정 `Animal` 타입 간의 `타입 레벨 특성`을 제거했기 때문에 `associatedtype`을 포함해서 특정 `Animal` 타입에 의존하는 모든 타입 관계도 제거됨. 따라서 우리는 이 animal이 어떤 종류의 먹이를 기대하는지 알 수가 없게 됨.

타입 관계에 의존하기 위해서는 특정 타입의 animal이 고정되어 있는 context로 다시 돌아가야 함.

animal에서 직접 `eat()`을 호출하는 대신,  
`some` `Animal`을 허용하는 `feed()`메서드를 호출해야 함.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0af2e0e4-8e79-437b-849f-5ed0c2e82e20)
`any` `Animal`은 `some` `Animal`과 다른 타입이지만,  
컴파일러는 `underfying 값`을 unboxing하고, 이를 `some` `Animal` 매개변수에 직접 전달하여,  
`any` `Animal`의 인스턴스를 `some` `Animal`로 변환할 수 있음.

unboxing은 컴파일러가 상자를 열고 그 안에 저장된 값을 꺼내는 것으로 비유됨.

some Animal 매개변수의 scope에 대해, 값은 고정된 `underfying 타입`을 가지므로 `associatedtype`에 대한 접근을 포함한 `underfying 타입`에 대한 모든 작업에 접근할 수 있음.

이는 필요할 때 유연한 저장소를 선택할 수 있게 해주면서,  
함수 scope에 대한 `underfying 타입`을 고정시켜 `정적 타입 시스템`의 완전한 표현력을 갖는 context로 돌아갈 수 있게 만들어 줌.

그리고 대부분의 경우 unboxing에 대해 생각할 필요가 없음. 이는 `Animal`에서 프로토콜 메서드를 호출하는 것이 실제로 `underfying 타입`에서 메서드를 호출하는 방식과 유사하기 때문.

```swift
protocol Animal {
    associatedtype Feed: AnimalFeed
    func eat(_ food: Feed)
}

struct Farm {
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Feed.grow()
        let produce = crop.harvest()
        animal.eat(produce)
    }

    func feedAll(_ animals: [any Animal]) {
        for animal in animals {
            feed(animal) // any Animal을 some Animal로 unboxing
        }
    }
}
```

위처럼 `feed`로 animal(`any` `Animal`)을 전달할 수 있고,  
각 반복마다 특정 `Animal`에게 먹일 적절한 작물을 harvest하고 product시켜서 eat하도록 할 수 있음.

### some과 any 정리

`some`

- `underfying 타입`이 고정됨.
- `Generic` 코드에 대한 타입 관계에 의존할 수 있으므로, 작업 중인 프로토콜의 API 및 `associatedtype`에 대한 전체 접근 권한을 가짐.

`any`

- 임의의 `Concrete 타입`을 저장해야 하는 경우 사용.
- `Type Erasure`를 제공 ⇒ 이질적인 컬렉션을 표현할 수 있고, optional을 사용해서 `underfying 타입`이 없음을 나타내고, 추상화를 구현 세부사항으로 만들 수 있음.

일반적으로,  
기본적인 `some`을 쓰고,  
임의의 값을 저장해야 하는 상황일 때 `some` 대신 `any`를 사용.

이런 접근방식을 제공하게 되면,  
제공되는 저장소의 유연성이 필요할 때만 `Type Erasure`와 `Semantics 제한`의 비용을 지불하게 됨.

(이 작업 흐름은 기본적으로 let을 쓰고, mutation이 필요할 때만 var를 쓰는 흐름과 유사)

---

### Swift 6에서 변경 예정

([https://github.com/apple/swift-evolution/blob/main/proposals/0335-existential-any.md](https://github.com/apple/swift-evolution/blob/main/proposals/0335-existential-any.md))

기존에는 많이 사용하는 아래와 같은 케이스에서 `Existential Type`과 `Concrete Type`의 선언방식이 동일하여 구분이 되지 않는 문제가 발생.

```swift
// Existential Type인지 Concrete Type인지 구분이 되지 않음
var animal: Animal
var cow: Cow
```

```swift
// Animal class를 상속받아야 하는 것인지, Animal Protocol을 confirm해야하는 것인지 구분이 되지 않음
func feed<A: Animal>(_ animal: A) { }
```

> Swift 6에서는 프로토콜 변수를 선언할 때 프로토콜 타입명 앞에 항상 `any`를 붙이도록 강제 예정
{: .prompt-warning }

```swift
var anyAnimal: any Animal = Cow()
```