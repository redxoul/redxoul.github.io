---
title: (SwiftUI) PreviewProvider 프로토콜, Preview (feat. Xcode14(Beta))
author: Cody
date: 2022-08-15 00:34:00 +0800
categories: [iOS Swift]
tags: [Swift, SwiftUI, iOS13]
pin: false
mermaid: true
---
`SomeSwiftUIView`라는 `SwiftUI` 파일을 생성을 하면 하단에
`SomeSwiftUIView_Preview`라는 이름의
`PreviewProvider`프로토콜을 따르는 구조체가 함께 생성이 됩니다.

```swift
struct SomeSwiftUIView_Previews: PreviewProvider {
    static var previews: some View {
        SomeSwiftUIView()
    }
}
```

이 구조체는
1. 우리가 작성한 `SwiftUI` `View`의 내용을 `Preview`(혹은 `Canvas`)에 반영시켜주거나,
2. `PreviewProvider` 내의 `previews`의 내용을 `Preview`에 반영시켜주어

우리가 확인하고자 하는 `View`를 빠르게 확인할 수 있게 해줍니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/80309f2b-e4e3-4f20-ba33-d0e2a1c4478a)

1. 우리가 실제로 작성해서 사용하는 `View`는 상단의 `SomeSwiftUIView`에 작성하면 되고,
2. 해당 `SomeSwiftUIView`를 사용한 다양한 Variation를 확인하고 싶다면 `PreviewProvider`쪽을 수정하면 됩니다.

Preview가 `paused` 상태가 되었을 때는
`command`+`option`+`P` 단축키로 다시 갱신된 `Preview`로 실행시켜줄 수 있습니다.

아래와 같이 `previews`를 수정해서
`SomeSwiftUIView`를 변경하거나, 다른 `View`와 함께 작성해본 내용을 확인해볼 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/342a2818-ffec-4015-8f20-21b911f5837b)

기본적으로 프로젝트에서 현재 선택된 시뮬레이터를 기반으로 빌드하여 Preview에 보여주게 됩니다.

다른 디바이스에서의 `SomeSwiftUIView`의 모습을 확인하고 싶을 때 2가지 방법이 있습니다.

1. Xcode 상단에 Simulator를 원하는 디바이스로 변경해주거나,
2. `previewDevice` Modifier를 사용하면 됩니다. `PreviewDevice`의 rawValue는 시뮬레이터 이름을 그대로 써주면 됩니다.

`previewDevice` Modifier를 작성한 경우, Xcode에서 선택한 시뮬레이터보다 우선 적용이 되며

기존에 빌드했던 디바이스가 아니라면 새로 빌드(Prepare)한 후에 Preview에 반영을 해줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ca590849-4996-4815-b0e0-d34e4153be55)

```swift
// previewDevice 예시
struct SomeSwiftUIView_Previews: PreviewProvider {
    static var previews: some View {
        SomeSwiftUIView()
            .previewDevice(PreviewDevice(rawValue: "iPhone 8"))
    }
}
```

아래처럼 여러 디바이스에서의 Preview를 동시에 확인도 가능합니다.
하지만 각각을 모두 빌드해야 하기 때문에 반영이 느려집니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/60947501-696e-4418-bf64-c884172241c0)

디바이스 방향(orientation)을 변경해서 보여주는 `previewInterfaceOrientation` Modifier도 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/717e12d2-fd69-4a49-b459-f4fa4f3a3bdb)

```swift
// previewInterfaceOrientation 예시
struct SomeSwiftUIView_Previews: PreviewProvider {
    static var previews: some View {
        SomeSwiftUIView()
            .previewInterfaceOrientation(.landscapeLeft)
    }
}
```

### Live Mode

Preview에서 `동적으로 데이터를 받아와서 반영된 모습을 확인`하고 싶거나,

`애니메이션을 확인`하고 싶다면 `Live Mode`로 확인할 수 있습니다. Preview 디바이스 상단에 삼각형버튼을 누르면 Live Mode가 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3ad5fdaf-ed80-4783-897d-3d2f0eeacc74)

### Xcode14(Beta)에서 Preview의 변경점

1. `Live Mode`가 기본 적용됩니다.

2. `Canvas Device Setting`을 통해서 `Color Scheme`, `Orientation`, `Dynamic Type`에 따른 Preview를 세팅할 수가 있습니다.

(위에서 설명한 Xcode13에서처럼 PreviewProvider에 Modifier를 작성할 필요가 없어졌고 이 세팅은 Live Mode로 지원합니다)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b98fb991-0cd7-438d-9176-fffeaac9212b)

3. `Color Scheme Variants`, `Orientation Variants`, `DynamicType Variants`를 Preview에서 선택해서 `동시에 확인`이 가능해졌습니다. 이 모드에서는 `Live Mode가 해제`됩니다.

(Live Mode가 해제되는 것이 조금 아쉽지만, Xcode13에서 Light Mode/Dark Mode 혹은 Landscape/Portrait을 동시에 확인하기 위해 PreviewProvider에 2개의 preview를 작성하면 각각 빌드를 하여 속도가 느려지는 문제가 있었는데, Xcode14에서는 이를 빠르게 확인할 수 있게 되어 좋아졌습니다)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/96ec5039-8af2-4bc3-adba-a229c3e6bfa8)

`추가적으로. 프로젝트가 시뮬레이터를 사용할 수가 없는 경우?!`

Preview는 시뮬레이터 빌드 기반이기 때문에
빌드가 Success가 되어야만 Preview도 보이게 됩니다.

빌드가 Fail이 되면 Preview에 🟢표시가 🔴로 표시되며, 변경된 사항에 대해 Preview에 반영을 해주지 않습니다(마지막으로 Succss였던 Preview를 보여주게 됩니다).
