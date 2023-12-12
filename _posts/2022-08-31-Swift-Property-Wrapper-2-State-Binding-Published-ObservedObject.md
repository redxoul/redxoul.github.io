---
title: (SwiftUI) Property Wrapper (2) @State/@Binding, @Published/@ObservedObject
author: Cody
date: 2022-08-31 00:34:00 +0800
categories: [iOS Swift]
tags: [Swift, SwiftUI, iOS13]
pin: false
mermaid: true
---
 
이번 글에서는 Swift, SwiftUI로 작성 시 자주 쓰게 되는 `Property Wrapper`를 정리해봅니다.

### @State와 @Binding

`@State`와 `@Binding`은 SwiftUI에서 자주 사용하는 Property Wrapper입니다.

`@State`로 선언한 값을 참조하여 SwiftUI를 그리면, 해당 값이 변경될 때 SwiftUI가 이를 반영하여 변경됩니다.

```swift
// 예시
struct PlayButton: View {
    @State private var isPlaying: Bool = false

    var body: some View {
        Button(isPlaying ? "Pause" : "Play") {
            isPlaying.toggle()
        }
    }
}
```

`@State`로 선언한 값은 선언된 SwiftUI Struct내에서 참조하고 변경이 가능하며, 자식 뷰에서는 참조만 가능하고 값을 변경할 수는 없습니다.

자식 뷰에서 값을 참조하고 값을 변경도 하고 싶다면, 자식 뷰에서는 `@Binding`을 사용하면 됩니다.

```swift
struct PlayButton: View {
    @Binding var isPlaying: Bool // Binding으로 변경
    var body: some View {
        Button(isPlaying ? "Pause" : "Play") {
            isPlaying.toggle()
        }
    }
}
```

그리고 부모 뷰의 ``@State``값을 자식 뷰의 ``@Binding``값으로 전달할 땐 ``$``기호를 앞에 붙여줍니다.

```swift
struct PlayerView: View {
    var episode: Episode
    @State private var isPlaying: Bool = false

    var body: some View {
        VStack {
            Text(episode.title)
                .foregroundStyle(isPlaying ? .primary : .secondary)
            PlayButton(isPlaying: $isPlaying) // Binding에게 참조를 전달
        }
    }
}
```

`@State`를 뷰계층에서 초기화 시 `SwiftUI`의 Storage관리와 충돌이 일어날 수 있기 때문에

이를 방지하기 위해서 `private`로 선언해서 사용하기를 권장합니다.

`@State`와 `@Binding`은 모든 스레드에서 안전하게 값 변경이 가능합니다.

### @Published, 그리고 @ObservedObject

위에서 본 `@State`와 `@Binding`은 `SwiftUI` 내에서 뷰의 상태값을 변경하는데만 사용하도록 권장되고 있습니다.

만약 뷰 외부에서도 사용하고 싶다면? Model 클래스에서 값을 바인딩하고 변경된 값을 반영하고 싶다면?`@Published`, `@ObservedObject`를 사용하면 됩니다.

먼저 `@Published`를 먼저 보겠습니다.

이 `@Published`는 `Combine`의 `Published`를 Property Wrapper로 감싼 것으로

`ObservableObject` 프로토콜을 따르는 개체 내에서 사용되고,

`@Published` 프로퍼티의 값이 변경되었을 때, 해당 값을 방출합니다.

정확히는 '`값이 변경되기 전에(`objectWillChange`)`' 값을 내보냅니다.

`@Published`는 `class의 프로퍼티에만 사용`할 수 있습니다.

아래는 `ObservableObject` 프로토콜로 작성된 `CounterViewModel` 내에서 `@Published`로 작성된 count 프로퍼티입니다.

```swift
class CounterViewModel: ObservableObject {
    @Published var count: Int = 0
}
```

이 `@Published`로 선언된 값을 SwiftUI에서 받아서 사용할 때

`@ObservedObject` Property Wrapper가 사용됩니다.

아래와 같이 `CounterViewModel`의 count값을 바꿔주면, 변경된 count값이 방출되어 Text에 반영이 됩니다.

```swift
struct Counter: View {
    @ObservedObject var viewModel = CounterViewModel()
    
    var body: some View {
        HStack(spacing: 20) {
            Button {
                viewModel.count = viewModel.count-1 >= 0 ? viewModel.count-1 : 0
            } label: {
                Image(systemName: "minus.square")
                    .font(.system(size: 50))
            }
            
            Text("\(viewModel.count)")
                .font(.system(size: 50))
            
            Button {
                viewModel.count += 1
            } label: {
                Image(systemName: "plus.square")
                    .font(.system(size: 50))
            }
        }
    }
}
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/60f6567e-9a89-4fe1-b895-b56291acb2aa)

다음 글은 이어서,Property Wrapper 중 `@EnvironmentObject`와 `@Environment`에 대해서 정리할 예정입니다.