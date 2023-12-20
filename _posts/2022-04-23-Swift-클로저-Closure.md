---
title: (Swift) 클로저 Closure
author: Cody
date: 2022-04-23 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

클로저(Closure)는 코드 블럭 `{ }` 으로 Objective-C의 `block`이나 타 언어의 `람다`, `익명함수`와 유사한 개념입니다.

Swift에서는 함수가 '이름이 있는 클로저'라고 하는 편이 맞습니다.

이전 글에서 Swift에서는 함수가 [1급객체](https://swiftycody.github.io/posts/Swift-1%EA%B8%89-%EA%B0%9D%EC%B2%B4-First-class-citizen-%EC%99%80-%EA%B3%A0%EC%B0%A8%ED%95%A8%EC%88%98-%EA%B3%A0%EA%B3%84%ED%95%A8%EC%88%98/)라는 포스팅을 했는데, 바로 클로저의 존재가 이를 가능하게 해줍니다.

### 클로저의 형태

클로저는 아래의 세가지 형태가 있습니다.

- `전역 함수` : 이름이 있고 어떤 값도 캡쳐하지 않는 클로저
- `중첩 함수` : 이름이 있고 관련한 함수로 부터 값을 캡쳐 할 수 있는 클로저
- `클로저 표현` : 경량화 된 문법으로 쓰여지고 관련된 문맥(context)으로부터 값을 캡쳐할 수 있는 이름이 없는 클로저

### 클로저 표현

Swift에서 클로저 표현은 간결하고 명확한 표현을 지원해줍니다.

- 기본 표현 문법

```swift
{ (매개변수들) -> 반환타입 in
    클로저 body
}
```

```swift
// 예시
{ (x: Int, y: Int) -> Bool in
    print("x: ", x, "y: ", y)
}
```

클로저는 코드블럭 `{ }` 내에서 크게 `in` 앞과 뒤로 나뉘어 집니다.

in 앞쪽은 (매개변수들) -> 반환타입 으로 클로저의 형태를 작성해주고,

in 뒷쪽은 body로 클로저 내에서 매개변수로 처리해줄 내용을 작성해줍니다.

그리고 Swift는

1. 문맥(context)에서 `매개변수 타입(parameter type)과 반환타입(return type)의 추론`,

2. 단일 표현 클로저에서의 `암시적 반환`,

3. 축약된 인자 이름,

4. 후위 클로저 문법`을 통해 최적화를 할 수 있도록 해줍니다.

예시를 통해서 최적화 방법을 알아보겠습니다.

Swift 표준 라이브러리에는 `sorted(by:)`라는 메서드가 있습니다.

이 메서드는 collection타입(주로 배열)의 인스턴스에서 제공하는 메서드로 `by:` 에 어떻게 정렬을 할 것인지를 작성한 클로저를 넣어주면 해당 방법대로 정렬을 해줍니다.

```swift
let almonds = ["Honey Butter", "Wasabi", "Corn", "Buldak", "Ddukboki", "Mint Choco"]
almonds.sorted(by: ... )
```

`sorted(by:)`의 summary를 보면
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e11bd0df-4b69-4c66-818c-3113e937be7c)

Element 2개를 매개변수로 받아서 Bool을 리턴하는 클로저를 넣도록 정의되어 있습니다.

우리 예시는 String 배열이니 `String` 2개를 매개변수로 받아서 `Bool`을 리턴하는 클로저를 작성해서 전달하면 됩니다.

클로저를 전달하는 가장 쉬운 방법은 함수를 작성해서 전달하는 방법입니다.

아래 함수는 이름이 있고, 어떤 값도 캡쳐하지 않는 `전역함수` 형태의 클로저입니다.

```swift
let almonds = ["Honey Butter", "Wasabi", "Corn", "Buldak", "Ddukboki", "Mint Choco"]

funcdecrease(_ s1:String,_ s2:String) -> Bool {
return s1 > s2
}

let decreasedAlmonds = almonds.sorted(by: decrease)
// ["Wasabi", "Mint Choco", "Honey Butter", "Ddukboki", "Corn", "Buldak"]
```

위 `decrease` 함수를 클로저 표현 문법으로 변경하여 `sorted(:by)`에 넣어보면 아래와 같이 됩니다.

`in` 앞에 먼저 작성했던 함수의 `매개변수`, `반환타입` 형태 `(String, String) -> Bool` 를 작성하고,

`in` 뒤에 `body` 부분을 작성해주면 됩니다.

아래처럼 매개변수로 바로 들어가 있는 형태의 클로저를 `인라인 클로저` 라고 부릅니다.

```swift
let almonds = ["Honey Butter", "Wasabi", "Corn", "Buldak", "Ddukboki", "Mint Choco"]

let decreasedAlmonds = almonds.sorted(by: { (s1:String, s2:String) -> Boolinreturn s1 > s2
})
// ["Wasabi", "Mint Choco", "Honey Butter", "Ddukboki", "Corn", "Buldak"]
```

줄바꿈 없이 한줄로 표현도 됩니다.

```swift
let decreasedAlmonds = almonds.sorted(by: { (s1:String, s2:String) -> Boolinreturn s1 > s2 })
```

### 문맥(context)에서 매개변수 타입(parameter type)과 반환타입(return type)의 추론

위에서 본 `sorted(by:)`의 summary를 봤을 때, 컴파일러는 매개변수로 2개의 Element를,

여기서는 `String`을 넘겨야하며, 반환타입이 `Bool`이어야 한다는 사실을 이미 알고 있습니다.

`이미 알고 있는 타입은 문맥으로 타입추론을 하기 때문에 생략이 가능`합니다.

매개변수와 반환타입을 생략 가능하지만, 코드의 가독성을 위해 명시하여도 됩니다.

```swift
let decreasedAlmonds = almonds.sorted(by: { s1, s2inreturn s1 > s2 })
```

### 단일 표현 클로저에서의 암시적 반환

위 코드의 경우 body부분이 `return s1 > s2` 라는 단일 표현으로된 클로저인데, 이런 경우 `return` 키워드를 생략할 수 있습니다.

단일 표현 클로저의 경우 body의 표현이 암시적으로 반환이 됩니다.

```swift
let decreasedAlmonds = almonds.sorted(by: { s1, s2in s1 > s2 })
```

### 축약된 인자 이름

`인라인 클로저의 경우` 자동으로 축약 인자 이름(`$0, $1, $2`, ...)를 제공해줍니다.

축약 인자 이름을 사용하면 매개변수 이름도 적어줄 필요가 없어지게 되고, 매개변수 부분과 `in` 키워드까지 생략할 수 있습니다.

```swift
let decreasedAlmonds = almonds.sorted(by: { $0 > $1 })
```

### 후위 클로저 문법

어떤 함수의 마지막 매개변수가 클로저라면 `후위 클로저 문법`을 사용할 수 있습니다.

아래와 같은 함수가 있다면,

```swift
// 마지막 매개변수를 클로저로 넣는 함수
funcsomeFunction(with someClosure: () -> Void) {
    // 함수 body
}
```

아래와 같이 클로저를 전달해서 사용하게 되고,

```swift
someFunction(with: {
    // closure's body
})
```

이는 후위 클로저 문법을 사용하여 아래와 같이 표현할 수 있습니다.

매개변수 이름도 생략을 시키고 소괄호 ( ) 밖으로 중괄호 블럭 { } 으로 클로저 body를 작성하면 됩니다.

```swift
someFunction() {
    // closure's body
}
```

이렇게 줄이고 나니, 마치 보통 사용하던 전역함수와 같은 형태가 되었는데,

알고보니 전역함수 형태가 클로저를 이용해 작성되었던 것을 알 수 있습니다.

이제 예시로 작성했던 `sorted(by:)`를 후위 클로저 문법으로 수정을 해보면,

아래처럼 되고,

```swift
let decreasedAlmonds = almonds.sorted() { $0 > $1 }
```

후위 클로저 외에 다른 매개변수가 없다면 `소괄호( )도 생략`이 가능합니다.

```swift
let decreasedAlmonds = almonds.sorted { $0 > $1 }
```

후위 클로저가 있는 함수를 작성을 하려고 하면 Xcode가 자동으로 둘 중 하나를 선택하도록 알려주기도 합니다.

추천 입력에는 소괄호까지 생략된 형태인 것을 확인해 볼 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f24b017e-2fd9-4728-b503-3b6da53d46d9)

### 클로저는 참조 타입

Swift는 함수를 [1급객체로 취급](https://swiftycody.github.io/posts/Swift-1%EA%B8%89-%EA%B0%9D%EC%B2%B4-First-class-citizen-%EC%99%80-%EA%B3%A0%EC%B0%A8%ED%95%A8%EC%88%98-%EA%B3%A0%EA%B3%84%ED%95%A8%EC%88%98/)을 하고, 클로저가 `참조 타입`으로 분류됩니다.

그래서 클로저가 캡쳐하는 값들도 모두 [기본적으로 참조 타입으로 값 캡쳐](https://swiftycody.github.io/posts/Swift-%ED%81%B4%EB%A1%9C%EC%A0%80%EC%99%80-%EA%B0%92-%EC%BA%A1%EC%B3%90/)를 합니다.
