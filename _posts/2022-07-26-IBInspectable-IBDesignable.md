---
title: IBInspectable, IBDesignable
author: Cody
date: 2022-07-26 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Swift, IBInspectable, IBDesignable]
pin: false
mermaid: true
---
스토리보드에서도 SwiftUI처럼 실시간(?)으로 코드 구현부를 스토리보드에 반영할 수 있는 방법이 있어
이를 정리해보는 포스팅을 작성해봅니다.

바로 `@IBInspectable`, `@IBDesignable`인데요.

간단한 예제입니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0730315f-f577-44a1-95e6-c5fab3a15786)

계산기를 만드는 중인 스토리보드입니다.

아시다시피 스토리보드에서 `View`를 누르면 오른쪽의 `Attributes` `Inspector`에서 해당 `View`의 `Attribute`들을 수정할 수 있습니다.

이 상태에서 계산기 버튼들에 `cornerRadius`를 주어 둥글게 만들고 싶은데, 이럴 경우 코드를 작성해야 합니다.

### @IBInspectable

이름에서도 느껴지듯이 이 `Attribute`를 사용하면 스토리보드의 `Inspector`에서 값을 바로 수정할 수 있게 해 줍니다.

`UIButton`을 상속받은 `RoundButton`을 작성해봅니다.

```swift
// RoundButton.swift
class RoundButton: UIButton {
    @IBInspectable var isRound: Bool = true {
        didSet {
            if isRound {
                self.layer.cornerRadius = self.frame.height / 2
            }
        }
    }
}
```

`isRound`값을 `IBInspectable`로 선언해주고,

해당 값이 `true`일 때, `cornerRadius`를 설정해주도록 했습니다.

그리고 `RoundButton`을 `Custom` `Class`로 설정해주면,
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1863c8b0-a8db-4375-85b0-b757b95ccfc2)

다음과 같이 `Inspector`에서 해당 값을 수정할 수 있게 되었습니다.
![값을 바꿀 수 있게 해준다고 했지 바로 반영해준다고는 안 했다..](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ed61b8aa-9c25-4cdd-a1bd-aa8158cffbef)
(값을 바꿀 수 있게 해준다고 했지 바로 반영해준다고는 안 했다)

하지만 값을 바꾼다고 스토리보드에서 바로 값이 적용이 되지는 않습니다.

이제 `@IBDesignable`을 사용할 때입니다.

### @IBDesignable

바로 작성해봅니다. 아까 작성한 코드에 `@IBDesignable`만 선언해주면 됩니다.

```swift
// RoundButton.swift
@IBDesignable // 추가
class RoundButton: UIButton {
    @IBInspectable var isRound: Bool = true {
        didSet {
            if isRound {
                self.layer.cornerRadius = self.frame.height / 2
            }
        }
    }
}
```

그리고 스토리보드로 돌아가면 `cornerRadius`가 반영되어 있는 걸 확인해볼 수 있습니다.

`Inspector`에는 `Designables`가 적용이 되었는지 표시가 됩니다.

표시가 바로 안된다면 `cmd`+`B`를 눌러 빌드를 진행하고 나면 바로 반영이 됩니다)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6b5a8030-a2b2-4aa0-9886-910ac91ae075)