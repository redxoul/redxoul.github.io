---
title: (Swift) escaping 클로저와 순환참조
author: Cody
date: 2022-04-18 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

Swift에서는 함수가 `1급객체`이기 때문에 함수를 ``매개변수``로 넣을수가 있습니다.

이 때 함수가 끝나고 실행되거나 함수 밖에 저장되는 클로저일 때,

(onNext, onComplete, onError이벤트 클로저 같은)

보통 비동기 작업 후 실행되는 (completeHander로 많이 쓰이는)클로저는 `매개변수` 타입 앞에 `@escaping` 키워드를 명시해주어야 합니다.

명시해주지 않았을 때에는 error가 발생합니다.

(converting non-escaping parameter '(매개변수 명)' to generic parameter 'Element' may allow it to escape)

```swift
// 함수를 저장해놓기 위한 배열
var completeHandlers = [() ->Void]()

// completeHandlers에 함수를 저장하는 함수
func addCompleteHandler(completeHander: @escaping () -> Void) {
    completeHandlers.append(completeHander)
}
```

## @escaping 클로저와 순환참조

`escaping`클로저 내에서 `self`를 참조해야할 때가 있습니다.

클로저 내부에서 인스턴스를 참조를 하게 되었을 때 강한 순환참조가 발생할 수가 있습니다.

(`non-escaping`클로저라면 `self`를 참조해도 클로저 내에서 작업을 마치고 메모리 해지가 되기 때문에 상관이 없습니다)

```swift
// 함수를 저장해놓기 위한 배열
var completeHandlers = [() ->Void]()

class SomeClass {
  var someValue = 10
  
  // completeHandlers에 함수를 저장하는 함수
  func addCompleteHandler(completeHander: @escaping () -> Void) {
    completeHandlers.append(completeHander)
  }
  
  func someFunc() {
    addCompleteHandler(completeHander: {
      self.someValue += 10 // self(SomeClass의 인스턴스)가 캡쳐
      print("addedHandler: \(self.someValue)")
    })
  }
}
```

이런 경우 `SomeClass` 인스턴스의 `someFunc()`에서 `addCompleteHandler`로 클로저를 전달하여 `completeHandlers`에 저장되고,

전달한 클로저에서 `self`(SomeClass의 인스턴스)의 참조 카운트가 올라가게 되는데,

`someClass`인스턴스를 다 사용하고 난 뒤에도 카운트가 남게되어 메모리 누수가 될 수 있습니다.

그래서 클로저의 인스턴스 참조로 안한 강한 순환참조를 없애기 위해 `[weak self]`를 사용합니다.

1) 아래처럼 `[weak self]`로 선언 후 `옵셔널 체이닝`으로 사용을 하거나,

```swift
addCompleteHandler(completeHander: { [weak self] in
	self?.someValue += 10
	print("addedHandler: \(self?.someValue)")
})
```

2) `guard let self = self` 로 빠른 탈출 구문으로 사용하는 방법이 있습니다.

```swift
addCompleteHandler(completeHander: { [weak self] in
	guard let self = self else { return }
	self.someValue += 10
	print("addedHandler: \(self.someValue)")
})
```