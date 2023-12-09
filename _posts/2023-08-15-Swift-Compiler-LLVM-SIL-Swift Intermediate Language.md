---
title: (Swift) Swift Compiler(LLVM), SIL(Swift Intermediate Language)
author: Cody
date: 2023-08-16 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, Compiler, LLVM, SIL]
pin: false
mermaid: true
---
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e5981c63-180a-45aa-b633-bba86e5e0b1e){: width="50%" height="50%"}


`LLVM` 프로젝트는 모듈식의 재사용 가능한 컴파일러와 툴체인의 집합.
`LLVM`이라는 이름은 약자가 아니며 그것이 오픈소스 프로젝트의 풀 네임.
(`LLVM`은 `Swift` 뿐만 아니라 `Kotlin`, `Rust` 등에서도 사용중)
 
`Swift 툴체인`의 핵심은 `Swift 컴파일러`이며 `소스 코드(.swift)`를 실행 파일에 연결할 수 있는 `object코드(.o)`로 변환하는 역할을 함.

## LLVM 컴파일러 인프라에서 실행되는 데이터 흐름.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e0ed4b49-a5c3-4336-bddb-1a49ff6ffaae){: width="60%" height="60%"}

1. `Parse(구문 분석)`: Swift 소스 코드는 먼저 토큰으로 `Parse`되고  `Abstract Syntax Tree(AST. 추상 구문 트리)`에 입력됨. 이것은 각 표현식이 노드인 트리라고 생각할 수 있음. 노드는 또한 소스 위치 정보를 가지고 있어서 error가 감지되면 노드는 문제가 발생한 정확한 위치를 알려줄 수 있음.
2. `Semantic Analysis(Sema, 의미 분석)`: 이 단계에서 컴파일러는 `AST`를 사용하여 프로그램의 의미를 분석합니다. 여기서 `Type Check`가 발생. `Type Check`된 `AST`를 `SILGen` 단계로 전달.
3. `SILGen`: 이 단계는 이 단계가 없었던 Clang과 같은 이전 컴파일러 파이프라인에서 출발. `AST`는 `SIL(Swift Intermediate Language, 중간 언어)`로 낮아짐. `SIL`에는 기본 연산 블록이 포함되어 있으며 `Swift 타입`들, `참조 카운팅` 및 `디스패치 규칙`을 이해. `SIL`에는 `Raw(원시)` 및 `Canonical(정규화)`의 두 가지 타입이 있음. `Raw SIL`의 `Canonical SIL` 결과는 최소 최적화 단계 세트를 통해 실행(모든 최적화가 해제된 경우에도). `SIL`에는 소스 위치 정보도 포함되어 있어 의미 있는 error를 생성할 수 있음.
4. `IRGen`: 이 도구는 `SIL`을 `LLVM`의 중간 표현으로 낮춤. 이 시점에서 명령(instruction)은 더 이상 `Swift`에 한정되지 않음. (모든 `LLVM` 기반은 이 표현을 사용) `IR`은 매우 추상적. `SIL`과 마찬가지로 `IR`은 `Static Single Assignment(SSA, 정적 단일 할당)` 형식. 무제한의 레지스터를 갖는 것으로 machine를 모델링하여 최적화를 더 쉽게 찾을 수 있음. `Swift 타입`에 대해서는 아무것도 모름.
5. `LLVM`: 이 마지막 단계는 `IR`을 최적화하고 특정 플랫폼에 대한 기계 명령어로 낮춤. 백엔드(기계 명령어 출력)에는 ARM, x86, Wasm 등이 포함됨.
위의 다이어그램은 `Swift 컴파일러`가 `object 코드(.o)`를 생성하는 방법을 보여줌. `Source code formatter`, `Refactoring tool`, `문서 생성기` 및 `Syntax highlighter`와 같은 다른 도구는 `AST`와 같은 중간 결과를 활용하여 최종 결과를 보다 강력하고 일관되게 만들 수 있음.
   

## SIL(Swift Intermediate Language)

### Overflow 감지

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/bd6d8ff3-6dae-4635-949e-96658b38e97c)
`SILGen`의 pass로 인해 컴파일러가 소스를 정적으로 분석하고 130이 Int8(~127)에 맞지 않다는 걸 확인할 수 있음.  
   
 

### Definite initialization(명확한 초기화)

`Swift`는 기본적으로 초기화되지 않은 메모리에 액세스하기 어렵게 만드는 안전한 언어.
`SILGen`은 `Definite initialization`라는 확인 프로세스를 통해 개런티를 제공.

```swift
final class Printer {
    var value: Int
    init(value: Int) { self.value = value }
    func print() { Swift.print(value) }
}

func printTest() {
    var printer: Printer
    if .random() {
        printer = Printer(value: 1)
    }
    else {
        printer = Printer(value: 2)
    }
    printer.print()
}

printTest()
```

위 코드는 잘 동작하지만 `else`문을 주석처리시 컴파일러가 `SIL`으로 인해 `error(Variable ‘printer’ used before being initialized)`에 정확히 플래그를 지정. 이 `error`는 `SIL`이 `Printer`에 대한 메서드 호출의 의미 체계를 이해하기 때문에 가능.
 

### Allocation and devirtualization

`SILGen`은 할당 및 메서드 호출의 최적화를 도움. 다음 코드를 사용하여 즐겨 사용하는 일반 텍스트 편집기를 사용하여 `magic.swift`라는 파일에 넣기.

```swift
class Magic {
    func number() -> Int { return 0 }
}

final class SpecialMagic: Magic {
    override func number() -> Int { return 42 }
}

public var number: Int = -1

func magicTest() {
    let specialMagic = SpecialMagic()
    let magic: Magic = specialMagic
    number = magic.number()
}

magicTest() // 42
```

클래스의 가상테이블을 사용하여 값 `42`를 반환하는 올바른 함수를 찾아냄.  
 

### Raw SIL

터미널 창에서 `magic.swift`가 있는 소스 디렉토리로 변경하고 다음 명령을 실행.

```shell
swiftc -O -emit-silgen magic.swift > magic.rawsil
```

> ⚠️ 커맨드라인툴 설치가 자꾸 뜨면 터미널에 아래를 실행 후 위 명령어 수행.  
> `sudo xcode-select -s /Library/Developer/CommandLineTools`

이것은 최적화된 `Swift 컴파일러`를 실행하고 `Raw SIL`을 생성하여 `magic.rawsil` 파일로 출력. 텍스트 편집기에서 `magic.rawsil`을 열고 아래로 스크롤하면 `magicTest()` 함수의 다음 정의를 찾을 수 있음.

```shell
// magicTest()
sil hidden [ossa] @$s5magic0A4TestyyF : $@convention(thin) () ->
() {
bb0:
 %0 = global_addr @$s5magic6numberSivp : $*Int // user: %14
 %1 = metatype $@thick SpecialMagic.Type // user: %3 // function_ref SpecialMagic.__allocating_init()
 %2 = function_ref @$s5magic12SpecialMagicCACycfC : $@convention(method) (@thick SpecialMagic.Type) -> @owned SpecialMagic // user: %3
 %3 = apply %2(%1) : $@convention(method) (@thick SpecialMagic.Type) -> @owned SpecialMagic // users: %18, %5, %4
 debug_value %3 : $SpecialMagic, let, name "specialMagic" // id: %4
 %5 = begin_borrow %3 : $SpecialMagic // users: %9, %6
 %6 = copy_value %5 : $SpecialMagic // user: %7
 %7 = upcast %6 : $SpecialMagic to $Magic // users: %17, %10, %8
 debug_value %7 : $Magic, let, name "magic" // id: %8
 end_borrow %5 : $SpecialMagic // id: %9
 %10 = begin_borrow %7 : $Magic // users: %13, %12, %11
 %11 = class_method %10 : $Magic, #Magic.number : (Magic) -> () -> Int, $@convention(method) (@guaranteed Magic) -> Int // user: %12
 %12 = apply %11(%10) : $@convention(method) (@guaranteed Magic) -> Int // user: %15
 end_borrow %10 : $Magic // id: %13
 %14 = begin_access [modify] [dynamic] %0 : $*Int // users: %16, %15
 assign %12 to %14 : $*Int // id: %15
 end_access %14 : $*Int // id: %16
 destroy_value %7 : $Magic // id: %17
 destroy_value %3 : $SpecialMagic // id: %18
 %19 = tuple () // user: %20
 return %19 : $() // id: %20
} // end sil function '$s5magic0A4TestyyF'
```

위는 `magicTest()`의 `SIL` 정의. 레이블 `bb0`은 기본 블록 `0`을 나타내며 계산 단위. (`if/else` 문이 있는 경우 가능한 각 경로에 대해 두 개의 기본 블록 `bb1` 및 `bb2`가 생성됨) `%1`, `%2` 등의 값은 `가상 레지스터`.
`SIL`은 `SSA(Single Static Assignment, 단일 정적 할당)` 형식이므로 레지스터는 무제한이며 재사용되지 않음. 여기서 논의에 중요하지 않은 더 많은 작은 세부 사항이 있음. 이를 통해 개체를 `allocating`, `assigning`, cal`ling 및 `deallocating`하는 방법을 대략적으로 볼 수 있음. 이것은 Swift 언어의 전체 semantics를 표현.
 

### Canonical SIL

`Canonical SIL`에는 `-Onone`으로 최적화가 해제된 경우 최소화된 최적화 pass 세트를 포함하여 최적화를 포함시킴.  
다음 터미널 명령을 실행.

```shell
swiftc -O -emit-sil magic.swift > magic.sil
```

위 명령은 `Canonical SIL`을 포함하는 `magic.sil` 파일을 생성.  
파일 끝으로 스크롤하여 magicTest()를 찾기.

```shell
// magicTest()
sil hidden @$s5magic0A4TestyyF : $@convention(thin) () -> () {
bb0:
 %0 = global_addr @$s5magic6numberSivp : $*Int // user: %3
 %1 = integer_literal $Builtin.Int64, 42 // user: %2
 %2 = struct $Int (%1 : $Builtin.Int64) // user: %4
 %3 = begin_access [modify] [dynamic] [no_nested_conflict] %0 : $*Int // users: %4, %5
 store %2 to %3 : $*Int // id: %4
 end_access %3 : $*Int // id: %5
 %6 = tuple () // user: %7
 return %6 : $() // id: %7
} // end sil function '$s5magic0A4TestyyF'
```

위 파일은 `Raw SIL`과 동일한 것을 표현하지만 훨씬 간결함.
주요 작업은 정수 리터럴 42를 전역 주소 위치 `store %2 to %3 : $*Int`에 저장하는 것
어떤 클래스도 초기화 또는 초기화 해제되지 않으며 가상 메서드도 호출되지 않았음. struct가 스택을 사용하고 class가 힙을 사용한다는 말을 들었을 때 이는 일반화임을 명심해야 함. `Swift에서는 모든 것이 힙에서 초기화되기 시작하며 SIL 분석을 통해 allocation을 스택으로 이동하거나 아예 제거할 수도 있음`. 가상 함수 호출은 최적화 프로세스를 통해 `devirtualize`되지 않고 직접 또는 인라인으로 호출될 수도 있음.
 

> 📄  
> 소스 위치 정보는 `AST` 및 `SIL`에 상주하므로 더 나은 오류 보고가 가능.  
> `SIL`은 `SSA` 형식으로 작성된 기본 명령 블록을 사용하는 저수준 설명. 순수한 `LLVM`, `IR`로는 불가능한 많은 최적화를 가능하게 하는 Swift 유형의 semantics를 이해.  
> SIL은 명확한 초기화, 메모리 할당 최적화 및 가상화 해제를 지원.