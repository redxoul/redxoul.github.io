---
title: (RxCocoa) XCTest와 Nimble
author: Cody
date: 2023-01-04 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift, RxCocoa]
pin: false
mermaid: true
---


### XCTest
`Xcode`에서는 `UnitTest`, `UITest`를 위해 `XCTest`라는 `framework`를 제공합니다.  
`XCTest`를 작성하는 방법은 다음과 같습니다.

(1) `XCTestCase`를 상속받는 클래스를 만듭니다.

(2) `setUp` 메서드를 `override`한 후 작성해 주면,

테스트 코드가 동작하기 전에 호출되어 테스트하는데 필요한 리소스들을 미리 세팅해 줄 수 있습니다.

(3) `test`로 시작하는 메서드를 작성하고 테스트 코드를 작성해줍니다.

```swift
final class LocationInfoModelTests: XCTestCase {

    override func setUp() {
        // 기초 설정
        // 테스트가 시작되기 전에 호출되어집니다.
    }
    
    func testSomeExpectation() {
        // 테스트 코드 작성
    }
}
```

테스트 코드를 작성할 때는

`XCTAssert~` 형태의 메서드를 호출하여 테스트에서 기대하는 결과를 비교해줍니다.

`XCTAssert`에는 아래와 같은 종류들이 있습니다.

```swift
XCTAssertEqual(expression1, expression2, ...)       // 두 expression이 같은지 테스트
XCTAssertNotEqual(expression1, expression2, ...)    // 두 expression이 다른지 테스트
XCTAssert(expression, ...)      // expression이 true인지 테스트
XCTAssertTrue(expression, ...)  // expression이 true인지 테스트
XCTAssertFalse(expression, ...) // expression이 false인지 테스트
XCTAssertNil(expression, ...)       // expression이 nil인지 테스트
XCTAssertNotNil(expression, ...)    // expression이 nil이 아닌지 테스트
XCTAssertGreaterThan(expression1, expression2, ...) // expression1이 expression2보다 큰지 테스트
XCTAssertGreaterThanOrEqual(expression1, expression2, ...) // expression1이 expression2보다 크거나 같은지 테스트
XCTAssertLessThanOrEqual(expression1, expression2, ...) // expression1이 expression2보다 작거나 같은지 테스트
XCTAssertLessThan(expression1, expression2, ...) // expression1이 expression2보다 작은지 테스트
```

더 많은 종류의 `Assertion`들은 문서를 참고

https://developer.apple.com/documentation/xctest/comparable_value_assertions

사용예는 아래와 같습니다.

각 `Assertion`에 맞는 `expression`을 넣고, 뒤에 디테일한 메세지를 커스텀하게 남길 수도 있습니다.

```swift
final class LocationInfoModelTests: XCTestCase {

    override func setUp() {
        // 기초 설정
        // 테스트가 시작되기 전에 호출되어집니다.
    }
    
    func testSomeExpectation() {
        // 테스트 코드 작성
        let someMethod = { return "SwiftyCody"}
        let someValue = "SwiftyCody"
        let anotherValue = "Swifty"
        
        XCTAssertEqual(someMethod(), someValue, "someMethod(\(someMethod()))와 someValue(\(someValue))의 값이 같아야 합니다.") // 두 값이 같은지 테스트
        XCTAssertEqual(someMethod(), someValue) // Custom Message 생략
        XCTAssertNotEqual(someValue, anotherValue) // 두 값이 다른지 테스트
    }
}
```

위와 같이 기댓값과 다른 결과가 `Assertion`을 거치면,

에러가 나면서 작성한 커스텀 메세지나 기본 메세지가 경고를 해줍니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b41ce4d3-7f25-4adb-b532-905946a76121)

### Nimble

`Nimble`은 `Test Assertion`을 좀 더 읽기 쉬운 표현으로 작성할 수 있게 해줍니다.

기본적으로 제공되는 `Assertion`들은 항목들에 어떤 값들을 넣어야하는지 직관적이지 않은데,

`Nimble`은 구문에서부터 파악하기가 쉽니다.

`Equality`, `Inequality`, `Comparable` Value의 경우 `==`, `!=`, `>=`, `<=` 같은 `operator`들로 쉽게 표현할 수도 있습니다.

```swift
XCTAssertEqual(someMethod(), someValue, "someMethod(\(someMethod()))와 someValue(\(someValue))의 값이 같아야 합니다.") // 두 값이 같은지 테스트
// Nimble로 표현(1)
expect(someMethod()).to(equal(someValue), description: "someMethod(\(someMethod()))와 someValue(\(someValue))의 값이 같아야 합니다.")
// Nimble로 표현(2)
expect(someMethod()) == someValue
```