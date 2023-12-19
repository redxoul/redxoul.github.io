---
title: (Swift) 클로저와 Capture List
author: Cody
date: 2022-04-23 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---
## 캡쳐 리스트

캡쳐링을 하는 값의 `참조 규칙`을 캡쳐리스트를 통해서 정해줄 수 있습니다.

클로저의 캡쳐리스트 내에 정의하면 캡쳐링할 때 본래의 타입에 맞게 캡쳐링되도록 사용되게 할 수 있습니다.

즉, `값 타입은 클로저가 생성될 시점의 값이 copy`되어지고,

`참조 타입은 클로저가 호출되는 시점에 참조`되어 사용되도록 사용되어지게 합니다.

캡쳐리스트는 클로저 `in` 앞에 `[ ]` 내에 넣는 형식으로 작성합니다.

## 값 타입의 참조규칙

예시:

```swift
var x = 0
var y = 0

let someClosure = { [x] in
	print("in closure.x: \(x)")
  print("in closure.y: \(y)")
}

x = 20
y = 20

someClosure()
//in closure.x: 0
//in closure.y: 20
```

위 예시처럼,

`y`는 값 타입이지만, 캡쳐링되면서 참조하게 되어 외부에서 변동된 값이 반영됩니다.

`x`도 값 타입이지만, 캡쳐링할 때 캡쳐리스트 `[ ]`에 넣게 되면서 값이 `0`으로 복사되어 캡쳐링된 것을 확인할 수 있습니다.

## 참조타입의 참조규칙

예시 1:

```swift
class someClass {
	var value = 0
}

var x = someClass()
var y = someClass()

let someClosure = { [x] in
	print("in clousre x.value: \(x.value)")
  print("in clousre y.value: \(y.value)")
}

x.value = 10
y.value = 10

someClosure()
//in clousre x.value: 10
//in clousre y.value: 10
```

`x`, `y` 모두 참조타입이며,

캡쳐리스트에 넣은 `x`도, 넣지않은 `y`도 모두 참조되어 사용되는 것이 확인됩니다.

예시2:

캡쳐하려는 값이 참조 타입일 때, `메모리 참조 규칙`을 정할 수 있습니다.

예시1처럼 아무것도 적지 않으면 기본적으로 `strong`이 되고 `weak`, `unowned`를 설정할 수 있습니다.

`strong`참조가 되었을 때 `강한순환참조`가 발생하여 메모리누수가 되는 상황이 있는데, 이를 막기 위해 `weak`와 `unowned`를 사용합니다.

`weak`참조와 `unowned`참조 모두 인스턴스의 참조카운트를 증가시키지 않고 참조하지만,

`weak`는 사용시점에 메모리카운트가 `0`일 때 `nil`을 반환시키는 `optional`로 사용(`?`로 unwrapping)하는 반면에

`unowned`는 메모리카운트가 `0`이더라도 `optional`로 사용하지 않아 댕글링포인터를 참조해 충돌이 일어날 가능성이 생겨납니다.

그래서 `unowned`로 사용을 하려면 사용시점에 절대 해제되지 않을거라고 판단이 되는 곳에서만 사용해야 합니다.

(순환참조는 [이전 포스팅](https://swifty-cody.tistory.com/11)을 참조. 메모리 참조에 대한 포스팅은 별도로 작성하겠습니다)

```swift
class someClass {
    var value = 0
}

var x: someClass? = someClass()
var y = someClass()

let someClosure = { [weak x, unowned y] in
    print("in clousre x.value: \(String(describing: x?.value))")
    print("in clousre y.value: \(y.value)")
}

x = nil
y.value = 10

someClosure()
//in clousre x.value: nil
//in clousre y.value: 10
```