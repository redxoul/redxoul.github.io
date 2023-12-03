---
title: (Swift) KVO(Key-Value Observing)
author: Cody
date: 2023-02-09 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, KVO]
pin: false
mermaid: true
---
**`Key-Value Observing(KVO)`**은 다른 객체의 **property** 변경에 대해 객체에 알리는 데 사용하는 코코아 프로그래밍 패턴입니다.

**Model**과 **View** 사이와 같이 앱의 논리적으로 분리된 것 사이의 변경 사항을 전달하는 데 유용합니다.

**`willSet`**, **`didSet`**과 유사하지만 **`KVO`**는 객체 **외부**에서 **property**변경을 관찰하는 데 사용된다는 차이점이 있습니다.

**`NSObject`**를 상속받은 클래스에서만 **`Key-Value Observing`**을 사용할 수 있습니다.

**`Key-Value Observing`**을 위해 **Observe**할 **property** 앞에 **`@objc` attribute**와 **`dynamic` modifier**를 붙여주어야 합니다.

```swift
class Almond: NSObject { // NSObject를 상속받아야 함
    @objc dynamic var name: String // 관찰할 대상

    init(name: String) {
        self.name = name
    }
}
```

**Observe**는 다음과 같이 설정해 줍니다.

관찰할 대상을 **`KeyPath`**(\.name)로 넣어줍니다.

```swift
var almond = Almond(name: "HoneyButter")
almond.observe(\.name, options: [.initial, .old, .new]) { object, change in // .initial 옵션일 땐 observe하자마자 newValue로 초기값을 받아옴
    print("old almond: \(change.oldValue)")
    print("new almond: \(change.newValue)")
}
```

설정할 때 **`initial`**, **`old`**, **`new`**, **`prior`**로 **options**를 줄 수 있는데요.

관찰하고 싶은 상태들을 옵션으로 넣어주면 됩니다.

위와 같이 **`initial`**을 설정하면 초기값도 포함하여 받아오게 되고, **observe** 설정하자마자 **`newValue`**로 초기값이 받아와 집니다.

```
old almond: nil
new almond: Optional("HoneyButter")
```

그리고 값을 변경시켜 주면

```swift
almond.name = "Wasabi"
```

조금 전 받은 초기값(HoneyButter)은 **`oldValue`**로, 변경시켜 준 값(Wasabi)은 **`newValue`**로 받습니다.

```
old almond: Optional("HoneyButter")
new almond: Optional("Wasabi")
```

마지막으로 **`prior`**를 **options**에 포함하면 어떻게 될까요?

이번엔 print를 조금 바꿔봤습니다. 이유는 아래에 나옵니다.

```swift
var almond = Almond(name: "HoneyButter")
almond.observe(\.name, options: [.initial, .old, .new, .prior]) { object, change in
    print("old almond" + (change.isPrior ? "(prior): " : ": ") + "\(change.oldValue)")
    print("new almond" + (change.isPrior ? "(prior): " : ": ") + "\(change.newValue)")
}
```

초기값까지는 아까와 동일합니다.

```swift
old almond: nil
new almond: Optional("HoneyButter")
```

하지만 새 값으로 바꿔주면?

```
almond.name = "Wasabi"
```

아래처럼 변경되기까지의 과정(?)까지 받게 됩니다.

(1) 새 값을 **`newValue`**로 넣기 전에 기존 **`newValue`**(HoneyButter)를 **`oldValue`**로 넣고, **`newValue`**에 **`nil`**을 넣은 상태(**`prior`**)를 받고

(2) 새 값(Wasabi)을 **`newValue`**로 넣은 상태도 전달받습니다.

```
old almond(prior): Optional("HoneyButter")
new almond(prior): nil
// (편의상 분리)
old almond: Optional("HoneyButter")
new almond: Optional("Wasabi")
```

위에서 **print**에 **`isPrior`**값을 넣어준 이유는 **`prior`**상태의 값을 구분해 주기 위해 필요하기 때문이었습니다.

여기까지 `KVO`에 대한 글이었습니다.