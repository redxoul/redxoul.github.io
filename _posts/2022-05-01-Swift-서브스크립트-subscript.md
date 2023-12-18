---
title: (Swift) 서브스크립트 subscript
author: Cody
date: 2022-05-01 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---
`서브스크립트`는 `콜렉션 타입(Array, Dictionary, Set)`, `시퀸스` 등 집합의 인스턴스에서 특정 항목에 쉽게 접근할 수 있게 해주는 문법입니다.

우리가 평소에 자연스럽게 사용하던 배열 인스턴스의 특정 항목에 접근하기 위해 사용하는 `someArray[index]`,  `Dictionary`에서의 `someDictionary[key]` 과 같은 문법이 `서브스크립트`입니다.

`서브스크립트`를 통해 추가적인 메서드 필요없이 특정 값을 할당하거나 가져올 수 있습니다.

그리고 하나의 타입에 여러 `서브스크립트`를  정의하고, `overload`도 가능. `매개변수`도 하나가 아니라 여러개를 받도록 정의할 수 있습니다.

### 서브스크립트의 작성

서브스크립트를 선언하는 방법은 계산프로퍼티를 선언하는 것과 유사합니다.

```swift
subscript(index: Int) -> Int {
    get {
        // index에 맞는 적절한 반환 값
    }
    set(newValue) {
        // index에 적절한 set
    }
}
```

`읽기전용(read-only) 서브스크립트`를 선언하려면 위처럼 `get`, `set()`을 작성하지 않고

아래처럼 클로저 내에 반환 값만 잘 작성해주면 됩니다.

```swift
subscript(index: Int) -> Int {
    // index에 맞는 적절한 반환 값
}
```

### 예시

```swift
struct Almond {
    let name: String
    subscript(count: Int) -> String {
        return "\(name)맛 아몬드는 \(count)개를 먹어야지"
    }
}

let myAlmond = Almond(name: "MintChoco")
print(myAlmond[0]) // MintChoco맛 아몬드는 0개를 먹어야지
```

아래는 여러개의 매개변수를 받아서 처리하는 예시입니다.

```swift
enum Almond {
    case honeyButter, wasabi, corn, mintChoco
}

struct VendingMachine {
    subscript(row: Int, column: Int) -> Almond {
        let value = row*column
        if value%2 == 0 {
            return .mintChoco
        }
        else if value%3 == 0 {
            return .wasabi
        }
        else if value%4 == 0 {
            return .corn
        }
        else {
            return .honeyButter
        }
    }
}

let aVendingMachine = VendingMachine()
let myAlmond = aVendingMachine[1, 3] // wasabi
let yourAlmond = aVendingMachine[2, 10] // mintChoco
```