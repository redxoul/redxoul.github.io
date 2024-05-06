---
title: (SwiftUI) ButtonStyle 프로토콜
author: Cody
date: 2022-09-06 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, SwiftUI, iOS13]
pin: false
mermaid: true
---

바로 이전 글에서 정리했던 ViewModifier 프로토콜과 유사하게  
`ButtonStyle` 이라는 프로토콜이 있습니다.

이 프로토콜로 작성한 구조체는

SwiftUI Button의 `.buttonStyle()` modifier를 통해 `ButtonStyle`을 설정할 때 쓸 수 있습니다.

간단한 예제입니다.

`ButtonStyle` 프로토콜의 구조체는

`func makeBody(configuration:) -> some view` 를 구현해주면 됩니다.

```swift
struct CustomButtonStyles: ButtonStyle {
    var systemName: String
    var title: String
    
    func makeBody(configuration: Configuration) -> some View {
        return VStack {
            Image(systemName: systemName)
                .resizable()
                .frame(width: 30, height: 30)
                .padding([.leading, .trailing], 10)
            
            Text(title)
                .font(.system(size: 12, weight: .bold))
        }
        .padding()
        .foregroundColor(.blue)
        .background(Color.white)
        .clipShape(RoundedRectangle(cornerRadius: 10))
    }
}
```

그리고 아래와 같이 적용하면 끝입니다.

```swift
struct CustomButtonStyles_Previews: PreviewProvider {
    static var previews: some View {
        ZStack {
            Color.gray.opacity(0.5)
                .edgesIgnoringSafeArea(.all)
            
            Button("") {
                print("some Action")
            }
            .buttonStyle(CustomButtonStyles(systemName: "house.fill", title: "Home"))
        }
    }
}
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/57d82c7b-b5f0-4f6a-916b-71ef73eeb5ac)