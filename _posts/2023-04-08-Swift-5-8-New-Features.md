---
title: Swift 5.8 New Features
author: Cody
date: 2023-04-08 00:34:00 +0800
categories: [iOS, Swift]
tags: [iOS, Swift, Swift5.8, Xcode14.3]
pin: false
mermaid: true
---
`Xcode 14.3`이 릴리즈되면서 `Swift 5.8`도 업데이트되었습니다.

변경점들을 간단하게 정리해봅니다.

## Function back deployment ([SE-0376](https://github.com/apple/swift-evolution/blob/main/proposals/0376-function-back-deployment.md))

`@backDeployed(before:)` `attribute`를 통해 이전 버전의 프레임워크에서 새 API를 사용할 수 있게 해줍니다.

함수에 대한 코드를 앱의 바이너리에 작성 후 런타임 시 검사하여 수행됩니다. 사용자가 적절한 새 OS를 사용하는 경우 시스템 자체 버전의 함수가 사용되고, 아닌 경우 앱 바이너리에 복사된 버전에 대신 사용됩니다.

단, `@backDeployed`는 `함수`, `메서드`, `서브스크립트`, `연산프로퍼티`에만 적용됩니다.

당연하게도 만능으로 새 기능을 이전 OS에서 쓸 수 있게 한다던가 하는건 아닌거죠.

아래는 예시입니다.

```swift
@available(macOS 12, *)
public struct Temperature { // MacOS 12부터 사용중인 Temperature 구조체
    public var degreesCelsius: Double

// ...
}

extension Temperature {
    @available(macOS 12, *)
    @backDeployed(before: macOS 13)
    public var degreesFahrenheit: Double { // MacOS 13에서 새로 추가된 계산프로퍼티
        return (degreesCelsius * 9 / 5) + 32
    }
}
```

기존에 위와 같이 `MacOS 12`에서 `Temperature` 구조체를 사용하고 있었다는 예시입니다.

개발자들도 해당 구조체를 사용하여 날씨 앱을 만들어 사용하고 있었는데,

`MacOS 13`에서 해당 구조체에 `@backDeployed`가 적용된 `degreesFahrenheit` 연산프로퍼티가 추가된 상황입니다.

이런 경우 개발자가 저 `degreesFahrenheit` 연산프로퍼티를 활용할 때 `이전 OS(MacOS 12)` 사용자에게도 동일한 프로퍼티를 사용하여 기능을 제공할 수 있다는 의미입니다.

새로 추가된 `degreesFahrenheit` 프로퍼티를 사용해서 앱을 배포하면 해당 코드는 앱 바이너리에 작성되어 `이전 OS(MacOS 12)`에서는 이것이 런타임에 불려지고, `새 OS(MacOS 13)`부터는 시스템 자체에서 불러오게 됩니다.

예전이었다면 새 `API`에 대해서 무조건 `@available(MacOS 13, *)` 으로 구분해주어 썼었겠지만, 이제부터는 `@backDeployed`가 적용된 `함수`, `메서드`, `서브스크립트`, `연`산프로퍼티``가 있다면 그렇지 않은 케이스도 생긴 것입니다.

위에서 쓴대로 이 프로퍼티 래퍼가 만능이 아니라 매우 제한적이지만, 다가오는 6월 `WWDC2023`에서 새로운 `API`가 공개될 때 과연 어떤 `API`에 `@backDeployed`가 적용이 되어 나올지 찾아보는 재미도 있을거 같습니다. 저는 새 `API` 뿐만 아니라 기존의 `API`도 `@backDeployed`가 일부 적용될 수 있지 않을까 기대해봅니다.(Combine에 새 operator가 나오고 `@backDeployed`가 적용된다면?!)

## Сollection downcasts in cast patterns are now supported

이제 캐스트 패턴에서 `Collection`의 다운캐스트가 지원됩니다.

예시입니다.

```swift
func collectionDowncast(_ arr: [Any]) {
    switch arr {
    case let ints as [Int]: // [Any] -> [Int] 다운캐스트
				// ...
    case is [Bool]:
				// ...
    }
}
```

위 코드는 `Swift 5.8`에서부터 정상동작할 것입니다.

이전 버전에서는 해당 라인에서 "Collection downcast in cast pattern is not implemented; use an explicit downcast to '[Int]' instead."라고 에러가 발생할 것이고 아래처럼 사용해야 할 것입니다.

```swift
if let ints = arr as? [Int] { ...
```

## Implicit self is now permitted for weak self captures, after self is unwrapped([SE-0365](https://github.com/apple/swift-evolution/blob/main/proposals/0365-implicit-self-weak-capture.md))

우리가 `@escaping` 클로저를 보낼 때 `self`에서의 동작을 하게 될 때

캡쳐리스트로 `[weak self]`를 작성하고, `guard let` 구문으로 언래핑 후 `self.method()` 형태로 작성을 많이 하는데요.

`Swift 5.8`부터는 이런 경우, `self.`를 생략하여 암시적 호출이 가능해집니다.

```swift
class ViewController {
    let button: Button

    func setup() {
        button.tapHandler = { [weak self] in
            guard let self else { return }
            dismiss() // Swift5.7까지는 `self.dismiss()`로 작성
        }
    }

    func dismiss() { ... }
}
```

`Swift 5`버전까지는 아래 코드처럼 `[weak self]`를 캡쳐하는 `non-escaping` 클로저에서 언래핑을 하지 않더라도 `self.`를 생략한 암시적 호출이 가능합니다.

하지만 앞으로 공개될 `Swift 6`에서부터는 이런 케이스에서 `error`가 발생합니다. `Swift 6`에서는 `non-escaping`클로저와 `escaping`클로저 모두 `[weak self]`를 캡쳐시 동일하게 동작하기 때문에 `if let`, `guard let`으로 언래핑으로 하거나 `[unowned self]`, `[self]` 캡쳐방식을 선택해야 합니다([SE-0365](https://github.com/apple/swift-evolution/blob/main/proposals/0365-implicit-self-weak-capture.md)).

```swift
class ExampleClass {
    func makeArray() -> [String] {
				// `Array.map` takes a non-escaping closure:
        ["foo", "bar", "baaz"].map { [weak self] string in
            double(string) // Call to method 'double' in closure requires explicit use of 'self' to make capture semantics explicit; this is an error in Swift 6
        }
    }

    func double(_ string: String) -> String {
        string + string
    }
}
```

## The compiler flag -enable-upcoming-feature X

이번 `Swift 5.8`에는 포함이 되었지만 활성화되어 있지 않은 `feature`들이 있습니다.

이 `feature`들은 다음 `major`버전(`Swift 6`)에서 오픈될 예정인데요.

`Xcode 14.3`에서는 `complier flag`에 `-enable-upcoming-feature X(feature명)`을 세팅함으로써 미리 활성화해볼 수 있습니다.

해당 `flag`로 미리 활성화할 수 있는 몇가지가 소개되었습니다.

(소스에서는 `#if hasFeature(X)`를 통해서 구분할 수 있습니다)

- Concise magic file names([SE-0274](https://github.com/apple/swift-evolution/blob/main/proposals/0274-magic-file.md))

- `enable-upcoming-feature ConciseMagicFile` 플래그를 세팅하여 활성화합니다.

`#file` 식별자를 썼을 때 매우 긴 full 경로, 예를 들어 `/Users/becca/Desktop/0274-magic-file.swift` 와 같이 불러왔었는데

이제는 `MagicFile/0274-magic-file.swift` 처럼 간결한 `<module-name>/<file-name>` 형식으로 가져옵니다.

기존처럼 전체경로가 필요한 경우 `#filePath`를 호출하면 됩니다.

표준 라이브러리의 `assertion` 및 `error` 메세지는 `#file`을 사용합니다.

```swift
print(#file)
print(#filePath)
fatalError("Something bad happened!")
```

출력

```
MagicFile/0274-magic-file.swift
/Users/becca/Desktop/0274-magic-file.swift
Fatal error: Something bad happened!: file MagicFile/0274-magic-file.swift, line 3
```

- Forward-scan matching for trailing closures([SE-0286](https://github.com/apple/swift-evolution/blob/main/proposals/0286-forward-scan-trailing-closures.md))

- `enable-upcoming-feature ForwardTrailingClosures` 플래그를 세팅하여 활성화합니다.

기존의 후행클로저의 역방향 스캔 매칭 규칙을 정방향 스캔 매칭으로 변경합니다.

(이것은 설명이 길어서 문서를 참조)

- regex literal syntax([SE-0354](https://github.com/apple/swift-evolution/blob/main/proposals/0354-regex-literals.md))

- `enable-upcoming-feature BareSlashRegexLiterals` 플래그를 세팅하여 `BareSlash`정규식 리터럴을 활성화합니다.

## Result builders 내의 모든 변수 사용제한 해제(SwiftUI, [SE-0373](https://github.com/apple/swift-evolution/blob/main/proposals/0373-vars-without-limits-in-result-builders.md))

`Result builder` 내에서 변수 선언 및 사용에 대한 제한이 해제되었습니다.

이제 `Result builder`에서 아래와 같은 변수들을 제한없이 사용할 수 있습니다.

- 초기화되지 않은 변수
- 기본적으로 초기화된 변수
- 계산 변수
- 관측 변수
- property wrapper가 있는 변수
- lazy 변수

해당 제한들 없이 `Result builder` 내 지역변수를 활용한 예시들입니다.

```swift
import SwiftUI

// 사용자 입력 확인 및 서식 지정
struct ContentView: View {
    var body: some View {
        GeometryReader { proxy in
            @Clamped(10...100) var width = proxy.size.width
            Text("\(width)")
        }
    }
}
```

```swift
import SwiftUI

// UserDefaults와 상호작용
struct AppIntroView: View {
    var body: some View {
        @UserDefault(key: "user_has_ever_interacted") var hasInteracted: Bool
        ...
        Button("Browse Features") {
            ...
            hasInteracted = true
        }
        Button("Create Account") {
            ...
            hasInteracted = true
        }
    }
}
```

---

`WWDC23`을 앞두고 미리 `Swift`, `SwiftUI`, `Xcode` 등을 보강하고 앞으로를 기대하게 만드는 내용도 포함하여 재밌엇습니다.

더 자세한 내용은 [Xcode 14.3 Release Note](https://developer.apple.com/documentation/xcode-release-notes/xcode-14_3-release-notes)를 참조하면 좋습니다.