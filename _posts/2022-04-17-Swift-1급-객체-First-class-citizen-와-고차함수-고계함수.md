---
title: (Swift) 1급 객체(First class citizen)와 고차함수(고계함수)
author: Cody
date: 2022-04-17 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

프로그래밍 언어에서 ``1급 객체``란 아래의 조건을 충족시키면 됩니다.

1. 변수나 데이터에 할당 할 수 있어야 한다.

2. 객체의 매개변수로 넘길 수 있어야 한다.

3. 객체의 반환값으로 리턴 할수 있어야 한다.

`Swift` 기본타입들(`Int`, `Bool`, `Struct`, ...)은 물론 `1급 객체`이고,

`함수` 또한 `1급 객체`로 취급이 됩니다.

### 1. 변수나 데이터에 할당

```swift
// Int형 파라메터 2개를 받아서 Int를 반환하는 함수형을 선언
var mathFunction: (Int, Int) -> Int // Int값 두 개를 입력받고 Int를 반환하는 함수
func addTwoInts(_ a: Int, _ b: Int) -> Int {
return a + b
}

// addTwoInts함수를 mathFunction변수에 할당
mathFunction = addTwoInts(_:_:)
```

### 2. 객체의 매개변수로 함수를 넘기기

```swift
// Int를 받아서 Int를 반환하는 함수형 선언
var someFunction: (Int) -> Int

// 함수
func increamenter(input : Int) -> Int {
    return input*5
}

// someFunction 변수에 increamenter함수를 할당
someFunction = increamenter(input:)

// 함수를 매개변수로 받는 함수
func functionParameter(f: (Int) -> Int) -> Int {
    return f(5)
}

//
functionParameter(f: someFunction) // 25
```

### 3. 객체의 반환값으로 함수를 반환

```swift
// 반환형으로 쓰기 위한. 입력한 스텝에 하나를 빼거나 더하는 함수
func stepForward(_ input: Int) -> Int {
    return input + 1
}
func stepBackward(_ input: Int) -> Int {
    return input - 1
}

// backward함수가 true냐 false냐에 따라 위에서 선언한 적절한 함수를 반환하는 함수
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
    return backward ? stepBackward : stepForward
}

// 사용 예시
var currentValue = 3
let moveNearerToZero = chooseStepFunction(backward: currentValue > 0)
// moveNearerToZero는 이제 stepBackward() 함수를 가르키고 있음.

// moveNearerToZero를 호출할 때마다 
// stepBackward() 함수가 호출돼 입력 값이 1씩 줄어들어 결국 0이 됨
print("Counting to zero:")
// Counting to zero:
while currentValue != 0 {
    print("\(currentValue)... ")
    currentValue = moveNearerToZero(currentValue)
}
print("zero!")
// 3...
// 2...
// 1...
// zero!
```

### 고차함수

위처럼 함수를 `매개변수`로 받거나, 반환값으로 갖는 함수를 ``고차함수``(혹은 `고계함수`)라고 부릅니다.