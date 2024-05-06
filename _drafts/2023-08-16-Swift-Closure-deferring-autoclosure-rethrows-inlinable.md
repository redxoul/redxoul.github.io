---
title: (Swift) Closure의 실행 지연(deferring), @autoclosure, rethrows, @inlinable
author: Cody
date: 2023-08-16 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, autoclosure, rethrows, inlinable]
pin: false
mermaid: true
---
# Building ifelse

아래에서는 `R`이 사용하는 통계 프로그래밍 언어와 같은 `ifelse()` 문을 구현.

```stylus
// R
ifelse(condition, valueTrue, valueFalse)
```

Swift 삼항 연산자 `condition ? valueTrue : valueFalse` 과 동일한 작업을 수행.

플레이그라운드에 아래 코드를 작성.

```swift
func ifelse(condition: Bool,
            valueTrue: Int,
            valueFalse: Int) -> Int {
    if condition {
        return valueTrue
    } else {
        return valueFalse
    }
}

let value = ifelse(
    condition: Bool.random(),
    valueTrue: 100,
    valueFalse: 0
)
```

위 코드는 `Int`로만 작업할 수 있기 때문에 범용적으로 사용하기 위해 수정.

먼저 인터페이스를 먼저 수정.

```swift
func ifelse(_ condition: Bool,
            _ valueTrue: Int,
            _ valueFalse: Int) -> Int {
    condition ? valueTrue : valueFalse
}
let value = ifelse(.random(), 100, 0)
```

자주 사용되는 언어 구성의 경우 인수 label을 제거하는 것이 좋음. wildcard label(`_`)은 이를 제거할 수 있는 기능을 제공.

위 코드의 문제는 이것이 `Int` 타입에 대해서만 작동한다는 것. 원하는 중요한 타입에 대해 많은 오버로드로 대체할 수 있음.

```swift
func ifelse(_ condition: Bool,
            _ valueTrue: Int,
            _ valueFalse: Int) -> Int {
    condition ? valueTrue : valueFalse
}

func ifelse(_ condition: Bool,
            _ valueTrue: String,
            _ valueFalse: String) -> String {
    condition ? valueTrue : valueFalse
}

func ifelse(_ condition: Bool,
            _ valueTrue: Double,
            _ valueFalse: Double) -> Double {
    condition ? valueTrue : valueFalse
}

func ifelse(_ condition: Bool,
            _ valueTrue: [Int],
            _ valueFalse: [Int]) -> [Int] {
    condition ? valueTrue : valueFalse
}
```

위처럼 오버로드로 각 타입을 반복 작성하는 것 대신

아래처럼 모든 `Swift` 유형에 대해 유형이 지워진 대체 타입인 `Any 타입`을 사용할 수 있음.

```swift
func ifelse(_ condition: Bool,
            _ valueTrue: Any,
            _ valueFalse: Any) -> Any {
    condition ? valueTrue : valueFalse
}
let value = ifelse(.random(), 100, 0) as! Int
```

이 코드는 모든 타입에서 작동하지만 원하는 원래 타입으로 다시 변환해야 하는 중요한 주의 사항이 있음.

`Any타입`을 사용해도 아래처럼 타입을 혼합하는 상황에서 보호되지 않음.

```swift
let value = ifelse(.random(), "100", 0) as! Int
```

이 문은 테스트에서 작동할 수 있지만 난수가 참이면 프로덕션에서 충돌이 발생.

`Any`는 매우 다재다능하지만 사용하기에 오류가 발생하기 쉬움.

더 나은 방법은 `Generic`을 사용하는 것.

```swift
func ifelse<V>(_ condition: Bool,
               _ valueTrue: V,
               _ valueFalse: V) -> V {
    condition ? valueTrue : valueFalse
}
// let value = ifelse(.random(), "100", 0) // doesn’t compile anymore
let value = ifelse(.random(), 100, 0)
```

이 디자인은 `타입 정보를 보존하고 인수를 반환 타입과 동일한 타입으로 제한`시켜줌.

## Deferring execution(실행 지연)

```swift
func expensiveValue1() -> Int {
    print("side-effect-1")
    return 2
}

func expensiveValue2() -> Int {
    print("side-effect-2")
    return 1729
}

let taxicab = ifelse(
    .random(),
    expensiveValue1(),
    expensiveValue2()
)
```

위 코드를 실행시 두 함수가 모두 호출됨. `사용하는 표현식만 evaluate`되길 원할 것.

이럴 때 실행을 지연(`defer`)시키는 `클로저`를 전달해서 이 문제를 해결할 수 있음.

```swift
func ifelse<V>(_ condition: Bool,
               _ valueTrue: () -> V,
               _ valueFalse: () -> V) -> V {
    condition ? valueTrue() : valueFalse()
}
```

위 코드는 실행을 지연시키지만, `호출방식도 변경이 필요`해짐.

```swift
let value = ifelse(.random(), { 100 }, { 0 })
let taxicab = ifelse(
    .random(),
    { expensiveValue1() },
    { expensiveValue2() }
)
```

하나의 함수만 호출되지만 인수를 클로저로 래핑해야 하는 것은 조금 귀찮을 수 있고 코드가 깔끔하지 않지만,

다행스럽게도 Swift에는 이를 고칠 방법을 제공해줌.

→ `@autoclosure`

```swift
func ifelse<V>(_ condition: Bool,
               _ valueTrue: @autoclosure () -> V, // @autoclosure 를 추가
               _ valueFalse: @autoclosure () -> V) -> V { // @autoclosure 를 추가
    condition ? valueTrue() : valueFalse()
}

func expensiveValue1() -> Int {
    print("side-effect-1")
    return 2
}

func expensiveValue2() -> Int {
    print("side-effect-2")
    return 1729
}

let taxicab = ifelse(
    .random(),
    expensiveValue1(),
    expensiveValue2()
)
```

매개변수 타입을 `@autoclosure`로 지정하면 컴파일러가 `자동으로 클로저의 인수를 래핑`.

이 변경은 호출되는 곳을 이전 상태로 복원하고 여전히 실행을 연기하므로 사용된 인수만 evaluate됨.

## Using expressions that can fail

실패가능한 표현식일 때는 위 `ifelse` 함수를 사용할 수 없음.

```swift
func expensiveFailingValue1() throws -> Int {
    print("side-effect-1")
    return 2
}

func expensiveFailingValue2() throws -> Int {
    print("side-effect-2")
    return 1729
}

let failableTaxicab = ifelse(
    .random(),
    try expensiveFailingValue1(), // Call can throw, but it is executed in a non-throwing autoclosure
    try expensiveFailingValue2() // Call can throw, but it is executed in a non-throwing autoclosure
)
```

아래처럼 다른 버전의 함수를 작성해서 해결할 수는 있음.

```swift
func ifelseThrows<V>(_ condition: Bool,
                     _ valueTrue: @autoclosure () throws -> V,
                     _ valueFalse: @autoclosure () throws -> V) throws
-> V {
    condition ? try valueTrue() : try valueFalse()
}

let taxicab2 = try ifelseThrows(
    .random(),
    try expensiveFailingValue1(),
    try expensiveFailingValue2()
)
```

위 코드의 문제점: 첫 번째 표현식만 `throw`하거나 두 번째 표현식만 `throw`한다고 가정하면, 모든 경우를 처리하기 위해 동일한 기능의 네 가지 버전을 만들어야 함. `throws 키워드는 Swift 함수의 signature에 포함되지 않기 때문`에 이름이 약간 다른 네 가지 `ifelse`가 필요함. 다행히 이러한 모든 경우를 처리하는 한 가지 버전의 함수를 작성할 수 있음.

→ `rethrows`

```swift
func ifelse<V>(_ condition: Bool,
               _ valueTrue: @autoclosure () throws -> V,
               _ valueFalse: @autoclosure () throws -> V)
rethrows -> V {
    condition ? try valueTrue() : try valueFalse()
}
```

`rethrows`는 실패한 클로저의 오류를 호출자에게 전달시켜줌. 클로저 매개변수가 `throw`되지 않으면 함수가 `throw`되지 않으며 `try`로 표시할 필요가 없다고 추론.

위 코드는 아래의 모든 케이스를 동작시켜줌.

```swift
let value = ifelse(.random(), 100, 0 )
let taxicab = ifelse(
    .random(),
    expensiveValue1(),
    expensiveValue2()
)

let taxicab2 = try ifelse(
    .random(),
    try expensiveFailingValue1(),
    try expensiveFailingValue2()
)

let taxicab3 = try ifelse(
    .random(),
    expensiveValue1(),
    try expensiveFailingValue2()
)

let taxicab4 = try ifelse(
    .random(),
    try expensiveFailingValue1(),
    expensiveValue2()
)
```

추가 `추상화 계층 비용을 발생`시키지 않고 싶고 `구현이 절대 변경되지 않으므로 @inlinable 함수를 표시`하는 것이 좋음.

이렇게 추가된 키워드는 `메서드의 본문이 함수 호출 오버헤드 없이 클라이언트 코드에 직접 포함`되어야 함을 컴파일러에 암시시켜줌.

```swift
@inlinable
func ifelse<V>(_ condition: Bool,
               _ valueTrue: @autoclosure () throws -> V,
               _ valueFalse: @autoclosure () throws -> V)
rethrows -> V {
    condition ? try valueTrue() : try valueFalse()
}
```

> `Any`는 `Swift`의 궁극적인 `type-erasure`이지만 사용 시 오류가 발생하기 쉬움. 일반적으로 `Generic`이 더 나은 대안.  
> 클로저를 값을 반환하는 매개변수로 전달하여 함수 본문 내에서 인수 evaluate를 연기.  
> `@autoclosure`는 표현식 인수의 실행을 지연시키기 때문에 short-circuit 동작을 구현하는 방법.  
> `rethrow`는 `throws`로 표시되거나 표시되지 않을 수 있는 클로저에서 오류를 전파하는 방법.  
> `@inlinable`은 함수에 대한 명령을 호출 사이트로 내보내야 함을 컴파일러에 암시.
{: .prompt-info }

## Performance

`optimizing 컴파일러`로 프로그램을 작성할 때 멋진 점 중 하나는 코드를 명확하고 유지 관리 가능하게 만드는 추상화 비용이 없거나 거의 없다는 것.

여기에서 수행한 작업을 보려면 이 코드를 `ifelse.swift`라는 텍스트 파일에 작성.

```swift
@inlinable
func ifelse<V>(_ condition: Bool,
               _ valueTrue: @autoclosure () throws -> V,
               _ valueFalse: @autoclosure () throws -> V)
rethrows -> V {
    condition ? try valueTrue() : try valueFalse()
}
func ifelseTest1() -> Int {
    if .random() {
        return 100
    } else {
        return 200
    }
}
func ifelseTest2() -> Int {
    Bool.random() ? 300 : 400
}
func ifelseTest3() -> Int {
    ifelse(.random(), 500, 600)
}
```

그리고 커맨드라인에서 아래를 실행.

```
swiftc -O -emit-assembly ifelse.swift > ifelse.asm
```

어셈블리 파일을 열어보면 이 파일에는 호출 규칙 및 진입점에 대한 수많은 상용구 의식이 포함되어 있음.

불필요한 부분을 무시하고 아래를 확인.

```
_$s6ifelse0A5Test1SiyF:
 :
 callq _swift_stdlib_random
 testl $131072, -8(%rbp)
 movl $100, %ecx
 movl $200, %eax
 cmoveq %rcx, %rax
 :
_$s6ifelse0A5Test2SiyF:
 :
 callq _swift_stdlib_random
 testl $131072, -8(%rbp)
 movl $300, %ecx
 movl $400, %eax
 cmoveq %rcx, %rax
 :
_$s6ifelse0A5Test3SiyF:
 :
 callq _swift_stdlib_random
 testl $131072, -8(%rbp)
 movl $500, %ecx
 movl $600, %eax
 cmoveq %rcx, %rax
 :
```

ifelseTest1(), ifelseTest2() 및 ifelseTest3()에 대한 assembly 지침.

`callq` 명령어는 함수를 호출하여 난수를 가져오기.

`testl` 명령어는 난수 반환 값(64비트 기본 포인터가 가리키는 주소 -8에 있음)을 가져옴. 0x20000 또는 17번째 비트인 `131072`에 대해 이를 확인. `Bool.random`에 대한 Swift 소스를 보면 아래를 찾을 수 있음.

```swift
@inlinable
public static func random<T: RandomNumberGenerator>(
    using generator: inout T
) -> Bool {
    return (generator.next() >> 17) & 1 == 0
}
```

이것이 `131072`의 의미.

17번째 비트를 이동하고 마스킹하고 모든 것을 하나의 명령으로 테스트.

다음으로 함수의 두 가지 가능한 결과 값이 레지스터 `cx` 및 `ax`로 이동(`movl` 명령어 사용). 접두사 `e`는 레지스터의 확장된 32비트 버전을 나타냄. 나머지 비트는 모두 64비트를 채우기 위해 0을 붙임.

마지막으로 `cmoveq` 명령어(같으면 조건부 이동)는 이전 테스트 명령의 결과를 사용하여 `cx` 레지스터를 `ax` 레지스터로 이동. `rcx` 및 `rax`의 접두사 `r`은 레지스터의 전체 64비트를 사용함을 나타냄.

> 맹글링된 symbol \_$s6ifelse0A5Test1SiyF:는 ifelse.ifelseTest1() -> Int의 고유한 symbol 이름.(앞의 ifelse.는 모듈 이름 또는 이 경우 파일 이름) Linker는 프로그램의 모든 외부 기호에 대해 짧고 보장된 고유 이름이 필요. [https://github.com/apple/swift/blob/main/docs/ABI/Mangling.rst](https://github.com/apple/swift/blob/main/docs/ABI/Mangling.rst) 에서 mangling 사양을 찾을 수 있음. /Library/Developer/CommandLineTools/usr/bin/. 에 있는 명령줄 도구 swift-demangle을 실행할 수도 있음. 예를 들어 `swift-demangle \_\\$s6ifelseAAyxSb\_xyKXKxyKXKtKlF` 은 `ifelse.ifelse<A>(Swift.Bool, @autoclosure () throws -> A, @autoclosure () throws -> A) throws -> A` 의 symbol로 확인됨.
{: .prompt-info }

> 컴파일러는 소스 코드의 추상화 비용 전부는 아니더라도 많은 부분을 제거함. 다른 소스 코드에 동일한 의미가 있는 경우 컴파일러는 동일한 기계 명령어를 내보낼 가능성이 높음.  추상화는 비용을 지불해야 함. (ifelse와 같은)새로운 언어 기능을 만들기 전에 신중하게 생각해야 함.
{: .prompt-info }



