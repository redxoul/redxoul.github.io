---
title: (SwiftUI) ViewModifier 프로토콜
author: Cody
date: 2022-09-05 00:34:00 +0800
categories: [iOS Swift]
tags: [Swift, SwiftUI, iOS13]
pin: false
mermaid: true
---

`ViewModifier` 프로토콜은

'View 혹은 View Modifier에 적용해서 기존과 다른 버전을 생성하는 프로토콜' 입니다.

다시말해 커스텀 Modifier를 만들 수 있게 해주는 프로토콜입니다.

간단한 예시로 보겠습니다.

아래와 같이 `ViewModifier` 프로토콜을 작성해줍니다.

저는 제 앱에서 공통으로 사용할 타이틀 크기와 색상을 지정해주었습니다.

```swift
struct MyAppTitle: ViewModifier {
    func body(content: Content) -> some View {
        return content
            .font(.system(size: 30, weight: .bold))
            .foregroundColor(.indigo)
    }
}
```

그리고 위에서 작성한 `ViewModifier`를 View에서 적용시키기 위한 함수를 작성해줍니다.

```swift
extension View {
    func myAppTitle() -> some View {
        return self.modifier(MyAppTitle())
    }
}
```

PreviewProvider를 통해 확인해보면,

제가 적용한 `.myAppTitle()` modifier가 잘 적용된 것을 볼 수 있습니다.

```swift
struct MyAppTitle_Previews: PreviewProvider {
    static var previews: some View {
        VStack(alignment: .leading) {
            Text("타이틀")
                .myAppTitle()
            
            Text("내용")
        }
    }
}
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7fb35b6d-f730-42fd-9fcf-611058f2cd1d)